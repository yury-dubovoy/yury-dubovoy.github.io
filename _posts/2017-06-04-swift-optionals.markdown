---
layout: post
title: Swift Optionals
date: '2017-06-04 10:00:00 +0300'
tags: [swift]
published: true
---

<!-- Всем привет! Я уже несколько лет профессионально (т.е. работа такая) занимаюсь iOS-разработкой, по большей части на Swift. Регулярно на почве свифтовых опционалов возникали ситуации, когда я знал _что_ нужно делать, но не совсем внятно представлял, _почему_ именно так. Приходилось отвлекаться и углубляться в документацию — количество "заметок на полях" пополнялось с удручающей периодичностью. В определенный момент они достигли критической массы, и я решил упорядочить их в едином исчерпывающем руководстве. Материал получился довольно объемным, поскольку предпринята попытка раскрыть тему максимально подробно. Статья будет полезна как начинающим Swift-разрабочикам, так и матерым профессионалам — есть ненулевая вероятность, что и последние найдут для себя что-то новое. А если не найдут, то добавят свое новое в комментарии, и всем будет польза. Прошу под кат!  -->

#### **Что такое Optionals?**
_Optionals_ (опционалы) — это удобный механизм обработки ситуаций, когда значение переменной может отсутствовать. Значение будет использовано, только если оно есть.

#### **Зачем нужны Optionals, когда есть проверка на nil?**
Во-первых, проверка на равенство/неравенство `nil` применима только к [nullable-типам](https://en.wikipedia.org/wiki/Nullable_type) и не применима к примитивным типам, структурам и перечислениям. Для обозначения отсутсвия значения у переменной примитивного типа приходится вводить спецзначения, такие как _NSNotFound_[^1]. Соответственно, пользователь этой переменной должен учитывать, что спецзначения возможны. В Swift даже примитивный тип можно использовать в опциональном стиле, т.е явным образом указывать на то, что значения может не быть.

Во-вторых: явная опциональность проверяется на этапе компиляции, что снижает количество ошибок в runtime. Опциональную переменную в Swift нельзя использовать точно так же, как неопциональную (за исключением неявно извлекамых опционалов, подробности в разделе _Implicit Unwrapping_). Опционал нужно либо принудительно преобразовывать к обычному значению, либо использовать конструкции условия, такие как `if let`, `guard let` и `??`. Опционалы в Swift реализуют не просто проверку, но целую парадигму [опционального типа](https://en.wikipedia.org/wiki/Option_type) в [теории типов](https://en.wikipedia.org/wiki/Type_theory).

В-третьих, опционалы синтаксически более лаконичны, чем проверки на `nil`, что особенно хорошо видно на цепочках опциональных вызовов — так называемый _Optional Chaining_.

[^1]:[_NSNotFound_](https://developer.apple.com/reference/foundation/foundation_constants/nsnotfound) не только нужно рассматривать как спецзначение, но и следить, чтобы оно не входило в множество значений переменной. Ситуация усложняется еще и тем, что _NSNotFound_ считается равным _NSIntegerMax_, т.е. может иметь разные значения для разных (32-bit/64-bit) платформ. Это значит, что _NSNotFound_ нельзя напрямую записывать в файлы и архивы или использовать в [_Distributed Objects_](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/PreferencePanes/Tasks/Communication.html).


#### **Как это работает?**
Опционал в Swift представляет из себя особый объект-контейнер, который может содержать в себе либо `nil`, либо объект конкретного типа, который указывается при объявлении этого контейнера. Эти два состояния обозначаются терминами _None_ и _Some_ соответственно. Если при создании опциональной переменной присваемое значение не указывать, то `nil` присваивается по умолчанию[^2]. Опционал объявляется посредством комбинации имени типа и лексемы `?`. Таким образом, запись `Int?` — это объявление контейнера, экземпляр которого может содержать внутри `nil`(состояние _None Int_) либо значение типа `Int` (состояние _Some Int_). Именно поэтому при преобразовании `Int?` в `Int` используетя термин _unwrapping_ вместо _cast_, т.е. подчеркивается "контейнерная" суть опционала. Лексема `nil` в Swift обозначает состояние _None_, которое можно присвоить любому опционалу. Это логично приводит к невозможности присвоить `nil` (состояние _None_) переменной, которая не является опционалом.

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
поскольку `nil` будет присвоен по умолчанию.

Перечисление `Optional<Wrapped>` также содержит свойство unsafelyUnwrapped, которое предоставляет доступ (только для чтения) к `.some`-значению опционала:

{% highlight swift %}
public enum Optional<Wrapped> : ExpressibleByNilLiteral {

        ...

    /// The wrapped value of this instance, unwrapped without checking whether
    /// the instance is `nil`.
    public var unsafelyUnwrapped: Wrapped { get }
}
{% endhighlight %}

Если опционал находится в состоянии `.none`, обращение к `unsafelyUnwrapped` приведет к сбою программы. В режиме отладки _debug build -Onone_ это приведет к ошибке рантайма[^3], в релизной сборке _optimized build -O_ — к ошибке рантайма **[либо неопределенному поведению](https://developer.apple.com/documentation/swift/optional/1641793-unsafelyunwrapped)**. Более безопасной операцией является _Force Unwrapping_ (или _Explicit Unwrapping_) — принудительное извлечение `.some`-значения, обозначаемое лексемой `!`. Применение _Force Unwrapping_ к опционалу в состоянии `.none` приведет к ошибке рантайма[^4].

[^3]:Вывод в консоль: _fatal error: unsafelyUnwrapped of nil optional_
[^4]:Вывод в консоль: _fatal error: unexpectedly found nil while unwrapping an Optional value_

{% highlight swift %}

let my_variable1 = Int?(42) // содержит 42, тип Optional Int
let my_value1A = my_variable1! // содержит 42, тип Int
let my_value1B = my_variable1.unsafelyUnwrapped // содержит 42, тип Int

let my_variable2 = Int?.none // содержит nil, тип Optional Int
let my_value2A = my_variable2! // ошибка рантайма
// ошибка рантайма в режиме -Onone, НЕОПРЕДЕЛЕННОЕ ПОВЕДЕНИЕ в режиме -O
let my_value2B = my_variable2.unsafelyUnwrapped

{% endhighlight %}


#### **Идиомы использования**
Нет особого смысла использовать обычное перечисление с двумя состояниями. Вполне можно реализовать подобный механизм самостоятельно: создать _enum_ c двумя состояниями и конструкторами для соответствующих значений, добавить какой-нибудь постфиксный оператор для _Force Unwrapping_ (например, как это сделано [здесь](http://jamesonquave.com/blog/re-implementing-optionals-using-swifts-powerful-enum-type/)), добавить возможность сравнения с `nil` или вообще придумать "свой" `nil` и т.д. Опционалы должны быть интегрированы непосредственно в сам язык, чтобы их использование было естественным, не чужеродным. Разумеется, можно рассматривать такую интеграцию как "синтаксический сахар", однако языки высокого уровня для того и существуют, чтобы писать (и читать) код на них было легко и приятно. Использование опционалов в Swift подразумевает ряд идиом или паттернов, которые помогают уменьшить количество ошибок и сделать код более лаконичным. К таким идиомам относятся _Implicit Unwrapping_, _Optional Chaining_, _Nil-Coalescing_ и _Optional Binding_.

#### **Implicit unwrapping**

Безопасное использование _Force Unwrapping_ подразумевает предварительную проверку на `nil`, например, в условии `if`:

{% highlight swift %}

// tryGetResult() может вернуть nil
let my_variable: Int? = tryGetResult() // тип Optional Int
if my_variable != nil {
  let my_value = my_variable! // содержит значение, вычисленное в tryGetResult()
} else {
  let my_value = my_variable! // ошибка рантайма, tryGetResult() вернула nil
}

{% endhighlight %}

Иногда из структуры программы очевидным образом следует, что переменная технически является опционалом, но к моменту первого использования всегда находится в состоянии `.some`, т.е. не является `nil`. Для использования опционала в неопциональном контексте (например, передать его в функцию с параметром неопционального типа) приходится постоянно применять _Force Unwrapping_ с предварительной проверкой, что скучно и утомительно. В этих случаях можно применить неявно извлекаемый опционал — _Implicitly Unwrapped Optional_. Неявно извлекамый опционал объявляется посредством комбинации имени типа и лексемы `!`:

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
sayHello(times: my_variable2!) // можем, но не обязаны извлекать значение явно
sayHello(times: my_variable2) // неявное извлечение
sayHello(times: my_variable3) // ошибка рантайма

{% endhighlight %}

В вызове `sayHello(times: my_variable2)` извлечение значения `42` из `my_variable2` **все равно осуществляется, только неявно**. Использование неявно извлекаемых опционалов делает код более удобным для чтения — нет восклицательных знаков, которые отвлекают внимание (вероятно, читающего код будет беспокоить использование _Force Unwrapping_ без предварительной проверки). На практике это скорее анти-паттерн, увеличивающий вероятность ошибки. Неявно извлекаемый опционал заставляет компилятор "закрыть глаза" на то, что опционал используется в неопциональном контексте. Ошибка, которая может быть выявлена во время компиляции (вызов `sayHello(times: my_variable1)`), проявится только в рантайме (вызов `sayHello(times: my_variable3)`). Явный код всегда лучше неявного. Логично предположить, что такое снижение безопасности кода требуется не только ради "синтаксического сахара", и это действительно так.

Неявно извлекаемые опционалы позволяют использовать `self` в конструкторе для иницализации свойств и при этом:

* не нарушать правил [двухэтапной инициализации](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID220) (в конструкторе все свойства должны быть инициализированы до обращения к `self`) — иначе код просто не скомпилируется;
* избежать излишней опциональности в свойстве, для которого она не требуется (по своему смыслу значение свойства не может отсутствовать).

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

В этом примере экземпляры классов _Country_ и _City_ должны иметь ссылки друга на друга к моменту завершения инициализации. У каждой страны обязательно должна быть столица и у каждой столицы обязательно должна быть страна. Эти связи не являются опциональными — они безусловны. В процессе инициализации объекта `country` необходимо инициализировать свойство `capitalCity`. Для инициализации `capitalCity` нужно создать экземпляр класса _City_. Конструктор _City_ в качестве параметра требует соответствующий экземпляр _Country_, но сложность в том, что экземпляр _Country_ на этот момент еще не до конца инициализирован (требуется доступ к `self`).

Эта задача имеет изящное решение: `capitalCity` объявляется мутабельным неявно извлекаемым опционалом. Как и любой мутабельный опционал, `capitalCity` по умолчанию инициализируется состоянием `nil`, т. е. к моменту вызова конструктора _City_ все свойства объекта `country` уже инициализированы. Требования двухэтапной инициализации соблюдены, конструктор _Country_ находится во второй фазе — можно передавать `self` в конструктор _City_. `capitalCity` является неявным опционалом, т.е. к нему можно обращаться в неопциональном контексте без добавления `!`.

Побочным эффектом использования неявно извлекаемого опционала является "встроенный" `assert`: если `capitalCity` по каким-либо причинам останется в состоянии `nil`, это приведет к ошибке рантайма и аварийному завершению работы программы.

Другим примером оправданного использования неявно извлекаемых опционалов могут послужить инструкции `@IBOutlet`: контекст их использования подразумевает, что переменной к моменту первого обращения автоматически будет присвоено `.some`-значение.  Если это не так, то произойдет ошибка рантайма. Автоматическая генерация кода в _Interface Builder_ создает свойства с `@IBOutlet` именно в виде неявных опционалов. Если такое поведение неприемлемо, свойство с `@IBOutlet` можно объявить в виде явного опционала и всегда обрабатывать `.none`-значения явным образом. Как правило, все же лучше сразу получить "падение", чем заниматься долгой отладкой в случае случайно отвязанного `@IBOutlet`-свойства.

#### **Optional Chaining**

_Optional Chaining_ — это процесс последовательных вызовов по цепочке, где каждое из звеньев возвращает опционал. Процесс прерывается на первом опционале, находящемся в состоянии `nil` — в этом случае результатом всей цепочки вызовов также будет `nil`. Если все звенья цепочки находятся в состоянии `.some`, то результирующим значением будет опционал с результатом последнего вызова. Для формирования звеньев цепочки используется лексема `?`, которая помещается сразу за вызовом, возвращающим опционал. Звеньями цепочки могут любые операции, которые возвращают опционал: обращение к локальной переменной (в качестве первого звена), вызовы свойств и методов, доступ по индексу.

_Optional сhaining_ всегда работает последовательно слева направо. Каждому следующему звену передается `.some`-значение предыдущего звена, при этом результирующее значение цепочки всегда является опционалом, т.е. цепочка работает по следующим правилам:

* первое звено должно быть опционалом;
* после лексемы `?` должно быть следующее звено;
* если звено в состояни `.none`, то цепочка прерывает процесс вызовов и возвращает `nil`;
* если звено в состояни `.some`, то цепочка отдает `.some`-значение звена на вход следующему звену (если оно есть);
* если результат последнего звена является опционалом, то цепочка возвращает этот опционал;
* если результат последнего звена не является опционалом, то цепочка возвращает этот результат, "завернутый" в опционал (результат присваивается `.some`-значению возвращаемого опционала).

{% highlight swift %}

// цепочка из трех звеньев: первое звено — опционал 'country.mainSeaport?',
country.mainSeaport?.nearestVacantPier?.capacity

// ошибка компиляции, после '?' должно быть следующее звено
let mainSeaport = country.mainSeaport?

// цепочка вернет 'nil' на первом звене
country = Country(name: "Mongolia")
let capacity = country.mainSeaport?.mainPier?.capacity

// цепочка вернет опционал — ближайший незанятый пирс в Хельсинки
country = Country(name: "Finland")
let nearestVacantPier = country.mainSeaport?.nearestVacantPier

// цепочка вернет опционал — количество свободных мест, даже если capacity
// является неопциональным значением
country = Country(name: "Finland")
let capacity = country.mainSeaport?.nearestVacantPier?.capacity

{% endhighlight %}

Важно отличать цепочки опциональных вызовов от вложенных опционалов. Вложенный опционал образуется, когда `.some`-значением одного опционала является другой опционал:

{% highlight swift %}

let valueA = 42
let optionalValueA = Optional(valueA)
let doubleOptionalValueA = Optional(optionalValueA)
let tripleOptionalValueA = Optional(doubleOptionalValueA)

let tripleOptionalValueB: Int??? = 42 // три '?' означают тройную вложенность
let doubleOptionalValueB = tripleOptionalValueB!
let optionalValueB = doubleOptionalValueB!
let valueB = optionalValueB!

print("\(valueA)") // 42
print("\(optionalValueA)") // Optional(42)
print("\(doubleOptionalValueA)") // Optional(Optional(42))
print("\(tripleOptionalValueA)") // Optional(Optional(Optional(42)))

print("\(tripleOptionalValueB)") // Optional(Optional(Optional(42)))
print("\(doubleOptionalValueB)") // Optional(Optional(42))
print("\(optionalValueB)") // Optional(42)
print("\(valueB)") // 42

{% endhighlight %}

_Optional сhaining_ не увеличивает уровень вложенности возвращаемого опционала. Тем не менее, это не исключает ситуации, когда результирующим значением какого-либо звена является опционал с несколькими уровнями вложенности. В таких ситуациях для продолжения цепочки необходимо прописать `?` в количестве, равном количеству уровней вложенности[^5]:

[^5]:Вообще говоря, необязательно, чтобы все уровни вложенности были "развернуты" с помощью лексемы `?`.Часть из них можно заменить на принудительное извлечение `!`, что сократит количество "неявных" звеньев в цепочке. Отдельный вопрос, есть ли в этом смысл.

{% highlight swift %}

let optionalAppDelegate = UIApplication.shared.delegate
let doubleOptionalWindow = UIApplication.shared.delegate?.window
let optionalFrame = UIApplication.shared.delegate?.window??.frame // два '?'
print("\(optionalAppDelegate)") // Optional( ... )
print("\(doubleOptionalWindow)") // Optional(Optional( ... ))
print("\(optionalFrame)") // Optional( ... )

{% endhighlight %}

Цепочка `UIApplication.shared.delegate?.window??.frame` фактически состоит из четырех звеньев: `UIApplication.shared.delegate?`, `.frame` и два звена, объединенных в одном вызове `.window??`. Второе "двойное" звено представлено опционалом второго уровня вложенности.

Важной особенностью этого примера также является особый способ формирования двойного опционала, отличающийся от способа формирования `doubleOptionalValue` в предыдущем примере. `UIApplication.shared.delegate!.window` является **опциональным свойством**, в котором возвращается опционал. Опциональность свойства означает, что может отсутствовать само свойство, а не только `.some`-значение у опционала, возвращаемого из свойства. Опциональное свойство, также как и все прочие свойства, может возвращать любой тип, не только опциональный. Опциональность такого рода формируется в [@objc-протоколах](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID284) с помощью модификатора `optional`:

{% highlight swift %}
public protocol UIApplicationDelegate : NSObjectProtocol {

...

@available(iOS 5.0, * )
optional public var window: UIWindow? { get set } // модификатор optional

...

}
{% endhighlight %}

В протоколах с опциональными свойствами и методами (требованиями) модификатор `@objc` указывается для каждого опционального требования и для самого протокола. Вызов нереализованного требования у объекта, реализующего такой протокол, возвращает опционал соответствующего типа в состоянии `.none`. Вызов реализованного требования возвращает опционал соответсвующего типа в состоянии `.some`. Таким образом, опциональные свойства и методы, в отличие от _optional сhaining_, увеличивают уровень вложенности возвращаемого опционала. Опциональный метод, как и свойство, "заворачивается" в опционал полностью — в `.some`-значение помещается метод целиком, а не только возращаемое значение:

{% highlight swift %}

@objc
public protocol myOptionalProtocol {

    @objc
    optional var my_variable: Int { get }

    @objc
    optional var my_optionalVariableA: Int? { get } // ошибка компиляции

    @objc
    optional var my_optionalVariableB: UIView? { get }

    @objc
    optional func my_func() -> Int

    @objc
    optional func my_optionalResultfuncA() -> Int? // ошибка компиляции

    @objc
    optional func my_optionalResultfuncB() -> UIView?

    @objc
    optional init(value: Int) // ошибка компиляции

}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, myOptionalProtocol {

    var window: UIWindow?

    func application(_ application: UIApplication,
      didFinishLaunchingWithOptions launchOptions:
      [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

        let protocolAdoption = self as myOptionalProtocol

        // Optional<Int>
        print("\(type(of: protocolAdoption.my_variable))")

        // Optional<Optional<UIView>>
        print("\(type(of: protocolAdoption.my_optionalVariableB))")

        // Optional<() -> Int>
        print("\(type(of: protocolAdoption.my_func))")

        // Optional<Int>
        print("\(type(of: protocolAdoption.my_func?()))")

        // Optional<() -> Optional<UIView>>
        print("\(type(of: protocolAdoption.my_optionalResultfuncB))")

        // Optional<UIView>
        print("\(type(of: protocolAdoption.my_optionalResultfuncB?()))")

        return true
    }
}

{% endhighlight %}

Для опциональных `@objc`-протоколов имеется ряд ограничений, в виду того, что они были введены в Swift специально для взаимодействия с кодом на Objective-C:
* могут быть реализованы только в классах, унаследованных от классов Objective-C, либо других классов с аттрибутом `@objc` (т.е. не могут быть реализованы в структурах и перечислениях);
* модификатор `optional` неприменим к требованиям-конструкторам `init`;
* cвойства и методы с аттрибутом `@objc` имеют ограничение на тип возвращаемого опционала — допускаются только классы.

Попытка применить _Force Unwrapping_ на нереализованном свойстве или методе приведет к точно такой же ошибке рантайма, как и применение _Force Unwrapping_ на любом другом опционале в состоянии `.none`.

#### **Nil-Coalescing**

Оператор _Nil-Coalescing_ возвращает `.some`-значение опционала, если опционал в состоянии `.some`, и значение по умолчанию, если опционал в состоянии `.none`. Обычно _Nil-Coalescing_ более лаконичен, чем условие `if else`, и легче воспринимается, чем тернарный условный оператор `?`:

{% highlight swift %}

let optionalText: String? = tryExtractText()

// многословно
let textA: String
if optionalText != nil {
    textA = optionalText!
} else {
    textA = "Extraction Error!"
}

// в одну строку, но заставляет думать
let textB = (optionalText != nil) ? optionalText! : "Extraction Error!"

// кратко и легко воспринимать
let textC = optionalText ?? "Extraction Error!"

{% endhighlight %}

Тип значения по умолчания справа должен соответствовать типу `.some`-значения опционала слева. Значение по умолчанию тоже может быть опционалом:

{% highlight swift %}

let optionalText: String?? = tryExtractOptionalText()
let a = optionalText ?? Optional("Extraction Error!")

{% endhighlight %}

Возможность использовать выражения в качестве правого операнда позволяет создавать цепочки из умолчаний:

{% highlight swift %}

let wayA: Int? = doSomething()
let wayB: Int? = doNothing()
let defaultWay: Int = ignoreEverything()

let whatDo = wayA ?? wayB ?? defaultWay

{% endhighlight %}

#### **Optional Binding и приведение типов**

_Optional Binding_ позволяет проверить, содержит ли опционал `.some`-значение, и если содержит, извлечь его и предоставить к нему доступ через с помощью локальной переменной (обычно константной). _Optional Binding_ работает в контексте конструкций `if`, `while` и `guard`.

В документации детали реализации _Optional Binding_ не описаны, но можно построить модель, хорошо описывающую поведение этого механизма.

В Swift каждый метод или функция без явно заданного `return` неявно возвращает [пустой кортеж](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html#//apple_ref/doc/uid/TP40014097-CH10-ID163) `()`. Оператор присваивания `=` является исключением и [не возвращает значение](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html#//apple_ref/doc/uid/TP40014097-CH6-ID62), тем самым позволяя избежать случайного присвоения вместо сравнения `==`.

 Допустим, что `=` неявно также обычно возвращает пустой кортеж, но если правый операнд является опционалом в состоянии `nil`, то оператор присваивания вернет `nil`. Тогда эту особенность можно будет использовать в условии `if`, поскольку пустой кортеж расценивается как _true_, а `nil` расценивается как _false_:

{% highlight swift %}

// не забыть пример кода!

{% endhighlight %}

Переменную, сформированную в контексте ветки _true_, можно будет автоматически объявить неявно извлекаемым опционалом или даже обычной переменной. Результат присвоения _true_ будет являться следствием `.some`-состояния правого операнда. В ветке _true_ `.some`-значение всегда будет успешно извлечено и "привязано" к новой переменной (поэтому механизм в целом и называется _Optional Binding_).

Область видимости извлеченной переменной в условии `if` ограничивается веткой _true_, что логично, поскольку в ветке _false_, если потребуется, всегда можно объявить новый опционал в состоянии `none`. Тем не менее, существуют ситуации, когда нужно расширить область видимости извлеченного `.some`-значения, а в ветке _false_ (опционал в состоянии `.none`) завершить работу функции. В таких ситуациях удобно воспользоваться условием `guard`:

{% highlight swift %}

let my_optionalVariable: Int? = extractOptionalValue()

// чересчур многословно
let my_variableA: Int
if let value = my_optionalVariable {
  my_variableA = value
} else {
  return
}
print(my_variableA + 1)

// лаконично
guard let my_variableB = my_optionalVariable else {
  return
}
print(my_variableB + 1)

{% endhighlight %}

В Swift гарантированное компилятором приведение типов (например, повышающее приведение или указание типа литерала) выполняется с помощью оператора `as`. В случаях, когда компилятор не может гарантировать успешное приведение типа (например, понижающее приведение), используется либо оператор принудительного приведения `as!`, либо оператор опционального приведения `as?`. Принудительное приведение работает в стиле _Force Unwrapping_, т.е. в случае невозможности выполнить приведение приведет к ошибке рантайма, в то время как опционального приведение в этом случае вернет `nil`:

{% highlight swift %}

class Shape {}
class Circle: Shape {}
class Triangle: Shape {}

let circle = Circle()
let circleShape: Shape = Circle()
let triangleShape: Shape = Triangle()

circle as Shape // гарантированное приведение
42 as Float // гарантированное приведение
circleShape as Circle // ошибка компиляции

circleShape as! Circle // circle
triangleShape as! Circle // ошибка рантайма

circleShape as? Circle // Optional<Circle>
triangleShape as? Circle // nil

{% endhighlight %}

Таким образом, оператор опционального приведения `as?` порождает опционал, который часто используется в связке с _Optional Binding_:

{% highlight swift %}

// не забыть пример кода!

{% endhighlight %}


#### **Фунции map и flatMap**
условно можно отнести к идиомам, потому что это просто функции, определнные в системном перечислении _Optional_



Использование условий в качестве проверки, был ли выполнен метод или было ли успешно присвание: каждый метод/функция без return неявно возращает пустой кортеж, что может быть использовано для проверки на `nil`. Дело вкуса, но не факт, что подобная запись выглядит более естественно, чем явная предварительная проверка. Зависит от договоренностей по стилю кодирования, принятых на проекте.
https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html#//apple_ref/doc/uid/TP40014097-CH21-ID249



(_МОЖЕТ, СТОИТ УБРАТЬ УПОМИНАНИЯ ЭТИХ МЕТОДОВ map<U>— ОНИ НЕ ИМЕЕЮТ ОТНОШЕНИЯ К ТЕМЕ?_).
Еще как имеют.


{% highlight swift %}
let arrayOfOptionals: [Int?] = [1, nil, 5, 2, nil]
let arrayOfNumbers = arrayOfOptionals.flatMap { $0 }
print(arrayOfNumbers) // [1, 5, 2]
{% endhighlight %}

// плохой пример, неудобно читать
let attachHeight: CGFloat = attach.map { $0.isForward() ? 100 : 0 } ?? 0

let attachHeight: CGFloat
if attach != nil {
    attachHeight = attach!.isForward() ? 100 : 0
} else {
    attachHeight = 0
}

// хороший пример, удобно читать
replaySplitter.frame = textCellLayout.replaySptillerFrame.map { $0 } ?? CGRect.zero

if textCellLayout.replaySptillerFrame != nil {
    replaySplitter.frame = textCellLayout.replaySptillerFrame!
} else {
    replaySplitter.frame = CGRect.zero
}



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



{% highlight swift %}

public func ??<T>(optional: T?, defaultValue: @autoclosure () throws -> T) rethrows -> T

public func ??<T>(optional: T?, defaultValue: @autoclosure () throws -> T?) rethrows -> T?

{% endhighlight %}


#### **Связка с Objective-C**
В Objective-C нет понятия опциональности, близко по смыслу находится возможность вернуть `nil` из метода, который обычно возвращает объект. В Swift и в Objective-C лексема `nil` имеет различную смысловую нагрузку. В Swift `nil` — это _Nothing_, т.е. признак отсутствия значения в переменной optional-типа, применяется к любым optional-типам. В Objective-C `nil` является указателем на несуществующий объект и применим только к типам объектов.


В Objective-C есть nullability annotations `_Nullable` и `_Nonnull` и nullability-аттрибуты свойств `nullable`, `nonnull`, `null_unspecified` и `null_resettable`. Они введены исключительно для совместимости с optional-семантикой в Swift, в Objective-C ни на что не влияют (если не считать предупреждений компиляции, например, при попытке присвоить nil nonnull-свойству). Кроме того, эти аттрибуты не применимы к примитивным типам, структурам и перечислениям. Подробнее про nullability annotations [здесь](https://developer.apple.com/swift/blog/?id=25) и очень понятно про nullability-аттрибуты свойств [здесь](https://habrahabr.ru/post/265175/).
В чем разница между `_Nullable` и `nullable`?
`null_unspecified` вообще нигде не упоминается, кроме как в статье на хабре.

**А ЧТО НАСЧЕТ конвертации кода?**
  [Дополнительные замечания](https://medium.com/@thanyalukj/nullability-keywords-and-interoperability-between-objective-c-and-swift-220338af958b):
  nonnull keywords are imported by Swift as a non-optional
  nullable keywords are imported by Swift as an optional
  Types declared without a nullability annotation are imported by Swift as implicitly unwrapped optional
  null_resettable keyword is an interesting one.

#### **Резюме, синтаксис**

Лексема `!` может быть использована в трех контекстах, связанных с опциональностью[^6] :

* для принудительного извлечения значения из опционала;
* для объявления неявного опционала;
* для принудительной конвертации типов в операторе `as!`.

[^6]:Унарный логического отрицания `!` не считается, поскольку относится к другому контексту.

Лексема `?` может быть использована в двух контекстах, связанных с опциональностью[^7]:

* для объявления явного опционала;
* для использования опционала в _optional chaining_.
* для опциональной конвертации типов в операторе `as?`.

[^7]:Тернарный условный оператор `?` не считается, поскольку относится к другому контексту.

Лексема `??` может быть использована в двух контекстах:

* в _Optional сhaining_ для обращения к опционалу второго уровня вложенности;
* в качестве оператора _Nil-Coalescing_.

#### **Заключение**

Нулевой указатель — это [ошибка на миллиард долларов](https://en.wikipedia.org/wiki/Tony_Hoare). Практика объединения значения указателя (адреса) с фактом отсутствия данных, например, нулевой указатель _null_ в С++ или Java, не решает проблему специфичных констант. Вызывающая сторона все равно должна учитывать контекст и проверять результат на равенство специфичной константе, означающее отсутствие данных. Тот факт, что константа _null_ всего одна, принципиально ситуацию не меняет и лишь добавляет неожиданностей при приведении типов.

Факт отсутствия данных должен обрабатываться отдельной сущностью, внешней по отношению к самим данным. "Правильный" указатель не может существовать без адресата, следовательно, не может "осознать" отсутствие адресата. Даже человеку, т.е. довольно сложной системе, приходится довольстоваться аксиомой _Cogito, ergo sum_ (лат.  — "Мыслю, следовательно существую"). У человека нет достоверных признаков собственного бытия или небытия, но у внешних по отношению к человеку сущностей эти критерии есть. В Swift такой внешней сущностью является опционал.

### {{ site.further_readings }}
* [Generic Enumeration: Optional (developer.apple.com)](https://developer.apple.com/documentation/swift/optional)
* [Instance Property: unsafelyUnwrapped (developer.apple.com)](https://developer.apple.com/documentation/swift/optional/1641793-unsafelyunwrapped)
* [Swift Language Guide: Optionals (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID330)
* [Unowned References and Implicitly Unwrapped Optional Properties (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID55)
* [Nil-Coalescing Operator (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html#//apple_ref/doc/uid/TP40014097-CH6-ID72)
* [Optional Chaining (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html)
* [Optional Protocol Requirements (developer.apple.com)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID284)
* [The as! Operator (developer.apple.com)](https://developer.apple.com/swift/blog/?id=23)
* [Nullability and Objective-C (developer.apple.com)](https://developer.apple.com/swift/blog/?id=25)
* [Option type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Option_type)
* [Nullable type (en.wikipedia.org)](https://en.wikipedia.org/wiki/Nullable_type)
* [Re-implementing Optionals using Swift’s powerful enum type (jamesonquave.com)](http://jamesonquave.com/blog/re-implementing-optionals-using-swifts-powerful-enum-type/)
* [Атрибуты свойств в Objective-C (habrahabr.ru)](https://habrahabr.ru/post/265175/)

### {{ site.footnotes }}
