## Оператор управления работой кода `match`

`match` весьма мощная конструкция, которая позволяет сравнивать значение с серией
шаблонов и затем выполнять код, в зависимости от того, какое значение было выбрано.
Шаблоном может выступать как литеральное значение, так и имена переменных, подстановочные
значения и многое другое. В главе 18 будет много рассказано о различных типах шаблонов
и что каждый из них делает. Мощь `match` выражается в выразительной синтаксической
конструкции и возможности перебора всех возможных значений.

Пример 6-3:

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

<span class="caption">Пример 6-3: перечисление и выражение `match`, которое использует
значение перечисления в качестве параметров</span>

Подробнее рассмотрим работу выражения `match`. Оно похоже на `if` с той разницей,
что может возвращать любые значения.

Внутри, выражение `match`, напоминает осьминога или сороконожку. Оно разделяется на лапки,
каждый из которых состоит из шаблона и кода. Первая лапка содержит значение `Coin::Penny`.
Далее следует оператор-разделитель `=>`, далее выполняемый код. В данном случае это
просто скалярное значение. Описание каждой лапки отделяется друг от друга запятой.

Когда выражение `match` выполняется, происходит сравнение результирующего значения с
шаблоном в каждой лапке по порядку. Если значение удовлетворяет правилам шаблона,
выполняется код, который связан с этим шаблоном. Если нет, проверка продолжается.

Код,  в каждой руке является выражением, т.е. возвращает значение.

Фигурный скобки в коде могут быть опущены, если выражение совсем простое:

```rust
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter,
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Шаблоны, которые связываются со значениями

Ещё одно полезное качество, которое есть у рук выражения `match` следующее:
они могут связывать части значений, которые удовлетворяют шаблону поиска.

К примеру, давайте изменим одно из значений перечисления, чтобы оно содержало в себе
значение.
Пример 6-4:

```rust
#[derive(Debug)] // So we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // ... etc
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

<span class="caption">Пример 6-4: Перечисление `Coin`, где одно из значений `Quarter`
содержит значение перечисления `UsState`</span>

Т.е. поиск может быть вложенным:

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}
```
Если Вы вызовите метод `value_in_cents(Coin::Quarter(UsState::Alaska))`, где переменная
`coin` имеет значение `Coin::Quarter(UsState::Alaska)`. То мы сможем вывести на
печать значение `UsState::Alaska`.

### Использования перечисления `Option<T>` в конструкции match

Перечисление `Option<T>` используется аналогичным образом.

Разберем работу с ним на примере. Возьмём перечисление  `Option<i32>`. Напишем функцию,
которая получает значение этого перечисления и содержит значение. Если значения нет,
оно имеет значение `None`.

Пример использования `match` и `Option<T>` 6-5:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);

    println!("{:?}", five);
    println!("{:?}", six);
    println!("{:?}", none);
}
```

<span class="caption">Пример использования `match` и `Option<T>`</span>

#### Анализ значений перечисления `Option<T>`. `Some(T)`

Давайте рассмотрим работу функции `plus_one` более подробно.

Значение `Some(5)` не равно `None`, поэтому анализ идёт дальше.

```rust,ignore
Some(i) => Some(i + 1),
```

Значение `Some(5)` соответствует шаблону `Some(i)`. Данные, содержащиеся в
`Some` используются и изменяются. Создаётся новое значение `Some(i)` с новыми данными
внутри.

#### `None`

Теперь рассмотрим `None` и его поиск по шаблону.

```rust,ignore
None => None,
```
Значения совпадают. Анализ останавливается на этом месте.

Использование перечисления в конструкции `match` весьма удобно и часто используется.
Понимание, как это работает, приходит с опытом. Надеюсь, что эта конструкция будет
вам помогать в работе.

### Перебор всех значений

Обратите внимание! Если мы уберем ветку возможных вариантов значения перечисления,
компилятор не скомпилирует код и сообщит об ошибке:

```rust,ignore
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```

Компилятор, проанализировав все возможные варианты перечисления, обнаружил, что какое-либо
из них не используется. Это ошибка.

### Использование `_`

В языке Rust есть специальный шаблон для `match`, который можно использовать, если
вы по каким-то причинам не хотите перебирать все значения перечисления. Т.е. `_`
используется для всех остальных вариантов. Например:

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```
Код шаблона `_` обрабатывает все остальные значения.  `()` это пустой объект.

Выражение `match` имеет сокращенную форму, о котором мы поговорим в следующей секции.