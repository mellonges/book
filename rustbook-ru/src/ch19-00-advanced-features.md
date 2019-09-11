# Расширенные возможности

Мы прошли долгий путь! К настоящему времени мы узнали 99% всего, что может вам
понадобятся при написании программ на Rust. Прежде чем мы сделаем еще один проект
в главе 20, давайте поговорим о нескольких вещах, с которыми вы можете столкнуться
в течение последнего 1% времени. Не стесняйся пропустите эту главу и вернитесь к ней,
как только вы столкнетесь с этими вещами в своей работе; функции, которые мы научимся
использовать здесь, полезны в очень специфических ситуациях. Мы не хотим оставлять
эти функции без внимания, хотя вы к ним не будет часто обращаться.

В этой главе мы рассмотрим:

* Небезопасный Rust: когда вам нужно отказаться от некоторых гарантий Rust и
   сообщим компилятору, что вы будете нести ответственность за соблюдение гарантий
* Расширенные сроки жизни: дополнительный синтаксис в сложных ситуациях
* Расширенные типажи: связанные типы, параметры типа по умолчанию, супертипажи и
  шаблон newtype для связи с типажами
* Расширенные типы: еще несколько о шаблоне newtype, псевдонимы типов,
   Тип «никогда» (“never”) и типы с динамическим размером
* Расширенные функции и замыкания: указатели на функции и возвращение замыканий
  из функции.

Итак приступим.