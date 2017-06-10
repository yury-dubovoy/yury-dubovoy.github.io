---
layout: post
title: Swift Optionals
date: '2017-06-04 10:00:00 +0300'
tags: [swift]
published: true
---

#### **Что такое Optionals?**
_Optionals_ (опционалы) — это удобный механизм обработки ситуаций, когда значение переменной может отсутствовать. Значение будет использовано, только если оно есть.

#### **Зачем нужны Optionals, когда есть проверка на nil?**
Во-первых, проверка на равенство/неравенство `nil` применима только к [nullable-типам](https://en.wikipedia.org/wiki/Nullable_type) и не применима к примитивным типам, структурам и перечислениям. Для обозначения отсутсвия значения у переменной примитивного типа приходится вводить спецзначения, такие как _NSNotFound_[^1]. Соответственно, пользователь этой переменной должен учитывать, что спецзначения возможны. В Swift даже примитивный тип можно использовать в опциональном стиле, т.е явным образом указывать на то, что значения может не быть.

Во-вторых: явная опциональность проверяется на этапе компиляции, что снижает количество ошибок в runtime. Опциональную переменную в Swift нельзя использовать точно так же, как неопциональную. Опционал нужно либо принудительно преобразовывать к обычному значению, либо использовать синтаксические конструкции с созданием новой переменной, такие как `if let`, `guard let` и `??`. Опционалы в Swift реализуют не просто проверку, но целую парадигму [опционального типа](https://en.wikipedia.org/wiki/Option_type) в [теории типов](https://en.wikipedia.org/wiki/Type_theory).

В-третьих, опционалы синтаксически более лаконичны, чем проверки на `nil`, что особенно хорошо видно на цепочках опциональных вызовов — так называемый _Optional Chaining_.

[^1]:[_NSNotFound_](https://developer.apple.com/reference/foundation/foundation_constants/nsnotfound) не только нужно рассматривать как спецзначение, но и следить, чтобы оно не входило в множество значений переменной. Ситуация усложняется еще и тем, что _NSNotFound_ считается равным _NSIntegerMax_, т.е. может иметь разные значения для разных (32-bit/64-bit) платформ. Это значит, что _NSNotFound_ нельзя напрямую записывать в файлы и архивы или использовать в [_Distributed Objects_](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/PreferencePanes/Tasks/Communication.html).


#### **Как это работает?**
Опционал в Swift представляет из себя особый объект-контейнер, который может содержать в себе либо `nil` либо объект конкретного типа, который указывается при объявлении этого контейнера. Эти два состояния обозначаются терминами _None_ и _Some_ соответственно. Если при создании опциональной переменной присваемое значение не указывать, то `nil` присваивается по умолчанию[^2]. Опционал объявляется посредством комбинации имени типа и лексемы `?`. Таким образом, запись `Int?` — это объявление контейнера, экземпляр которого может содержать внутри `nil`(состояние _None Int_) либо значение типа `Int` (состояние _Some Int_). Именно поэтому при преобразовании `Int?` в `Int` используетя термин _unwrapping_ вместо _cast_, т.е. подчеркивается "контейнерная" суть опционала. Лексема `nil` в Swift обозначает состояние _None_, которое можно присвоить любому опционалу. Это логично приводит к невозможности присвоить `nil` (состояние _None_) переменной, которая не является опционалом.

[^2]:В документации значение по умолчанию в случае отсутсвия явного присвоения не упоминается, но [сказано](https://developer.apple.com/documentation/swift/optional), что опционал _represents either a wrapped value or nil, the absence of a value_. Если опциональная переменная объявлена без явного присвоения (какое-либо _Some_ не присваивалось), то логично следует, что неявно присваивается _None_ — третьего "неинициализрованного" состояния у перечисления `Optional` нет.

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

        ...

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
var my_variable = Optional(42) // тип .some-значения Int опеределен неявно
{% endhighlight %}
{% highlight swift %}
var my_variable = Optional<Int>(42) // явное указание типа Int для наглядности
{% endhighlight %}
{% highlight swift %}
var my_variable = Int?(42) // тип Int необходимо указать явно
{% endhighlight %}
{% highlight swift %}
var my_variable: Int? = 42 // тип Int необходимо указать явно
{% endhighlight %}
{% highlight swift %}
var my_variable = Optional.some(42) // тип Int опеределен неявно
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

Перечисление `Optional<Wrapped>` также содержит свойство unsafelyUnwrapped, которое предоставляет доступ (только для чтения) к `.some`-значению опционала:

{% highlight swift %}
public enum Optional<Wrapped> : ExpressibleByNilLiteral {

        ...

    /// The wrapped value of this instance, unwrapped without checking whether
    /// the instance is `nil`.
    public var unsafelyUnwrapped: Wrapped { get }
}
{% endhighlight %}

Принудительное извлечение `.some`-значения обозначается лексемой `!` и называется _Force Unwrapping_ или _Explicit Unwrapping_. Применение _force unwrapping_ к опционалу в состоянии `.none` приведет к ошибке рантайма[^3] и аварийному завершению программы.

[^3]:Вывод в консоль: _fatal error: unexpectedly found nil while unwrapping an Optional value_

{% highlight swift %}

let my_variable1 = Int?(42) // содержит 42, тип Optional Int
let my_value1A = my_variable1! // содержит 42, тип Int
let my_value1B = my_variable1.unsafelyUnwrapped // содержит 42, тип Int

let my_variable2 = Int?.none // содержит nil, тип Optional Int
let my_value2 = my_variable2! // ошибка рантайма

{% endhighlight %}


#### **Идиомы использования**
Нет особого смысла использовать обычное перечисление с двумя состояниями. Вполне можно реализовать подобный механизм самостоятельно: создать _enum_ c двумя состояниями и конструкторами для соответствующих значений, добавить какой-нибудь постфиксный оператор для _force unwrapping_ (например, как это сделано [здесь](http://jamesonquave.com/blog/re-implementing-optionals-using-swifts-powerful-enum-type/)), добавить возможность сравнения с `nil` или вообще придумать "свой" `nil` и т.д. Опционалы должны быть интегрированы непосредственно в сам язык, чтобы их использование было естественным, не чужеродным. Разумеется, можно рассматривать такую интеграцию как "синтаксический сахар", однако языки высокого уровня для того и существуют, чтобы писать (и читать!) код на них было легко и приятно. Использование опционалов в Swift подразумевает ряд идиом или паттернов, которые помогают уменьшить количество ошибок и сделать код более лаконичным. К таким идиомам относятся _Implicit Unwrapping_, _Optional Chaining_, _Optional Binding_(конструкции `if let` и `guard let`), а также оператор _Nil-Coalescing_ `??`.

#### **Implicit unwrapping**

Безопасное использование _force unwrapping_ подразумевает предварительную проверку на `nil` в условии `if`:

{% highlight swift %}

let my_variable: Int? = tryGetResult() // тип Optional Int, значение неизвестно
if my_variable != nil {
  let my_value = my_variable! // содержит значение, вычисленное в tryGetResult()
} else {
  let my_value = my_variable! // ошибка рантайма, tryGetResult() вернула nil
}

{% endhighlight %}

Иногда из структуры программы очевидным образом следует, что переменная технически является опционалом, но к моменту первого использования всегда находится в состоянии .some, т.е. не является `nil`. Для использования опционала в неопциональном контексте (например, передать в функцию, у которой параметр обычного типа, а не опционал) приходится постоянно применять _force unwrapping_ с проверкой, что скучно и утомительно. В этих случаях можно применить неявно извлекаемый опционал _Implicitly Unwrapped Optional_. Неявно извлекамый опционал объявляется посредством комбинации имени типа и лексемы `!`:

{% highlight swift %}

let my_variable1: Int? = 42 // тип Optional Int
let my_variable2: Int! = 42 // тип Implicitly Unwrapped Optional Int
var my_variable3: Int! = 42 // тип Implicitly Unwrapped Optional Int

...

my_variable3 = nil // где-то непредвиденно присвоен nil

...

func sayHello(times:Int) {
    for _ in 0...times {
        print("Hello!")
    }
}

sayHello(times: my_variable1!) // обязаны извлекать значение явно
sayHello(times: my_variable1) // ошибка компиляции
sayHello(times: my_variable2!) // можно извлекать значение явно
sayHello(times: my_variable2) // неявное извлечение// обязаны извлекать явно
sayHello(times: my_variable3) // ошибка рантайма

{% endhighlight %}

В вызове `sayHello(times: my_variable2)` извлечения значения `42` из `my_variable2` **все равно осущетсвляется, только неявно**. Использование неявно извлекаемых опционалов делает код более удобным для чтения, но на практике это скорее анти-паттерн, увеличивающий вероятность ошибки. Неявно извлекаемый опционал заставляет компилятор "закрыть глаза" на то, что опционал используется в неопциональном контексте. Ошибка, которая может быть выявлена во время компиляции (вызов `sayHello(times: my_variable1)`), проявится только в рантайме (вызов `sayHello(times: my_variable3)`). Явный код всегда лучше неявного. Логично предположить, что такое явно снижение безопасности кода требуется не только ради "синтаксического сахара", и это действительно так.

Неявно извлекаемые опционалы позволяют использовать `self` в конструкторе для иницализации свойств и при этом:

* не нарушать правил [двухэтапной инициализации](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID220) (в конструкторе все свойства должны быть инициализированы до обращения к `self`);
* использовать свойство в неопциональном контексте (без необходимости добавлять `!` при обращении к свойству).

Наглядный пример, где требуется использовать `self` в конструкторе для иницализации свойств, приведен в [документации](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID55):

{% highlight swift %}

class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}

var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"

{% endhighlight %}

В этом примере экземпляры классов _Country_ и _City_ должны иметь ссылки друга на друга к моменту завершения инициализации. У каждой страны обязательно должна быть столица и у каждой столицы обязательно должна быть страна. Эти связи не являются опциональными. В процессе инициализации объекта `country` необходимо инициализировать свойство `capitalCity`. Для инициализации `capitalCity` нужно создать экземпляр класса _City_. Конструктор _City_ в качестве параметра требует экземпляр _Country_, но сложность в том, что экземпляр _Country_ на этот момент еще не до конца инициализирован (требуется доступ к `self`).

Эта задача имеет изящное решение: `capitalCity` объявляется неявно извлекаемым опционалом. Как и любой опционал, `capitalCity` по умолчанию присваивается `nil`, т. е. к моменту вызова конструктора _City_ все свойства `country` уже инициализированы. Требования двухэтапной инициализации соблюдены, конструктор _Country_ находится во второй фазе — можно передавать `self` в конструктор _City_. `capitalCity` является неявным опционалом, т.е. к нему можно обращаться в неопциональном контексте без добавления `!`.

**Таким образом, лексема `!` может быть использована в двух контекстах:**

* для принудительного извлечения значения из опционала;
* для объявления неявного опционала.

#### **Optional Chaining**

  **Таким образом, лексема `?` может быть использована в трех контекстах:**

  * для объявления явного опционала;
  * для опционального извлечения значения из опционала.


#### **Optional Binding**

конструкция iflet
конструкция guard



#### **Nil-Coalescing**
[Using the Nil-Coalescing Operator](https://developer.apple.com/documentation/swift/optional)

Важно не забывать, что опционал в качестве .some-значения тоже может содержать в себе опционал! В результате получим такую запись вида:
{% highlight swift %}
var value: AppDelegate?
var newValue = value.map { $0.window }
newValue??.makeKey()

var value: AppDelegate?
var newValue = value.flatMap { (value: AppDelegate) -> UIWindow? in
    //$0.window
    return value.window
}
newValue!.makeKey()


{% endhighlight %}

??
как же без них:
один ? на возврат нуля из функции map, другой ? на возврат из свойства window. А тройной можно?

  **Таким образом, лексема `??` может быть использована в двух контекстах:**

  * в качестве оператора Nil-Coalescing
  * для опционального извлечения значения из опционала;
  * для двойного .


#### **pap и flatMap**
условно можно отнести к идиомам, потому что это просто функции, определнные в системном перечислении _Optional_

(_МОЖЕТ, СТОИТ УБРАТЬ УПОМИНАНИЯ ЭТИХ МЕТОДОВ map<U>— ОНИ НЕ ИМЕЕЮТ ОТНОШЕНИЯ К ТЕМЕ?_).
Еще как имеют.


{% highlight swift %}
let arrayOfOptionals: [Int?] = [1, nil, 5, 2, nil]
let arrayOfNumbers = arrayOfOptionals.flatMap { $0 }
print(arrayOfNumbers) // [1, 5, 2]
{% endhighlight %}



конструкция map/flatmap
_I'm talking about the map and flatMap functions of Swift optionals (not the Array map function)._
http://blog.xebia.com/the-power-of-map-and-flatmap-of-swift-optionals/


На вход подаем трансформирующую функцию

{% highlight swift %}
public func flatMap<U>(_ transform: (Wrapped) throws -> U?) rethrows -> U?
{% endhighlight %}
которая принимает unwrapped элемент.

Разница между map и flatmap только в том, что во flatMap можно в кложурке отдавать nil, а в map нельзя

Не уверен, что этот механизм потом легко читать (пример с UICell), но, если привыкнуть к маппированию на проекте (маппирование принимает трансформирующую функцию), то вроде как и ок (сравнить с мапиирование коллекции).
https://www.natashatherobot.com/swift-using-map-to-deal-with-optionals/



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

#### **Optionals и ARC**
  https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID55
  Сделать переменную опциональной — не то же самое, что сделать weak-ссылку! Weak либо unowned нужно делать отдельно!

  А что будет, если переменная unowned превратится в ноль, хотя сама карточка еще существует (на нее дополнительную ссылку дали)

#### **Связка с Objective-C**
В Objective-C нет понятия опциональности, близко по смыслу находится возможность вернуть`nil` из метода, который обычно возвращает объект. В Swift и в Objective-C лексема `nil` имеет различную смысловую нагрузку. В Swift `nil` — это _Nothing_, т.е. признак отсутствия значения в переменной optional-типа, применяется к любым optional-типам. В Objective-C `nil` является указателем на несуществующий объект и применим только к типам объектов.


В Objective-C есть nullability annotations `_Nullable` и `_Nonnull` и nullability-аттрибуты свойств `nullable`, `nonnull`, `null_unspecified` и `null_resettable`. Они введены исключительно для совместимости с optional-семантикой в Swift, в Objective-C ни на что не влияют (если не считать предупреждений компиляции, например, при попытке присвоить nil nonnull-свойству). Кроме того, эти аттрибуты не применимы к примитивным типам, структурам и перечислениям. Подробнее про nullability annotations [здесь](https://developer.apple.com/swift/blog/?id=25) и очень понятно про nullability-аттрибуты свойств [здесь](https://habrahabr.ru/post/265175/).
В чем разница между `_Nullable` и `nullable`?
`null_unspecified` вообще нигде не упоминается, кроме как в статье на хабре.

#### **А ЧТО НАСЧЕТ конвертации кода?**
  [Дополнительные замечания](https://medium.com/@thanyalukj/nullability-keywords-and-interoperability-between-objective-c-and-swift-220338af958b):
  nonnull keywords are imported by Swift as a non-optional
  nullable keywords are imported by Swift as an optional
  Types declared without a nullability annotation are imported by Swift as implicitly unwrapped optional
  null_resettable keyword is an interesting one.

#### **Парадигма опционального типа (вместо заключения)**

null — это ошибка на миллион долларов. В свифте предпринята попытка избавиться от нее. Null должен быть отдельной сущностью, нельзя связывать значение объекта (данные, куда указывает указатель) с фактом отсутсвия данных. Сами данные хранят в себе информацию о том, существуют ли они — это очень тупо. Возможно, поэтому и человек не может точно сказать, существует ли он, поскольку не видит систему извне. Отсюда всякие тезисы типа "я мыслю, поэтому существую".



### {{ site.further_readings }}
* [Optional — Generic Enumeration (developer.apple.com)](https://developer.apple.com/documentation/swift/optional)
* [Swift Language Guide: Optionals (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID330)
* [Unowned References and Implicitly Unwrapped Optional Properties (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID55)
* [Nil-Coalescing Operator (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html#//apple_ref/doc/uid/TP40014097-CH6-ID72)
* [Optional Chaining (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html)
* [Nullability and Objective-C (developer.apple.com)](https://developer.apple.com/swift/blog/?id=25)
* [Option type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Option_type)
* [Nullable type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Nullable_type)
* [Re-implementing Optionals using Swift’s powerful enum type (jamesonquave.com)](http://jamesonquave.com/blog/re-implementing-optionals-using-swifts-powerful-enum-type/)
* [Атрибуты свойств в Objective-C (habrahabr.ru)](https://habrahabr.ru/post/265175/)

### {{ site.footnotes }}
