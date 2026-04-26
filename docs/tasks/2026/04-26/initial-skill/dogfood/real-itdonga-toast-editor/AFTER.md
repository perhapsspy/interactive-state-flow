# After

Keep the immediate markdown mirroring as-is. It records current editor source state promptly and does not appear expensive enough to defer.

Refactor the image upload path so the editor loader owns upload freshness and screen lifetime. The hook should still be small, but completed uploads should pass through a guard before calling Toast UI's `callback`.

```js
function createImageUploadHook(editSection) {
  let nextUploadId = 0;
  let activeUploads = new Set();

  function isEditorActive() {
    return document.body.contains(editSection);
  }

  return function imageUploadHook(blob, callback) {
    let uploadId = ++nextUploadId;
    activeUploads.add(uploadId);

    let fileName = blob.name.replace(/\.[^/.]+$/, '');
    let formData = new FormData();
    formData.append('alt', fileName);
    formData.append('image', blob);
    formData.append('csrfmiddlewaretoken', document.querySelector('input[name="csrfmiddlewaretoken"]').value);

    let request = new XMLHttpRequest();

    request.addEventListener('load', function () {
      activeUploads.delete(uploadId);

      if (!isEditorActive()) {
        return;
      }

      let data;
      try {
        data = JSON.parse(request.responseText);
      } catch (error) {
        return;
      }

      if (!data.image || !data.image.full) {
        return;
      }

      callback(data.image.full, fileName);
    });

    request.addEventListener('error', function () {
      activeUploads.delete(uploadId);
    });

    request.open('POST', '/api/image/', true);
    request.send(formData);
  };
}

function toastEditorLoader() {
  let editSection = document.querySelector('.editSection');
  let formRow = editSection.closest('.form-row');
  let textarea = formRow.querySelector('textarea');
  let editor = new toastui.Editor({
    el: editSection,
    initialEditType: 'markdown',
    initialValue: textarea.value,
    previewStyle: 'vertical',
    height: '600px',
    exts: ['scrollSync'],
    language: 'ko_KR',
    hooks: {'addImageBlobHook': createImageUploadHook(editSection)}
  });

  editor.on('change', function () {
    textarea.value = editor.getMarkdown();
  });
}
```

This keeps the codebase's plain JavaScript style. It adds a small owner for async image insertion, guards the commit by current screen/editor presence, and prevents malformed responses from throwing through the upload completion path.

If this admin page later gets dynamic editor teardown, the same owner can grow one explicit cleanup method that aborts pending XHRs and clears `activeUploads`.
