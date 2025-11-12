## Работа с observable объектами

Observable - объекты уведомляют других о том, что их поля изменились. 

DataBinding позволяет делать observable объекты, поля, коллекции.

Вообще для dataBinding можно юзать любые объекты данных. Но тогда изменения
в них не будут автоматически обновлять UI.

Поэтому dataBinding позволяет сделать объекты, поля и коллекции observable, чтобы 
UI обновлялся при изменении данных.

### Observable fields
-это observable объекты с одним полем внутри. Есть generic-версия для классов и версии
ObservableField для примитивов.

```
ObservableBoolean

ObservableByte

ObservableChar

ObservableShort

ObservableInt

ObservableLong

ObservableFloat

ObservableDouble

ObservableParcelable
```
Пример:
```kotlin
class User {
    val firstName = ObservableField<String>()

    val lastName = ObservableField<String>()

    val age = ObservableInt()
}
```

У observable-объектов есть set() и get() в java, но в kotlin мы просто обращаемся по имени.

```kotlin
user.firstName = "Bob"

val age = user.age
```

## observable коллекции

### ObservableArrayMap

ObservableArrayMap

```kotlin
ObservableArrayMap<String, Any>().apply {
    put("firstName", "Google")
    put("lastName", "Inc.")
    put("age", 17)
}
```
Тут мы находим значение в мапе по ключу как вызов mapName.key, а не через [].

Хотя и через [] не запрещено.

```xml
<data>

    <import type="android.databinding.ObservableMap"/>

    <variable name="user" type="ObservableMap&lt;String, Object&gt;"/>

</data>
…
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>

<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

### ObservableArrayList
```kotlin
ObservableArrayList<Any>().apply {
    add("Google")
    add("Inc.")
    add(17)
}
```

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>

    <variable name="user" type="ObservableList&lt;Object&gt;"/>
</data>
…
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>

<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

### observable objects

Класс, который реализует Observable, позволяет регистрировать слушатели, срабатывающие
на изменение данных в observable-объекте.

listeners можно добавлять и удалять.

BaseObservable - класс, реализующий добавление и удаление listeners.

Нужно унаследовать класс данных от BaseObservable, а потом аннотировать
геттеры полей как Bindable. И при изменении данных в сеттере вызывать
notifyPropertyChanged().

```kotlin
class User : BaseObservable() {

    @get:Bindable
    var firstName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.firstName)
        }

    @get:Bindable
    var lastName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.lastName)
        }
}
```

DataBinding генерирует класс BR (binding references). В нем содержатся айдишники
ресурсов для привязки данных.

@Bindable-аннотация генерирует новое поле в BR-классе.

### LiveData
В качестве observable предпочтительно юзать LiveData, ибо она отменяет все свои слушатели, когда активность или фрагмент уходят из resumed состояния.

Хотя есть observeForever() слушатель, который надо вручную снимать.

Подробнее написано позже будет.

### StateFlow
StateFlow + Coroutines хорошая связка для источника данных для DataBinding.

Обычно StateFlow используется вместе с LifecycleOwner, например activity или
viewLifecycleOwner.

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.lifecycleOwner = this
    }
}
```

```kotlin
class ScheduleViewModel : ViewModel() {

    private val _username = MutableStateFlow<String>("")

    val username: StateFlow<String> = _username

    init {
        viewModelScope.launch {
            _username.value = Repository.loadUserName()
        }
    }
}
```

```xml
<TextView
    android:id="@+id/name"
    android:text="@{viewmodel.username}" />
```

Здесь мы видим, как данные загружаются в StateFlow при инициализации viewModel.


И далее UI будет автоматом обновляться при обновлении данных в StateFlow.

### github репозиторий с примером databinding от google

https://github.com/android/databinding-samples