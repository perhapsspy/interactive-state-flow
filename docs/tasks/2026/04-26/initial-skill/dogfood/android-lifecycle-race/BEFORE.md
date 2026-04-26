# Before

Problem: a photo detail screen lets the user swipe between photos. Each swipe should update the selected photo immediately, then load metadata and a blurred preview. In this version, selection, network IO, expensive bitmap work, lifecycle checks, and UI commits are tangled in one handler.

```kotlin
class PhotoDetailFragment : Fragment() {
    private val adapter = PhotoPagerAdapter()
    private var selectedPhotoId: String? = null
    private var currentPhoto: PhotoUi? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        pager.adapter = adapter

        pager.onPageSelected = { index ->
            val photoId = adapter.photoIdAt(index)

            lifecycleScope.launch {
                // Source state is delayed behind work intended only for presentation.
                loadingSpinner.isVisible = true

                val photo = repository.fetchPhoto(photoId)
                val bitmap = imageLoader.loadFullSize(photo.url)
                val palette = withContext(Dispatchers.Default) {
                    Palette.from(bitmap).generate()
                }
                val blurred = withContext(Dispatchers.Default) {
                    blurRenderer.render(bitmap, palette)
                }

                selectedPhotoId = photoId
                currentPhoto = PhotoUi(photo, blurred, palette)

                // A weak lifecycle check does not prove this result still belongs to
                // the current page, view instance, or session.
                if (isAdded && viewLifecycleOwner.lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
                    title.text = photo.title
                    preview.setImageBitmap(blurred)
                    background.setColor(palette.getDominantColor(Color.BLACK))
                    loadingSpinner.isVisible = false
                }
            }
        }
    }
}
```

Failure modes:

- Swipe to photo A, then B. If A finishes last, A can overwrite B.
- `selectedPhotoId` is not updated until after IO and rendering complete, so other screen logic reads stale source state.
- Work continues after the view is recreated or the user navigates away.
- Background work returns directly to UI mutation instead of going through an owner that can decide freshness and usefulness.
- Presentation is all-or-nothing: no explicit pending or stale display state.
