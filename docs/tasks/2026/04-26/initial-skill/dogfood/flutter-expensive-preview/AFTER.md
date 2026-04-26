# After: Immediate Source State, Lagging Preview State

The refactor gives the screen one owner for source state and presentation commits. Text input stays immediate. Expensive preview generation and tag IO are scheduled with an operation id and input snapshot. Completed work can update presentation only after the controller accepts it as fresh.

```dart
@immutable
class ListingSourceState {
  const ListingSourceState({
    required this.description,
    required this.revision,
  });

  final String description;
  final int revision;
}

@immutable
class ListingPresentationState {
  const ListingPresentationState({
    required this.preview,
    required this.previewRevision,
    required this.tags,
    required this.tagsRevision,
    required this.previewPending,
    required this.tagsPending,
    this.error,
  });

  final ListingPreview? preview;
  final int? previewRevision;
  final List<String> tags;
  final int? tagsRevision;
  final bool previewPending;
  final bool tagsPending;
  final String? error;
}

@immutable
class PreviewJobInput {
  const PreviewJobInput({
    required this.description,
    required this.revision,
    required this.screenWidth,
    required this.imageCount,
  });

  final String description;
  final int revision;
  final double screenWidth;
  final int imageCount;
}

class ListingEditorController extends ChangeNotifier {
  ListingEditorController(this._api);

  final ListingApi _api;

  ListingSourceState source =
      const ListingSourceState(description: '', revision: 0);

  ListingPresentationState presentation =
      const ListingPresentationState(
        preview: null,
        previewRevision: null,
        tags: [],
        tagsRevision: null,
        previewPending: false,
        tagsPending: false,
      );

  int _nextOperationId = 0;
  int _activePreviewOperationId = 0;
  int _activeTagsOperationId = 0;
  bool _disposed = false;

  void editDescription(String value, double screenWidth) {
    // Source-of-truth state changes immediately.
    final revision = source.revision + 1;
    source = ListingSourceState(description: value, revision: revision);

    presentation = ListingPresentationState(
      preview: presentation.preview,
      previewRevision: presentation.previewRevision,
      tags: presentation.tags,
      tagsRevision: presentation.tagsRevision,
      previewPending: true,
      tagsPending: true,
    );
    notifyListeners();

    _schedulePreview(
      PreviewJobInput(
        description: value,
        revision: revision,
        screenWidth: screenWidth,
        imageCount: 12,
      ),
    );
    _loadTags(description: value, revision: revision);
  }

  void _schedulePreview(PreviewJobInput input) {
    final operationId = ++_nextOperationId;
    _activePreviewOperationId = operationId;

    // compute-style boundary: input/output are plain data, no BuildContext.
    compute(buildListingPreview, input).then((result) {
      _commitPreview(
        operationId: operationId,
        revision: input.revision,
        result: result,
      );
    }).catchError((error) {
      _commitPreviewFailure(
        operationId: operationId,
        revision: input.revision,
        error: error,
      );
    });
  }

  Future<void> _loadTags({
    required String description,
    required int revision,
  }) async {
    final operationId = ++_nextOperationId;
    _activeTagsOperationId = operationId;

    try {
      final tags = await _api.suggestTags(description);
      _commitTags(
        operationId: operationId,
        revision: revision,
        tags: tags,
      );
    } catch (error) {
      _commitTagsFailure(
        operationId: operationId,
        revision: revision,
        error: error,
      );
    }
  }

  void _commitPreview({
    required int operationId,
    required int revision,
    required ListingPreview result,
  }) {
    if (!_acceptsPreview(operationId, revision)) return;

    presentation = ListingPresentationState(
      preview: result,
      previewRevision: revision,
      tags: presentation.tags,
      tagsRevision: presentation.tagsRevision,
      previewPending: false,
      tagsPending: presentation.tagsPending,
    );
    notifyListeners();
  }

  void _commitTags({
    required int operationId,
    required int revision,
    required List<String> tags,
  }) {
    if (!_acceptsTags(operationId, revision)) return;

    presentation = ListingPresentationState(
      preview: presentation.preview,
      previewRevision: presentation.previewRevision,
      tags: tags,
      tagsRevision: revision,
      previewPending: presentation.previewPending,
      tagsPending: false,
    );
    notifyListeners();
  }

  bool _acceptsPreview(int operationId, int revision) {
    return !_disposed &&
        operationId == _activePreviewOperationId &&
        revision == source.revision;
  }

  bool _acceptsTags(int operationId, int revision) {
    return !_disposed &&
        operationId == _activeTagsOperationId &&
        revision == source.revision;
  }

  void _commitPreviewFailure({
    required int operationId,
    required int revision,
    required Object error,
  }) {
    if (!_acceptsPreview(operationId, revision)) return;
    presentation = ListingPresentationState(
      preview: presentation.preview,
      previewRevision: presentation.previewRevision,
      tags: presentation.tags,
      tagsRevision: presentation.tagsRevision,
      previewPending: false,
      tagsPending: presentation.tagsPending,
      error: '$error',
    );
    notifyListeners();
  }

  void _commitTagsFailure({
    required int operationId,
    required int revision,
    required Object error,
  }) {
    if (!_acceptsTags(operationId, revision)) return;
    presentation = ListingPresentationState(
      preview: presentation.preview,
      previewRevision: presentation.previewRevision,
      tags: presentation.tags,
      tagsRevision: presentation.tagsRevision,
      previewPending: presentation.previewPending,
      tagsPending: false,
      error: '$error',
    );
    notifyListeners();
  }

  @override
  void dispose() {
    _disposed = true;
    super.dispose();
  }
}
```

```dart
Future<ListingPreview> buildListingPreview(PreviewJobInput input) async {
  final parsed = ListingParser.parseMarkdown(input.description);
  final warnings = ListingLinter.findWarnings(parsed);
  final layout = PreviewLayoutBuilder.build(
    parsed,
    imageCount: input.imageCount,
    screenWidth: input.screenWidth,
  );

  return ListingPreview(
    sourceText: input.description,
    parsed: parsed,
    warnings: warnings,
    layout: layout,
  );
}
```

```dart
class ListingEditor extends StatelessWidget {
  const ListingEditor({super.key, required this.controller});

  final ListingEditorController controller;

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;

    return ListenableBuilder(
      listenable: controller,
      builder: (context, _) {
        final source = controller.source;
        final presentation = controller.presentation;

        return Column(
          children: [
            TextField(
              onChanged: (value) =>
                  controller.editDescription(value, width),
            ),
            if (presentation.error != null) Text(presentation.error!),
            TagChips(
              tags: presentation.tags,
              pending: presentation.tagsPending,
              stale: presentation.tagsRevision != source.revision,
            ),
            Expanded(
              child: ListingPreviewPane(
                preview: presentation.preview,
                pending: presentation.previewPending,
                stale: presentation.previewRevision != source.revision,
              ),
            ),
          ],
        );
      },
    );
  }
}
```

## Resulting Ownership

- `ListingSourceState` owns immediate truth: current description and revision.
- `ListingPresentationState` owns lagging preview/tags plus pending and stale markers.
- `ListingEditorController` owns freshness: operation ids, revision checks, disposal checks, and presentation commits.
- `buildListingPreview` owns only expensive derivation and returns plain data.
- The widget owns layout and immediate event forwarding, not async result commits.

## Behavior Contracts To Test

- `editDescription` increments `source.revision` and updates `source.description` before any async result completes.
- Preview result A cannot commit after preview result B when B belongs to a newer revision.
- Tag result A cannot overwrite tags for a newer description.
- Preview failures for stale revisions are ignored.
- Presentation can show stale preview and pending state while source text continues to update immediately.
