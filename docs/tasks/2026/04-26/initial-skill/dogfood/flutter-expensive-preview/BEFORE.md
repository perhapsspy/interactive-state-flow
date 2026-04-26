# Before: Tangled Flutter Preview State

This screen lets a user edit a product description and immediately see a rendered marketplace preview with generated tags, warnings, and a layout model. The preview generation is CPU-heavy enough to belong off the urgent text-input path, and tag suggestions come from async IO.

```dart
class ListingEditor extends StatefulWidget {
  const ListingEditor({super.key});

  @override
  State<ListingEditor> createState() => _ListingEditorState();
}

class _ListingEditorState extends State<ListingEditor> {
  final _controller = TextEditingController();

  String _description = '';
  ListingPreview? _preview;
  List<String> _suggestedTags = [];
  bool _busy = false;
  String? _error;

  Future<void> _onDescriptionChanged(String value) async {
    setState(() {
      _busy = true;
      _description = value;
      _error = null;
    });

    // This blocks the interaction path for large descriptions/images.
    final parsed = ListingParser.parseMarkdown(value);
    final warnings = ListingLinter.findWarnings(parsed);
    final layout = PreviewLayoutBuilder.build(
      parsed,
      imageCount: 12,
      screenWidth: MediaQuery.of(context).size.width,
    );

    // Async result can return after a newer keystroke and overwrite the UI.
    final tags = await ListingApi.suggestTags(value);

    if (!mounted) return;
    setState(() {
      _suggestedTags = tags;
      _preview = ListingPreview(
        sourceText: value,
        parsed: parsed,
        warnings: warnings,
        layout: layout,
      );
      _busy = false;
    });
  }

  Future<void> _save() async {
    setState(() => _busy = true);
    try {
      await ListingApi.saveDraft(
        description: _description,
        tags: _suggestedTags,
        previewJson: _preview?.toJson(),
      );
    } catch (e) {
      setState(() => _error = '$e');
    } finally {
      if (mounted) setState(() => _busy = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          controller: _controller,
          onChanged: _onDescriptionChanged,
        ),
        if (_busy) const LinearProgressIndicator(),
        if (_error != null) Text(_error!),
        TagChips(tags: _suggestedTags),
        Expanded(
          child: _preview == null
              ? const EmptyPreview()
              : ListingPreviewPane(preview: _preview!),
        ),
        FilledButton(onPressed: _save, child: const Text('Save')),
      ],
    );
  }
}
```

## Problems

- Source state, preview presentation, IO, CPU work, and save state all share `_busy`.
- Text input records source state, parses markdown, lints, builds layout, calls IO, and commits presentation in one handler.
- The actual source description is updated immediately, but the same urgent handler also performs expensive preview work.
- `suggestTags("old text")` can resolve after `suggestTags("new text")` and overwrite current tags.
- Generated preview is stored beside source state without a freshness owner.
- Background/isolate-worthy work is not separated from UI-owned commit logic.
- Save can serialize `_preview` that does not belong to the latest `_description`.
