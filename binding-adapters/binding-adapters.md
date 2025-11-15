## BindingAdapter
BindingAdapter позволяют вызывать дополнительный котлин-код во время установки значений в xml-разметки в databinding механизме.

Можно модифицировать метод, устанавливающий значения xml-атрибутов.

Когда значение, к которому привязан атрибут, меняется, binding-класс вызывает 
метод-сеттер для установки значения атрибута. Этот метод определяется автоматически, 
но его можно переопределить и сделать там дополнительную логику.

Если атрибут будет android:text, то сеттер будет setText(arg).

Причем setText() может иметь перегрузки в зависимости от типа параметра атрибута: строка, число и тд...

### собственные атрибуты

Можно создавать свои атрибуты, написав для них свои методы-сеттеры. 

Например, DrawerLayout имеет методы setScrimColor(), но не имеет атрибута scrimColor.

Но мы можем задать несуществующий атрибут app:scrimColor="", и это сработает, ибо есть 
сеттер в котлин-коде.

### Кастомное имя для сеттеров
Иногда некоторые атрибуты не имеют совпадающих по имени сеттеров.

Можно вручную связать метод-сеттер и xml-атрибут при помощи аннотации @BindingMethods.

Внутри @BindingMethods будет список объектов BindingMethod.

Например, android:tint ассоциируется с setImageTintList(ColorStateList) методом, хотя
его имя не совпадает.

```kotlin
@BindingMethods(value = [
    BindingMethod(
        type = android.widget.ImageView::class,
        attribute = "android:tint",
        method = "setImageTintList")])
```

### @BindingAdapter
Можно модифицировать сеттеры для атрибутов, чтобы добавить туда свой код.

Например, android:paddingLeft атрибут ассоциируется с setPadding(left, top, right, bottom). Сеттера с именем setLeftPadding() нету.

Можно создать сеттер setLeftPadding() при помощи аннотации @BindingAdapter

```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, padding: Int) {
    view.setPadding(padding,
                view.getPaddingTop(),
                view.getPaddingRight(),
                view.getPaddingBottom())
}
```

Первый аргумент - тип View, для которого сработает метод-сеттер.

Другой пример, можно сделать загрузку в фоновом потоке для атрибута imageUrl
через Picasso:

```kotlin
@BindingAdapter("imageUrl", "error")
fun loadImage(view: ImageView, url: String, error: Drawable) {

    Picasso.get().load(url).error(error).into(view)

}
```

И потом применить все это к картинке:

```xml
<ImageView
    app:imageUrl="@{venue.imageUrl}"
    app:error="@{@drawable/venueError}" />
```

### DataBinding игнорирует пространства имен xmlns (хз что это значит в доках)

### requireAll флаг
В примере выше сеттер срабатывает если заданы два атрибута одновременно: imageUrl и error. Можно сделать так, чтобы сеттер срабатывал, если установлен хотя бы один из атрибутов:

```kotlin
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {

    if (url == null) {
        imageView.setImageDrawable(placeholder);
    } else {
        MyImageLoader.loadInto(imageView, url, placeholder);
    }

}
```

