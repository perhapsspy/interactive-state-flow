# Source Summary

Source file evaluated: `app/static/js/admin-toast-editor.js`

The file initializes a Toast UI markdown editor inside `.editSection`, seeds it from the nearest form textarea, and mirrors every editor `change` back into that textarea with `editor.getMarkdown()`. It also registers `addImageBlobHook` so dropped or inserted image blobs are uploaded to `/api/image/` through `XMLHttpRequest`.

## Interaction Path Evaluated

Path: user inserts an image blob into the editor -> Toast UI calls `imageUploadHook(blob, callback)` -> the hook builds `FormData` and sends a `POST /api/image/` request -> the XHR `load` handler parses the response -> the hook calls `callback(data['image']['full'], fileName)` to insert the uploaded image into the editor.

This is an interactive async path because the user may continue editing, leave the page, submit the form, or insert multiple images while uploads are still in flight.
