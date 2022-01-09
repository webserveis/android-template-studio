# Load data from static source (assets)

Obtener datos estáticos desde assets

## Data Models

Archivo `dataproviders\MyItemDTO.kt`

```kotlin
import com.google.gson.annotations.SerializedName  
import java.util.*  
  
data class MyItemDTO(  
  
  @SerializedName("uid")  
  val uid: String,  
  
  @SerializedName("category")  
  val category: String,  
  
  @SerializedName("title")  
  val title: String,  
  
  @SerializedName("summary")  
  val summary: String?,  
  
  @SerializedName("javascript")  
  val javascript: String?,  
  
  @SerializedName("typescript")  
  val typescript: String?,  
  
  @SerializedName("sample")  
  val sample: String?,  
  
  @SerializedName("ctime")  
  val ctime: Date,  
  
  @SerializedName("mtime")  
  val mtime: Date,  
)
```

Archivo `models\MyItem.kt`
```kotlin
data class MyItem(  
  val uid: String,  
  val category: String,  
  val title: String,  
  val summary: String?,  
  val javascript: String?,  
  val typescript: String?,  
  val sample: String?,  
  val ctime: Date,  
  val mtime: Date  
)
```

Archivo `ui\main\MyItemUI.kt`
```kotlin
data class MyItemUI(  
  val uid: String,  
  val category: String,  
  val title: String,  
  val summary: String?,  
  val javascript: String?,  
  val typescript: String?,  
  val sample: String?  
)
```
## Data Mapper
Archivo `models\DataMapper.kt`
```kotlin
object DataMapper {  
  
  fun MyItemDTO.toMyItem(): MyItem {  
  return MyItem(  
  uid,  
  category,  
  title,  
  summary,  
  javascript,  
  typescript,  
  sample,  
  ctime,  
  mtime  
  )  
 }  
  fun MyItem.toMyItemUI(): MyItemUI {  
  return MyItemUI(  
  uid,  
  category,  
  title,  
  summary,  
  javascript,  
  typescript,  
  sample  
  )  
 }  
}
```
## Data loader
Archivo `dataproviders\LocalSource.kt`

 ```kotlin
 import android.content.Context  
import com.google.gson.GsonBuilder  
import com.google.gson.reflect.TypeToken  
import java.io.IOException  
  
class LocalSource(val context: Context) {  
  
  private fun getJsonDataFromAsset(context: Context, fileName: String): String? {  
  val jsonString: String  
  try {  
  jsonString = context.assets.open(fileName).bufferedReader().use { it.readText() }  
  } catch (ioException: IOException) {  
  ioException.printStackTrace()  
  return null  
  }  
  return jsonString  
  }  
  
  fun getJSON(fileName: String): List<MyItemDTO> {  
  val jsonString = getJsonDataFromAsset(context, fileName)  
  
  val gson = GsonBuilder()  
 .setDateFormat("yyyy-MM-dd HH:mm:ss")  
 .create()  
  
  val listDataType = object : TypeToken<List<MyItemDTO>>() {}.type  
  val dataList: List<MyItemDTO> = gson.fromJson(jsonString, listDataType)  
  
  return dataList  
  }  
  
}
 ```
## Repository
Archivo `repository\MyRepository.kt`

```kotlin
class MyRepository(private val staticSource: LocalSource) {  
  
  private val itemList : MutableList<MyItem> = mutableListOf()  
  
  companion object {  
  private const val TAG = "MyRepository"  
  }  
  
  suspend fun getItemList(): Flow<List<MyItem>> {  
  return flow {  
  delay(1000L)  
  val sourceList = staticSource.getJSON("js_oneliners.json")  
  val targetList = sourceList.map { it.toMyItem() }  
  
  Log.d(TAG, "getDataList() called" + sourceList.size + " == " + targetList.size)  
  //throw IndexOutOfBoundsException()  
  if (itemList.isEmpty()) itemList.addAll(targetList)  
  emit(targetList)  
  
  }  
  }  
  
  suspend fun getItemByUID(_uid:String): Flow<MyItem?> {  
  return flow {  
  delay(1000L)  
  
  if (itemList.isEmpty()) {  
  val sourceList = staticSource.getJSON("js_oneliners.json")  
  itemList.addAll(sourceList.map { it.toMyItem()})  
 }  
  val item: MyItem? = itemList.find { it.uid == _uid }  
  //throw IndexOutOfBoundsException()  
  emit(item)  
  
  }  
  }  
  
}
```
## ResulTypes
Archivo `ui\main\ResultTypeItemList.kt`
```kotlin
sealed class ResultTypeItemList {  
  object Loading : ResultTypeItemList()  
  data class Success(val data: List<MyItemUI>) : ResultTypeItemList()  
  data class Error(val exception: Throwable): ResultTypeItemList()  
}
```
Archivo `ui\main\ResultTypeItem.kt`
```kotlin
sealed class ResultTypeItem {  
  object Loading : ResultTypeItem()  
  data class Success(val data: MyItemUI?) : ResultTypeItem()  
  data class Error(val exception: Throwable): ResultTypeItem()  
}
```


## ViewModel
Archivo `ui\main\MyViewModel.kt`

```kotlin
class MyViewModel(application: Application) : AndroidViewModel(application) {  
  
  companion object {  
  private const val TAG = "SpoilerViewModel"  
  }  
  
  val repository: MyRepository  
  
  private val _uiItemListState: MutableStateFlow<ResultTypeItemList> = MutableStateFlow(ResultTypeItemList.Loading)  
  val uiItemListState: MutableStateFlow<ResultTypeItemList> = _uiItemListState  
  
  private val _uiItemState: MutableStateFlow<ResultTypeItem> = MutableStateFlow(ResultTypeItem.Loading)  
  val uiItemState: MutableStateFlow<ResultTypeItem> = _uiItemState  
  
  
  init {  
  Log.d(TAG, "init ViewModel ")  
  //fetchTaskContent()  
  val localSource = LocalSource(getApplication())  
  repository = MyRepository(localSource)  
  
  getItemList()  
 }  
  fun getItemList() {  
  Log.d(TAG, "getDataList: ")  
  //repository.getDataList()  
  itemListUserCase()  
 }  
  private fun itemListUserCase() {  
  viewModelScope.launch {  
  repository.getItemList()  
 .flowOn(Dispatchers.IO)  
 .onStart {  
  _uiItemListState.value = ResultTypeItemList.Loading  
  }  
  .catch { e ->  
  _uiItemListState.value = ResultTypeItemList.Error(e)  
  }  
  .distinctUntilChanged()  
 .collect {  
  val itemList = it.map { it.toMyItemUI() }  
  _uiItemListState.value = ResultTypeItemList.Success(itemList)  
  }  
 }  }  
  
  fun getItemByUID(uid: String) {  
  //repository.getDataList()  
  itemUserCase(uid)  
 }  
  private fun itemUserCase(uid: String) {  
  viewModelScope.launch {  
  repository.getItemByUID(uid)  
 .flowOn(Dispatchers.IO)  
 .onStart {  
  _uiItemState.value = ResultTypeItem.Loading  
  }  
  .catch { e ->  
  _uiItemState.value = ResultTypeItem.Error(e)  
  }  
  .distinctUntilChanged()  
 .collect {  
  val itemList = it?.toMyItemUI()  
  _uiItemState.value = ResultTypeItem.Success(itemList)  
  }  
 }  }  
  
}
```

## Observers
Archivo `ui\main\MyFragment.kt`

```kotlin
//sample
initSubscribers()  
mViewModel.getItemByUID("ca412c9a-7083-3185-7c25-06f805951957")
...
private fun initSubscribers() {  
  
  viewLifecycleOwner.lifecycleScope.launch {  
  mViewModel.uiItemListState.collect {  
  
  when (it) {  
  is ResultTypeItemList.Loading -> {  
  Log.i(TAG, "setupObservers: LOADING")  
 }  is ResultTypeItemList.Error -> {  
  Log.e(TAG, "setupObservers: ERROR" + it.exception)  
 }  is ResultTypeItemList.Success -> {  
  val data = it.data  
  //Log.d(TAG, "setupObservers: SUCCESS $data")  
  }  
  
 }  }  
  
 }  
  viewLifecycleOwner.lifecycleScope.launch {  
  mViewModel.uiItemState.collect {  
  
  when (it) {  
  is ResultTypeItem.Loading -> {  
  Log.i(TAG, "setupObservers: LOADING")  
 }  is ResultTypeItem.Error -> {  
  Log.e(TAG, "setupObservers: ERROR" + it.exception)  
 }  is ResultTypeItem.Success -> {  
  val data = it.data  
  Log.d(TAG, "setupObservers: SUCCESS $data")  
 }  
 }  }  
  
 }  
}
```
