---
layout: post
title: Swift Optionals
date: '2017-06-04 10:00:00 +0300'
tags: [swift]
published: true
---

#### **Что такое Optionals?**
_Optionals_ (опционалы) — это удобный механизм обработки ситуаций, когда значение может отсутствовать. Значение будет использовано, только если оно есть.

#### **Зачем нужны Optionals, когда есть проверка на nil?**
Во-первых, проверка на равенство/неравенство `nil` применима только к [nullable-типам](https://en.wikipedia.org/wiki/Nullable_type) и не применима к примитивным типам, структурам и перечислениям. Для обозначения отсутсвия значения у переменной примитивного типа приходится вводить спецзначения, такие как _NSNotFound_[^1]. Соответственно, пользователь этой переменной должен учитывать, что спецзначения возможны. В Swift даже примитивный тип можно использовать в опциональном стиле, т.е явным образом указывать на то, что значения может не быть.

Во-вторых: явная опциональность проверяется на этапе компиляции, что снижает количество ошибок в runtime. Опциональную переменную в Swift нельзя использовать точно так же, как неопциональную. Опционал нужно либо принудительно преобразовывать к обычному значению, либо использовать синтаксические конструкции с созданием новой переменной, такие как `if let`, `guard let` и `??`. Опционалы в Swift реализуют не просто проверку, но целую парадигму [опционального типа](https://en.wikipedia.org/wiki/Option_type) в [теории типов](https://en.wikipedia.org/wiki/Type_theory).

В-третьих, опционалы синтаксически более лаконичны, чем проверки на `nil`, что особенно хорошо видно на цепочках опциональных вызовов — так называемый _Optional Chaining_.

[^1]:[_NSNotFound_](https://developer.apple.com/reference/foundation/foundation_constants/nsnotfound) не только нужно рассматривать как спецзначение, но и следить, чтобы оно не входило в множество значений переменной. Ситуация усложняется еще и тем, что _NSNotFound_ считается равным _NSIntegerMax_, т.е. может иметь разные значения для разных (32-bit/64-bit) платформ. Это значит, что _NSNotFound_ нельзя напрямую записывать в файлы и архивы или использовать в [_Distributed Objects_](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/PreferencePanes/Tasks/Communication.html).


#### **Как это работает?**
Опционал в Swift представляет из себя особый объект-контейнер, который может содержать в себе либо `nil` либо объект конкретного типа, который указывается при объявлении этого контейнера. Эти два состояния обозначаются терминами _None_ и _Some_ соответственно. Если при создании опциональной переменной присваемое значение не указывать, то `nil` присваивается по умолчанию[^2]. Таким образом, запись `Int?` — это объявление контейнера, экземпляр которого может содержать внутри `nil`(состояние _None Int_) либо значение типа `Int` (состояние _Some Int_). Именно поэтому при преобразовании `Int?` в `Int` используетя термин _unwrapping_ вместо _cast_, т.е. подчеркивается "контейнерная" суть опционала. Лексема `nil` в Swift обозначает состояние _None_, которое можно присвоить любому опционалу. Это логично приводит к невозможности присвоить `nil` (состояние _None_) переменной, которая не является опционалом.

[^2]:В документации значение по умолчанию в случае отсутсвия явного присвоения не упоминается, но [сказано](https://developer.apple.com/documentation/swift/optional), что опционал _represents either a wrapped value or nil, the absence of a value_. Если опциональная переменная объявлена без явного присвоения (какое-либо `.some` не присваивалось), то логично следует, что неявно присваивается `.none` — третьего "неинициализрованного" состояния у перечисления `Optional` нет.

По факту опционал представляет собой системное перечисление:

{% highlight swift %}
public enum Optional<Wrapped> : ExpressibleByNilLiteral {

    /// The absence of a value.
    ///
    /// In code, the absence of a value is typically written using the `nil`
    /// literal rather than the explicit `.none` enumeration case.
    case none

    /// The presence of a value, stored as `Wrapped`.
    case some(Wrapped)

    /// Creates an instance that stores the given value.
    public init(_ some: Wrapped)

        ...

    /// Creates an instance initialized with `nil`.
    ///
    /// Do not call this initializer directly. It is used by the compiler
    // when you initialize an `Optional` instance with a `nil` literal.
    public init(nilLiteral: ())

    /// The wrapped value of this instance, unwrapped without checking whether
    /// the instance is `nil`.
    public var unsafelyUnwrapped: Wrapped { get }
}
{% endhighlight %}

Перечисление `Optional` имеет два возможных состояния: `.none` и `some(Wrapped)`. Запись `Wrapped?` обрабатывается препроцессором (_Swift’s type system_) и трансформируется в `Optional<Wrapped>`, т.е. следующие записи эквивалентны:
{% highlight swift %}
var my_variable: Int?
{% endhighlight %}
{% highlight swift %}
var my_variable: Optional<Int>
{% endhighlight %}

Лексема `nil` по факту обозначает `Optional.none`, т.е. следующие записи эквивалентны:
{% highlight swift %}
var my_variable: Int? = nil
{% endhighlight %}
{% highlight swift %}
var my_variable: Optional<Int> = Optional.none
{% endhighlight %}
{% highlight swift %}
var my_variable = Optional<Int>.none
{% endhighlight %}

Перечисление `Optional` имеет два конструктора. Первый конструктор `init(_ some: Wrapped)` принимает на вход значение соответсвующего типа, т.е. следующие записи эквивалентны:

{% highlight swift %}
var my_variable: Int? = 42
{% endhighlight %}
{% highlight swift %}
var my_variable = Optional<Int>(42) // явное указание типа для наглядности
{% endhighlight %}
{% highlight swift %}
var my_variable = Optional<Int>.some(42) // явное указание типа для наглядности
{% endhighlight %}

Второй конструктор `init(nilLiteral: ())` является реализацией протокола `ExpressibleByNilLiteral`
{% highlight swift %}
public protocol ExpressibleByNilLiteral {

    /// Creates an instance initialized with `nil`.
    public init(nilLiteral: ())
}
{% endhighlight %}
и инициализирует опциональную переменную состоянием `.none`. Этот конструктор используется только компилятором. Согласно [документации](https://developer.apple.com/documentation/swift/optional/1538243-init) его нельзя вызывать напрямую, т.е следующие записи приведут к ошибкам компиляции:
{% highlight swift %}
var my_variable1 = Optional<Int>(nil) // ошибка компиляции
var my_variable2 = Optional<Int>.none(nil)// ошибка компиляции
{% endhighlight %}
Вместо них следует использовать
{% highlight swift %}
var my_variable: Int? = nil // или var my_variable: Int? = Optional.none
{% endhighlight %}
или вообще не использовать явное присвоение
{% highlight swift %}
var my_variable: Int?
{% endhighlight %}
поскольку `nil`(`Optional.none`) будет присвоен по умолчанию.


  (_А ЧТО НАСЧЕТ ВОСКЛИЦАТЕЛЬНОГО?_).
Переменная unsafelyUnwrapped отвечает за ситуации, ... такие ситуации в Swift обозначаютсятся лексемой `!`


(_МОЖЕТ, СТОИТ УБРАТЬ УПОМИНАНИЯ ЭТИХ МЕТОДОВ map<U>— ОНИ НЕ ИМЕЕЮТ ОТНОШЕНИЯ К ТЕМЕ?_).
Еще как имеют.
https://www.natashatherobot.com/swift-using-map-to-deal-with-optionals/


{% highlight swift %}
let arrayOfOptionals: [Int?] = [1, nil, 5, 2, nil]
let arrayOfNumbers = arrayOfOptionals.flatMap { $0 }
print(arrayOfNumbers) // [1, 5, 2]
{% endhighlight %}

 _I'm talking about the map and flatMap functions of Swift optionals (not the Array map function)._
 http://blog.xebia.com/the-power-of-map-and-flatmap-of-swift-optionals/


(_кто-то даже рисовал свое перечисление_)
http://jamesonquave.com/blog/re-implementing-optionals-using-swifts-powerful-enum-type/



#### **Хватит теории, как пользоваться-то?**
#### **Идиомы использования**
[Using the Nil-Coalescing Operator](https://developer.apple.com/documentation/swift/optional)

конструкция iflet
конструкция guard

конструкция map/flatmap
http://blog.xebia.com/the-power-of-map-and-flatmap-of-swift-optionals/


На вход подаем трансформирующую функцию

{% highlight swift %}
public func flatMap<U>(_ transform: (Wrapped) throws -> U?) rethrows -> U?
{% endhighlight %}
которая принимает извлеченный элемент


{% highlight swift %}
public enum Optional<Wrapped> : ExpressibleByNilLiteral {

    ...

    /// Evaluates the given closure when this `Optional` instance is not `nil`,
    /// passing the unwrapped value as a parameter.
    ///
    /// Use the `map` method with a closure that returns a nonoptional value.
    /// - Parameter transform: A closure that takes the unwrapped value
    ///   of the instance.
    /// - Returns: The result of the given closure. If this instance is `nil`,
    ///   returns `nil`.
    public func map<U>(_ transform: (Wrapped) throws -> U) rethrows -> U?

    /// Evaluates the given closure when this `Optional` instance is not `nil`,
    /// passing the unwrapped value as a parameter.
    ///
    /// Use the `flatMap` method with a closure that returns an optional value.
    /// - Parameter transform: A closure that takes the unwrapped value
    ///   of the instance.  
    /// - Returns: The result of the given closure. If this instance is `nil`,
    ///   returns `nil`.
    public func flatMap<U>(_ transform: (Wrapped) throws -> U?) rethrows -> U?

        ...

}
{% endhighlight %}



#### i**mplicit unwrapping, forсe unwrapping, explicitly unwrap, [optional binding to deconstruct optionals] и еще много страшных слов**

принудительно преобразовывать к обычному значению
можно явно и неявно


#### **Ок, подводные грабли есть?**
??
как же без них:
один ? на возврат нуля из функции map, другой ? на возврат из свойства window. А тройной можно?

{% highlight swift %}
var value: AppDelegate?
var newValue = value.map { $0.window }
newValue??.makeKey()
{% endhighlight %}


#### **Optional Chaining**

#### **Связка с Objective-C**
В Objective-C нет понятия опциональности, близко по смыслу находится возможность вернуть`nil` из метода, который обычно возвращает объект. В Swift и в Objective-C лексема `nil` имеет различную смысловую нагрузку. В Swift `nil` — это _Nothing_, т.е. признак отсутствия значения в переменной optional-типа, применяется к любым optional-типам. В Objective-C `nil` является указателем на несуществующий объект и применим только к типам объектов.


В Objective-C есть nullability annotations `_Nullable` и `_Nonnull` и nullability-аттрибуты свойств `nullable`, `nonnull`, `null_unspecified` и `null_resettable`. Они введены исключительно для совместимости с optional-семантикой в Swift, в Objective-C ни на что не влияют (если не считать предупреждений компиляции, например, при попытке присвоить nil nonnull-свойству). Кроме того, эти аттрибуты не применимы к примитивным типам, структурам и перечислениям. Подробнее про nullability annotations [здесь](https://developer.apple.com/swift/blog/?id=25) и очень понятно про nullability-аттрибуты свойств [здесь](https://habrahabr.ru/post/265175/).
В чем разница между `_Nullable` и `nullable`?
`null_unspecified` вообще нигде не упоминается, кроме как в статье на хабре.

  (_А ЧТО НАСЧЕТ конвертации кода?_).


#### **Парадигма опционального типа (вместо заключения)**

null — это ошибка на миллион долларов. В свифте предпринята попытка избавиться от нее. Null должен быть отдельной сущностью, нельзя связывать значение объекта (данные, куда указывает указатель) с фактом отсутсвия данных. Сами данные хранят в себе информацию о том, существуют ли они — это очень тупо. Возможно, поэтому и человек не может точно сказать, существует ли он, поскольку не видит систему извне. Отсюда всякие тезисы типа "я мыслю, поэтому существую".

[Дополнительные замечания](https://medium.com/@thanyalukj/nullability-keywords-and-interoperability-between-objective-c-and-swift-220338af958b):
nonnull keywords are imported by Swift as a non-optional
nullable keywords are imported by Swift as an optional
Types declared without a nullability annotation are imported by Swift as implicitly unwrapped optional
null_resettable keyword is an interesting one.


### {{ site.further_readings }}
* [Optional (developer.apple.com)](https://developer.apple.com/documentation/swift/optional)
* [Optional Chaining (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html)
* [Nullability and Objective-C (developer.apple.com)](https://developer.apple.com/swift/blog/?id=25)
* [Option type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Option_type)
* [Nullable type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Nullable_type)
* [Атрибуты свойств в Objective-C (habrahabr.ru)](https://habrahabr.ru/post/265175/)

### {{ site.footnotes }}
