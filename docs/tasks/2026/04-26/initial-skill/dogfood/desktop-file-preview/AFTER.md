# After: Owner-Gated Desktop File Preview

Refactored Electron-style structure. Selection and source state update immediately; file IO/parsing/preview preparation run as owned background work; only the screen owner commits presentation after a freshness check.

```ts
type SourceState = {
  selectedPath: string | null;
  searchText: string;
  sort: SortSpec;
  sessionId: number;
};

type PresentationState = {
  visibleRows: FileRow[];
  previewHtml: string;
  previewPath: string | null;
  pendingPath: string | null;
  error: string | null;
};

class FilePreviewWindow {
  source: SourceState = {
    selectedPath: null,
    searchText: "",
    sort: defaultSort,
    sessionId: 0
  };

  presentation: PresentationState = {
    visibleRows: [],
    previewHtml: "",
    previewPath: null,
    pendingPath: null,
    error: null
  };

  private currentLoad?: AbortController;

  onFileSelected(path: string) {
    // Immediate user intent and source state.
    const sessionId = this.source.sessionId + 1;
    this.source = { ...this.source, selectedPath: path, sessionId };

    this.fileList.select(path);
    this.presentation = {
      ...this.presentation,
      pendingPath: path,
      error: null
    };
    this.renderChromeOnly();

    this.currentLoad?.abort();
    const abort = new AbortController();
    this.currentLoad = abort;

    // Background boundary is justified: disk IO, parsing, sorting/filtering, and
    // preview model generation can block selection feedback for large files.
    previewWorker.loadAndPrepare({
      path,
      searchText: this.source.searchText,
      sort: this.source.sort,
      sessionId,
      signal: abort.signal,
      previewWidth: this.previewPane.clientWidth
    }).then(result => this.acceptPreviewResult(result))
      .catch(error => this.acceptPreviewFailure({ path, sessionId, error }));
  }

  onSearchChanged(searchText: string) {
    // Search text is source state and is never debounced.
    const sessionId = this.source.sessionId + 1;
    this.source = { ...this.source, searchText, sessionId };
    this.renderSearchText();

    if (this.source.selectedPath) {
      this.startDerivedRefresh(this.source.selectedPath, sessionId);
    }
  }

  onSortChanged(sort: SortSpec) {
    const sessionId = this.source.sessionId + 1;
    this.source = { ...this.source, sort, sessionId };
    this.startDerivedRefresh(this.source.selectedPath, sessionId);
  }

  private startDerivedRefresh(path: string | null, sessionId: number) {
    if (!path) return;

    this.currentLoad?.abort();
    const abort = new AbortController();
    this.currentLoad = abort;

    this.presentation = { ...this.presentation, pendingPath: path, error: null };
    this.renderPendingPresentation();

    previewWorker.loadAndPrepare({
      path,
      searchText: this.source.searchText,
      sort: this.source.sort,
      sessionId,
      signal: abort.signal,
      previewWidth: this.previewPane.clientWidth
    }).then(result => this.acceptPreviewResult(result))
      .catch(error => this.acceptPreviewFailure({ path, sessionId, error }));
  }

  private acceptPreviewResult(result: PreparedPreview) {
    // Owner-only freshness gate. Worker output never mutates UI directly.
    if (result.sessionId !== this.source.sessionId) return;
    if (result.path !== this.source.selectedPath) return;
    if (!this.isActiveScreen()) return;

    this.presentation = {
      visibleRows: result.visibleRows,
      previewHtml: result.previewHtml,
      previewPath: result.path,
      pendingPath: null,
      error: null
    };

    this.grid.replaceRows(this.presentation.visibleRows);
    this.previewPane.innerHTML = this.presentation.previewHtml;
    this.setStatus("Ready");
  }

  private acceptPreviewFailure(failure: PreviewFailure) {
    if (failure.sessionId !== this.source.sessionId) return;
    if (failure.path !== this.source.selectedPath) return;
    if (failure.error.name === "AbortError") return;

    this.presentation = {
      ...this.presentation,
      pendingPath: null,
      error: "Preview unavailable"
    };
    this.renderPreviewError();
  }
}
```

Worker contract:

```ts
async function loadAndPrepare(input: PreviewRequest): Promise<PreparedPreview> {
  const bytes = await readFile(input.path, { signal: input.signal });
  const parsed = parseByExtension(input.path, bytes, input.signal);
  const visibleRows = parsed.rows
    .filter(row => row.name.includes(input.searchText))
    .sort(input.sort.compare)
    .slice(0, 500);

  return {
    path: input.path,
    sessionId: input.sessionId,
    visibleRows,
    previewHtml: renderPreviewHtml({ rows: visibleRows, width: input.previewWidth })
  };
}
```

Behavior contracts to test:

- Selecting a file immediately updates `source.selectedPath`, row highlight, pending state, and session id.
- If file A starts before file B but resolves after B, A is ignored.
- Search and sort update source state immediately while derived rows and preview may lag.
- Aborted or inactive-screen results cannot commit grid rows or preview HTML.
- Worker failures set presentation error state only through `FilePreviewWindow`.
