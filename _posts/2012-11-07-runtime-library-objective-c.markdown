---
layout: post
title: Runtime-библиотека Objective-C
date: '2012-11-07 10:00:00 +0300'
tags: [objective-c]
published: true
---

Runtime-библиотека Objective-C (_Objective-С runtime system_) — это динамическая разделяемая библиотека, предосталяющая интерфейс к набору функций и структур данных в каталоге `/usr/include/objc`:

{% highlight objective-c %}
#import <objc/runtime.h>
#import <objc/message.h>
#import <malloc/malloc.h>
{% endhighlight %}

![Path to Objective-C runtime library]({{ site.url }}/assets/images/objc_lib_path.png)

Практически все возможности Objective-C, которые в удобном виде предоставляет среда разработки, можно эмулировать с помощью C-функций и структур данных, определенных в библиотеке. Можно считать, что Objective-C — это старый добрый C + runtime-библиотека.

Библиотека отвечает за:

* [формирование классов и объектов](ссылка на статью про классы), механизмы протоколов и наследования;
* обработку свойств и сообщений;
* динамическое определение методов ([Message Forwarding](статья про Message Forwarding));
* предоставление доступа к различной мета-информации (имя класса, список методов класса и т.п.).

Программы на Objective-C взаимодействуют с библиотекой на 3-х различных уровнях:

* непосредственно код на Objective-C (синтаксис создания объектов, [отправки сообщений](ссылка на статью про классы) и т.п.);
* сообщения/методы, определенные в классе _NSObject_ (например, _methodForSelector:_);
* через прямые вызовы функций, определенных в библиотеке.

В документации явно [указывается](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tag/objc_cache), что в большинстве случаев разработки на Objective-C непосредственная работа с библиотекой не требуется. Это полезно, когда нужно задействовать в одном проекте другие языки программирования (например, С++). Вместе с тем понимание принципов работы библиотеки значительно облегчает правильное использование прочих языковых концепций, таких как селекторы, [_категории_](Категории в Objective-C в деталях), [_Message Forwarding_](статья про Message Forwarding) и т.д.  


### {{ site.further_readings }}


* [Objective-C Runtime Programming Guide (developer.apple.com)](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtInteracting.html)
* [Objective-C Runtime Reference (developer.apple.com)](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tag/objc_cache)
