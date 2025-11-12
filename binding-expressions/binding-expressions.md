## Binding expressions в xml-разметке

Внутри разметки пишем выражения, обрабатывающие события view.

DataBinding автоматически генерирует классы для связывания view и источников данных.

Чтобы добавить DataBinding в разметку, надо запаковать ее в layout-тег.

Внутри layout тега идут data-тег и корневой жлемент view.

Внутри data-тега определяются переменные 

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

   <data>
       <variable name="user" type="com.example.User"/>
   </data>

   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>

       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>

   </LinearLayout>

</layout>
```

Оператор @{} - внутри него пишут databinding expression.

### Простой пример databinding
Есть простой класс User, с неизменяемыми полями.

```kotlin
data class User(
    val firstName: String,
    val lastName: String
)
```

для каждого файла разметки генерируется binding-класс, содержащий внутри view и переменные этой разметки. Создаем объект этого binding-класса в onCreate.

Задав binding.user, имя пользователя автоматически появится в UI.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil
        .setContentView(this, R.layout.activity_main)

    binding.user = User("Test", "User")
}
```

Можно создать binding-объект по-другому, через layoutInflater.

Этот способ юзают в recyclerview, fragment, dialog и тд.

```kotlin
val binding = ActivityMainBinding.inflate(layoutInflater)
```

Эквивалентный способ:

```kotlin
val binding = DataBindingUtil.inflate(layoutInflater)
```

### Чего нету в databinding expression language
- нету this

- нету super

- нету new

- нету явного вызова generic

### Что есть в databinding expression language

- операторы логические, битовые, арифметические, сравнения

- instanceof

- скобки группировки ()

- индексы массива []

- explicit cast

- null

- string, literal, numeric

- доступ к полям и методам переменных

- тернарный оператор

- ?? null coalescing operator

### В xml уже юзают < скобка. Поэтому в data binding expression ее заменяют на &lt;

### ?? null coalescing operator
Это аналог ?: оператора в Kotlin для Nullable типов.

```xml
android:text="@{user.displayName ?? user.lastName}"
```

эквивалентно:

```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

### обращение к полю объекта
Когда мы пишем

```xml
android:text="@{user.lastName}"
```

Это может быть публичное поле, геттер, observableField.

### databinding expression избегает null pointer exception
Если результат databinding expression = null, то значением атрибута станет значение по умолчанию: 0, false, null.

### databinding expression language: обращение к другим view этого макета

```xml
<EditText
    android:id="@+id/example_text"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"/>

<TextView
    android:id="@+id/example_output"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{exampleText.text}"/>
```

Здесь textView автоматически подставляет текст из EditText.

### databinding expression и коллекции
Просто обращаемся по индексу

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
...

android:text="@{list[index]}"

android:text="@{sparse[index]}"

android:text="@{map[key]}"
```

### databinding expression и Map
Если мы хотим получить значение из мапы по ключу, можем писать через [].

А можем сделать mapObj.key

### Одиночные и двойные кавычки.

Для задания атрибута, xml уже юзает двойные кавычки. Значит, внутри
databinding expression надо юзать одиночные кавычки.

А можно сделать наоборот.

```xml
android:text="@{map[`firstName`]}"

android:text='@{map["firstName"]}'
```

### доступ к ресурсам приложения

Синтаксис прост.

И еще можно форматировать строки на лету:

И делать тернарный оператор и тд...

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"

android:text="@{@string/nameFormat(firstName, lastName)}"

android:text="@{@plurals/banana(bananaCount)}"
```

### обработка UI-событий

Большинство listener из Kotlin имеют свою xml-версию в виде атрибута.

Например, View.OnClickListener это onclick=""

```
android:onClick

android:onZoomOut

android:onZoomIn

android:onSearchClick

и тд
```

Можно привязать UI-события двумя путями

1. привязать метод или функцию к атрибуту. Если databinding expression вернет null,
то listener просто не будет создан. Причем listener-объект создается при привязке данных, а не при срабатывании события.

2. listener binding - это лямбда выражение внутри binding expression. И тогда листенер создается всегда и вызывается всегда. listener объект создается и срабатывает при срабатывании UI события.

### привязка метода к UI-событию

```kotlin
class MyHandlers {

    fun onClickFriend(view: View) { ... }

}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>

   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>

   </LinearLayout>

</layout>
```

Причем сигнатура onClickField() должна совпадать с сигнатурой метода в listener-интерфейсе.

### listener binding
```kotlin

class Presenter {
    fun onSaveClick(task: Task){}
}

```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>

    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">

        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />

    </LinearLayout>

</layout>
```

Вообще View.OnClickListener имеет параметр - view, но мы его не юзаем, поэтому у нас пустые скобки.

Другой пример:

```kotlin
class Presenter {

    fun onCompletedChanged(task: Task, completed: Boolean){}

}
```

```xml
<CheckBox
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
      android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

### импорты
Можно импортировать kotlin классы и юзать их в data binding expressions.

Можно задавать псевдонимы импортируемых классов при помощи alias.

```xml
<data>
    <import type="android.view.View" />
    ...
</data>

<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

Или можно сразу использовать импортированное в переменных:

```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>

    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>
```