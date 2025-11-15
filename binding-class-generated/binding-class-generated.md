## Генерация классов в DataBinding

DataBinding создает Binding-класс, в котором полями являются view и переменные из макета.

Все сгенерированные Binding-классы наследуются от ViewDataBinding класса.

Binding-класс генерируется для каждого файла разметки.

### создание binding-объекта
binding-объект надо создавать в момент создания разметки на экране, пока иерархия view не поменялась. 

1. inflate(inflater)

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding = MyLayoutBinding.inflate(layoutInflater)

    setContentView(binding.root)
}
```

2. inflate(inflater, viewGroup, attachToParent)

```kotlin
val binding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false)
```

3. bind(view)
Если разметка надута ранее, можно сделать binding из нее:

```kotlin
val binding = ActivityMainBinding.bind(viewRoot)
```

4. DataBindingUtil.bind(viewRoot)

```kotlin
val viewRoot = LayoutInflater.from(this).inflate(layoutId, parent, attachToParent)

val binding: ViewDataBinding? = DataBindingUtil.bind(viewRoot)
```

5. DataBindingUtil.inflate
Часто используется в фрагментах, адаптеров списков и тд:

```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)

// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

### DataBinding создает поля только для тех View, которые имеют id.

### Binding-объект содержит сеттеры и геттеры для переменных разметки

```xml
<data>
   <import type="android.graphics.drawable.Drawable"/>
   <variable name="user" type="com.example.User"/>
   <variable name="image" type="Drawable"/>
   <variable name="note" type="String"/>
</data>
```

Для каждой из этих переменных будет создан геттер и сеттер.

### ViewStub
ViewStub объекты изначально невидимы в макете. Когда они становятся видимыми, или их 
явным образом надувают через layoutInflater, они заменяются другой разметкой. 

ViewStub исчезают из иерархии view, также исчезают из binding-объекта. 

ViewStubProxy - можно найти в binding-объекте вместо ViewStub. Когда ViewStub еще 
существует, вернет ViewStub.

Когда ViewStub подменяется разметкой, вернет эту разметку. 

Можно установить ViewStubProxy.OnInflateListener(), чтобы сделать binding и для него.

### executePendingBindings()
В теории при изменении данных UI будет изменен на следующий кадр. 

Но если надо сделать привязку данных немедленно, можно вызвать этот метод.

### BR-класс
BR класс генерируется в пакете модуля. Он содержит id ресурсов, используемых в привязке данных. 

В этом примере recyclerView может юзать разные макеты под капотом. Но у всех 
макетов есть переменная item, которую можно задать в onBindViewHolder()
```kotlin
override fun onBindViewHolder(holder: BindingHolder, position: Int) {
    item: T = items.get(position)
    holder.binding.setVariable(BR.item, item);
    holder.binding.executePendingBindings();
}
```

### Кастомное название binding-класса
MainActivity -> ActivityMainBinding

Классы binding лежат в app/src/main/com/company/appname/databinding/

У тега data есть атрибут class - он определяет пакет и название binding класса.

```xml
<data class="com.example.ContactItem">
    ...
</data>
```
