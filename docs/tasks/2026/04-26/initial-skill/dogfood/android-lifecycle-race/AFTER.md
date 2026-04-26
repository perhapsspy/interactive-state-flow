# After

Refactor shape: the screen records selection immediately, owns a detail session token, cancels obsolete work, and accepts presentation results only through a freshness gate.

```kotlin
data class DetailSourceState(
    val selectedPhotoId: String,
    val generation: Long
)

data class DetailPresentationState(
    val photoId: String,
    val title: String?,
    val preview: Bitmap?,
    val dominantColor: Int?,
    val isPending: Boolean,
    val isStale: Boolean
)

class PhotoDetailViewModel(
    private val repository: PhotoRepository,
    private val imageLoader: ImageLoader,
    private val blurRenderer: BlurRenderer,
) : ViewModel() {
    private var nextGeneration = 0L
    private var activeJob: Job? = null

    private val _source = MutableStateFlow<DetailSourceState?>(null)
    val source: StateFlow<DetailSourceState?> = _source

    private val _presentation = MutableStateFlow<DetailPresentationState?>(null)
    val presentation: StateFlow<DetailPresentationState?> = _presentation

    fun selectPhoto(photoId: String, visible: Boolean) {
        val generation = ++nextGeneration

        // User intent and source truth are committed immediately.
        _source.value = DetailSourceState(
            selectedPhotoId = photoId,
            generation = generation
        )

        // Presentation may lag, but it is explicit and tied to the same owner identity.
        _presentation.value = DetailPresentationState(
            photoId = photoId,
            title = _presentation.value?.takeIf { it.photoId == photoId }?.title,
            preview = _presentation.value?.takeIf { it.photoId == photoId }?.preview,
            dominantColor = _presentation.value?.takeIf { it.photoId == photoId }?.dominantColor,
            isPending = true,
            isStale = _presentation.value?.photoId != photoId
        )

        activeJob?.cancel()

        if (!visible) return

        activeJob = viewModelScope.launch {
            val result = runCatching {
                val photo = repository.fetchPhoto(photoId)
                val bitmap = imageLoader.loadFullSize(photo.url)
                val prepared = withContext(Dispatchers.Default) {
                    val palette = Palette.from(bitmap).generate()
                    val blurred = blurRenderer.render(bitmap, palette)
                    PreparedPhotoPresentation(
                        photoId = photoId,
                        generation = generation,
                        title = photo.title,
                        preview = blurred,
                        dominantColor = palette.getDominantColor(Color.BLACK)
                    )
                }
                prepared
            }

            result.getOrNull()?.let(::commitPreparedPresentation)
        }
    }

    private fun commitPreparedPresentation(prepared: PreparedPhotoPresentation) {
        val current = _source.value ?: return

        // Freshness owner: only the ViewModel can decide whether async output belongs
        // to the current selection generation.
        if (prepared.photoId != current.selectedPhotoId) return
        if (prepared.generation != current.generation) return

        _presentation.value = DetailPresentationState(
            photoId = prepared.photoId,
            title = prepared.title,
            preview = prepared.preview,
            dominantColor = prepared.dominantColor,
            isPending = false,
            isStale = false
        )
    }

    override fun onCleared() {
        activeJob?.cancel()
    }
}

class PhotoDetailFragment : Fragment() {
    private val viewModel: PhotoDetailViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        pager.onPageSelected = { index ->
            val photoId = adapter.photoIdAt(index)
            viewModel.selectPhoto(
                photoId = photoId,
                visible = viewLifecycleOwner.lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)
            )
        }

        viewLifecycleOwner.lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                launch {
                    viewModel.source.collect { source ->
                        selectedBadge.text = source?.selectedPhotoId.orEmpty()
                    }
                }

                launch {
                    viewModel.presentation.collect { state ->
                        if (state == null) return@collect
                        loadingSpinner.isVisible = state.isPending
                        title.text = state.title ?: "Loading"
                        preview.alpha = if (state.isStale) 0.6f else 1.0f
                        state.preview?.let(preview::setImageBitmap)
                        state.dominantColor?.let(background::setColor)
                    }
                }
            }
        }
    }
}

private data class PreparedPhotoPresentation(
    val photoId: String,
    val generation: Long,
    val title: String,
    val preview: Bitmap,
    val dominantColor: Int
)
```

Contracts to test:

- Selecting B immediately updates `source.selectedPhotoId`, even if A is still loading.
- If A starts before B but finishes after B, A does not change `presentation`.
- `activeJob.cancel()` prevents obsolete work from committing.
- The fragment renders collected state only while started; async jobs never call view methods directly.
- Pending or stale presentation is explicit while the current preview catches up.
