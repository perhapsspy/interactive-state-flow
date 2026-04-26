# Before: Tangled Desktop File Preview

Electron-style pseudocode for a file browser where row selection, disk IO, parsing, sorting/filtering, preview generation, and UI commits all live inside the selection handler.

```ts
class FilePreviewWindow {
  selectedPath: string | null = null;
  rows: FileRow[] = [];
  filteredRows: FileRow[] = [];
  previewHtml = "";

  async onFileSelected(path: string) {
    // Selection feedback waits behind everything below.
    this.setStatus("Opening " + path);

    const stats = await fs.promises.stat(path);
    const bytes = await fs.promises.readFile(path);

    let parsed: ParsedFile;
    if (path.endsWith(".csv")) {
      parsed = parseCsv(bytes.toString("utf8")); // can take hundreds of ms
    } else if (path.endsWith(".json")) {
      parsed = parseJsonWithSchema(bytes.toString("utf8"));
    } else {
      parsed = parseText(bytes);
    }

    // Source data, filtering, sorting, and presentation are all recomputed here.
    this.selectedPath = path;
    this.rows = parsed.rows;
    this.filteredRows = parsed.rows
      .filter(row => row.name.includes(this.searchBox.value))
      .sort((a, b) => this.sortColumn.compare(a, b));

    // Preview work runs before the user sees selection as current.
    this.previewHtml = renderPreviewHtml({
      path,
      stats,
      rows: this.filteredRows,
      theme: this.currentTheme,
      width: this.previewPane.clientWidth
    });

    // Any older async call can arrive here and overwrite a newer selection.
    this.fileList.select(path);
    this.grid.replaceRows(this.filteredRows);
    this.previewPane.innerHTML = this.previewHtml;
    this.setStatus("Ready");
  }

  async onSearchChanged(value: string) {
    // Search repeats parse/preview work because there is no source/presentation split.
    if (this.selectedPath) {
      await this.onFileSelected(this.selectedPath);
    }
  }
}
```

Problems:

- The actual selection state is delayed by file IO, parsing, sorting, and preview rendering.
- Expensive derived data is treated as source state.
- Search changes re-enter the selection path and can trigger redundant disk reads.
- Async completions have no freshness/session check, so old selections can overwrite newer ones.
- Background work would be unsafe here because no owner controls cancellation, failures, or final UI commits.
