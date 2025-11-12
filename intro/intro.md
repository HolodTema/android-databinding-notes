## Введение в data binding Android

DataBinding позволяет привязать UI компоненты в разметке к источникам данных
декларативно.

Если раньше мы внутри активности писали подобный код
```kotlin
findViewById<TextView>(R.id.sample_text).apply {

    text = viewModel.userName

}
```

то теперь можно писать прямо в разметке

```xml
<TextView
    android:text="@{viewmodel.userName}" />
```

DataBinding освобождает код активностей и фрагментов от обслуживания View. 

- код активностей и фрагментов становится проще и чище

- улучшает производительность приложения

- предотвращает возможные утечки памяти и баги, ибо работает под капотом

### View Binding
В простых проектах ViewBinding приносит то же самое, что и DataBinding, 
но при этом он намного проще и производительней. 

Если наша цель - просто убрать findViewById, то лучше юзать ViewBinding

### Как подключить DataBinding
в build.gradle (app):

```kotlin
android {
    buildFeatures {
        dataBinding = true
    }
}
```

И тогда будет работать подсветка синтаксиса, автокомплит кода и тд...

### DataBinding и Layout editor preview
При создании разметки на нее вид можно посмотреть в layout editor preview.

Там отобразится значение default, если оно задано:

```xml
<TextView android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName, default=my_default}" />
```


### data binding expressions
Внутри xml разметки пишем выражения, привязывающие объекты kotlin к аттрибуту xml.

+ определяется, как выглядит data binding в xml. Теги layout, data, variable ...

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>

    <ConstraintLayout... /> <!-- UI layout's root element -->
</layout>
```

### DataBinding и observable
DataBinding имеет классы и методы для отслеживания изменений данных.

При изменении источника данных, UI обновится автоматически.

Переменные или свойства, объекты можно сделать observable.

### DataBinding генерирует Binding классы
Они используются для доступа к переменным в макете и к view макета.

### BindingAdapter
Для каждого databinding expression внутри разметки можно создать адаптер привязки.

Адаптер привязки позволяет запускать дополнительный код при обновлении UI и при
установке конкретного атрибута.

```kotlin
@BindingAdapter("app:goneUnless")
fun goneUnless(view: View, visible: Boolean) {

    view.visibility = if (visible) View.VISIBLE else View.GONE

}
```

Есть еще встроенные адаптеры для разных атрибутов.

### двусторонний (two-way) databinding
Двусторонняя привязка данных.

При изменении источника данных UI обновится автоматически.

При изменении данных через UI данные автоматически обновлятся в источнике данных.