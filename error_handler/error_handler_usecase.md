Ejemplo de como interceptar errores y personalizados para su salida
Errores posibles al leer un archivo de texto plano, en caso de estar vacio se genere una excepción de `MalformedContentException`

## Entidad de Errores
```kotlin
sealed class ChatViewerFailure(val exception: Exception = Exception("Failure")) {
    object MalformedContent : ChatViewerFailure(Exception("Failure::Malformed Content"))
    data class Feature(private val _exception: Exception) : ChatViewerFailure(_exception)


    override fun equals(other: Any?): Boolean {
        return other is com.webserveis.app.whatschatviewer.AppFailure
    }

    override fun hashCode(): Int {
        return exception.hashCode()
    }
}

class MalformedContentException(message: String = "Malformed Content") : Exception(message)
```

## Entidad de Estado
```kotlin
sealed class DocParseState {
    object IDLE : DocParseState()
    class WORKING(val progress: Float = 0F) : DocParseState()
    class SUCCESS(val result: List<String>) : DocParseState()
    class FAILED(val error: ChatViewerFailure) : DocParseState()
}
```

## DataSource
Obtención del contenido del archivo
```kotlin
class DocFileSource private constructor(private val application: Application) {

    companion object : SingleArgSingletonHolder<DocFileSource, Application>(::DocFileSource)


    @Throws(IOException::class)
    fun readTextFromUri(uri: Uri): List<String> {
        val lines: MutableList<String> = arrayListOf()

        val input = BufferedReader(InputStreamReader(application.contentResolver.openInputStream(uri)), 8192)
        var strLine: String?
        while (input.readLine().also { strLine = it } != null) {
            strLine?.let {
                lines.add(it)
            }

        }
        input.close()
        return lines
    }

}

```

## Repositorio
```kotlin
class MyRepository private constructor(private val dataSource: DocFileSource) {

    companion object : SingleArgSingletonHolder<MyRepository, DocFileSource>(::MyRepository)

    fun openFileText(fileUri: Uri) = runCatching {
        dataSource.readTextFromUri(fileUri)
    }

}
```

## UseCase

```kotlin
class ParseChatFileUseCase(val state: MutableStateFlow<DocParseState>, private val myRepository: MyRepository) {

    data class Params(
        val fileUri: Uri
    )

    suspend operator fun invoke(params: Params) {

        myRepository.openFileText(params.fileUri)
            .onSuccess { data ->
                processData(data).flowOn(Dispatchers.IO)
                    .onStart { state.emit(DocParseState.WORKING()) }
                    .catch { e ->
                        state.emit(errorHandler(e))
                    }
                    .collect {
                        state.emit(DocParseState.SUCCESS(it))
                    }

            }.onFailure { e ->
                //state.emit(DocParseState.FAILED(ChatViewerFailure.Feature(e as Exception)))
                state.emit(errorHandler(e))

            }


    }

    private fun processData(dataList: List<String>): Flow<List<String>> {
        return flow {
            if (dataList.isEmpty()) throw MalformedContentException()
            delay(2000L)

            emit(dataList)
        }

    }

    private fun errorHandler(throwable: Throwable): DocParseState.FAILED {
        return when (throwable.cause) {
            is MalformedContentException -> {
                DocParseState.FAILED(ChatViewerFailure.MalformedContent)
            }
            else -> {
                DocParseState.FAILED(ChatViewerFailure.Feature(throwable as Exception))

            }
        }
    }
}

```

## ViewModel

```kotlin
class MyViewModel(private val application: Application) : ViewModel() {

    companion object {
        private const val TAG = "MyViewModel"
    }

  
    private val _state: MutableStateFlow<DocParseState> = MutableStateFlow(DocParseState.IDLE)
    val stateFlow: StateFlow<DocParseState> = _state

    private val myDataSource = DocFileSource.getInstance(application)
    private val myRepository = MyRepository.getInstance(myDataSource)
    private val myUseCase: ParseChatFileUseCase = ParseChatFileUseCase(_state, myRepository)

    init {

    }


    fun openChatFile(chatFile: Uri) {
        val params = ParseChatFileUseCase.Params(chatFile)
        viewModelScope.launch(Dispatchers.IO) {
            myUseCase(params)
        }
    }


}

```

### Observer del StateFlow
```kotlin
private fun initObservers() {
    viewLifecycleOwner.lifecycleScope.launch {
        viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
            mViewModel.stateFlow.collect {
                Log.d(TAG, "initObservers() called")
                when (it) {
                    is DocParseState.FAILED -> {
                        Log.d(TAG, "DocParseState: FAILED")
                        errorHandler(it.error)
                    }
                    DocParseState.IDLE -> {
                        Log.d(TAG, "DocParseState: IDLE")
                    }
                    is DocParseState.SUCCESS -> {
                        Log.d(TAG, "DocParseState: SUCCESS")
                        Toast.makeText(requireContext(), "lines" + it.result.size, Toast.LENGTH_SHORT).show()
                        binding.stateLayout.content()
                    }
                    is DocParseState.WORKING -> {
                        Log.d(TAG, "DocParseState: WORKING")
                        binding.stateLayout.loading()
                    }
                }
            }
        }
    }
}
```

Handler Error y renderizado del error
```kotlin
private fun errorHandler(failure: ChatViewerFailure) {
    val localizedMessage = failure.exception.localizedMessage ?: "Unknown"

    when (failure) {
        is ChatViewerFailure.Feature -> {
            renderErrorUI(getString(R.string.error_title), localizedMessage, R.drawable.ic_baseline_error_24)
        }
        is ChatViewerFailure.MalformedContent -> {
            renderErrorUI(getString(R.string.error_malformed_content_title), getString(R.string.error_malformed_content_summary), R.drawable.ic_baseline_error_24)
        }
    }

}


private fun renderErrorUI(title: String, summary: String, @DrawableRes icon: Int? = null) {
    val view = binding.stateLayout.getView(StateLayout.STATE_ERROR)
    view?.run {
        findViewById<TextView>(R.id.error_title)?.text = title
        findViewById<TextView>(R.id.error_summary)?.text = summary
        if (icon != null) findViewById<ImageView>(R.id.error_icon)?.setImageResource(icon)
    }

    binding.stateLayout.error()
}
```
