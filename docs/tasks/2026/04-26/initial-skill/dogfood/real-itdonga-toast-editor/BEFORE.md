# Before

Relevant pseudocode from `app/static/js/admin-toast-editor.js`:

```js
function imageUploadHook(blob, callback) {
  let fileName = blob.name.replace(/\.[^/.]+$/, '');
  let formData = new FormData();
  formData.append('alt', fileName);
  formData.append('image', blob);
  formData.append('csrfmiddlewaretoken', document.querySelector('input[name="csrfmiddlewaretoken"]').value);

  let request = new XMLHttpRequest();
  request.addEventListener('load', function () {
    let data = JSON.parse(request.responseText);
    callback(data['image']['full'], fileName);
  });
  request.open('POST', '/api/image/', true);
  request.send(formData);
}
```

The editor setup also mirrors markdown immediately:

```js
editor.on('change', function () {
  textarea.value = editor.getMarkdown();
});
```

## Suspected Risks

- The source textarea update is immediate and simple; this path does not show an interactive-state-flow issue by itself.
- The image upload callback commits presentation/editor content after async IO with no freshness or ownership check.
- Multiple uploads can resolve in any order. That may be acceptable for independent image inserts, but the current hook has no operation identity, cancellation policy, or screen-lifetime check.
- If the editor is removed, the form is replaced, or the user navigates away while an upload is pending, the XHR `load` handler can still call the editor callback.
- JSON parse or response shape failures are not isolated from the interaction path; a bad response can throw inside the async handler.
