## Типаж `Deref` позволяет получить доступ к данным по ссылке

Первый важный типаж связанный с умным указателем это `Deref`, который позволяет
нам перезаписать оператор `*` получения данных по ссылке. Перезапись данного оператора
для умного указателя даёт доступ к данным удобным.

Мы вкратце упоминали оператор разыменования в Главе 8, в хэш-карте в разделе
«Обновление значения на основе старого значения». У нас была изменяемая ссылка,
и мы хотели изменить значение, указываемое ссылкой. Для этого сначала нужно было
разыменовать ссылку. Вот другой пример, использующий ссылки на значения `i32`:

```rust
let mut x = 5;
{
    let y = &mut x;

    *y += 1
}

assert_eq!(6, x);
```
Мы используем `*y` для доступа к данным, к которым изменяемая ссылка `y` ссылается,
а не к изменяемой ссылке как таковой. Далее, мы можем изменить данные, в данном
случае добавив 1.

Для ссылок, которые не являются умными указателями, есть только одно значение, на
которое ссылается указатель. Поэтому операция разыменования очевидна. Умные указатели
также могут сохранять метаданные о указателях или данные. Когда разыменование происходит,
нам необходим доступ к данным, а не к метаданным. Поэтому разыменование обычного
указателя - это получение данных, а не метаданных. Нам необходимо иметь возможность
использовать умные указатели в том же месте, где мы используем обычные указатели.
Для того, чтобы это было возможным, нам нужно перезаписать поведение оператора `*`,
реализовав типаж `Deref`.

Код 15-7 демонстрирует пример перезаписи оператора `*` используя `Deref` для
структуры, которая хранит данные mp3 и метаданные. `Mp3`, как умный указатель
владеет данными  `Vec<u8>` содержащими аудио. Дополнительно, структура содержит
опциональные метаданные (в данном случае, описание артиста и заголовок).
Мы хотим иметь возможность удобного доступа к данным аудио, не к метаданным. Для
того реализуем типаж `Deref`.

Для реализации типажа `Deref` необходимо реализовать метод `deref`, которые заимствует
`self` и возвращает внутренние данные:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::ops::Deref;

struct Mp3 {
    audio: Vec<u8>,
    artist: Option<String>,
    title: Option<String>,
}

impl Deref for Mp3 {
    type Target = Vec<u8>;

    fn deref(&self) -> &Vec<u8> {
        &self.audio
    }
}

fn main() {
    let my_favorite_song = Mp3 {
        // we would read the actual audio data from an mp3 file
        audio: vec![1, 2, 3],
        artist: Some(String::from("Nirvana")),
        title: Some(String::from("Smells Like Teen Spirit")),
    };

    assert_eq!(vec![1, 2, 3], *my_favorite_song);
}
```

<span class="caption">код 15-7: реализация типажа `Deref`</span>

Большая часть кода думаю вам понятна: объявление структуры, реализация типажа и
функция main, в которой создаётся экземпляр структуры. Мы только не объясняли ещё
тему ассоциированных типом (оставим это до Главы 19). Это всё лишь другой способ
объявления обобщенного параметра.

В макросе `assert_eq!` мы сравниваем вектор `vec![1, 2, 3]` и результат разыменования
ссылки на экземпляр `Mp3` `*my_favorite_song` (результат использования реализации
типажа Deref). Типаж `Deref` не будет реализован, компилятор сообщит об ошибке
(пожалуйста, проверьте).

Без реализации типажа `Deref` компилятор может только разыменовать `&` ссылки.
В данном примере мы получим значение экземпляра `Mp3`. Благодаря реализации типажа
`Deref` компилятор знает, что те типы данных, которые реализовали типаж `Deref`
реализовали метод, который возвращает ссылку (в данном случае `&self.audio`).
Для того, чтобы ссылка могла быть разыменована, компилятор неявным образом преобразует
выражение `*my_favorite_song` в:

```rust,ignore
*(my_favorite_song.deref())
```

Результатом разыменования будет значение в поле `self.audio`.  Причина по которой
метод `deref` возвращает ссылку является владение: если метод вернёт значение,
значение переместится из `self`. Целью работы метода не является передача владения,
а передача ссылки на корректное значение или экземпляр.

Обратите внимание, что при разыменовании с помощью символа `*` вызывается метод
`deref`.

### Разыменование явным образом с помощью функций и методов

В Rust предпочтение отдаётся явном действиям компилятора. Но при этом есть исключения.
При разыменовании аргументов функций и методов. При этом происходит автоматическая
конвертация ссылки в указатель внутри объекта, на который данная ссылка ссылается.
Принудительное разыменование происходит, когда аргумент ссылочного типа передаётся
в функцию отличается от ссылочного аргумента определённого в функции. Принудительное
разыменование добавлено в язык Rust для упрощения вызова функций и методов для
краткости кода. Чтобы не было явного множества использования ссылок и разыменований
с помощью символов `&` и `*`.

Приведём пример (15-7). В функции `compress_mp3:

```rust,ignore
fn compress_mp3(audio: &[u8]) -> Vec<u8> {
    // the actual implementation would go here
}
```
Если в Rust не будет принудительного разыменования, то эту функцию пришлось
переписать следующим образом:

```rust,ignore
compress_mp3(my_favorite_song.audio.as_slice())
```

В этом случае мы должны явным образом сообщить, что мы хотим получить данные из
поля `audio` переменной `my_favorite_song`. Если данные поля будут интенсивно
использоваться, в этом случает вызов метода `.audio.as_slice()` будет постоянным.

Но т.к. существует принудительное разыменование и мы реализовали типаж `Deref` в
`Mp3`, мы можем вызвать функцию таким образом:

```rust,ignore
let result = compress_mp3(&my_favorite_song);
```

Получение ссылок с помощью `&` это замечательно. Мы можем интерпретировать умные
указатели, как регулярные, постоянные ссылки. Процесс разыменования значит, что
Rust может использовать реализацию типажа `Deref`. Также Rust известно, что реализация
этого типажа `Vec<T>` возвращает `&[T]`. Компилятор видит, что можно использовать
функцию `Deref::deref` дважды чтобы получить `&Vec<u8>` из `&Mp3` и далее `&[T]`.
Весь это анализ и преобразования делаются в момент компиляции.

Таким же образом (`*` → `&T`) работает реализация типажа `DerefMut` (`*` → `&mut T`).

Компилятор производит принудительное разыменование в следующих случаях:

* Из `&T` в `&U` в случае `T: Deref<Target=U>`.
* Из `&mut T` в `&mut U` в случае `T: DerefMut<Target=U>`.
* Из `&mut T` в `&U` в случае `T: Deref<Target=U>`.

Первые два случая похожи (за исключением изменяемости ссылки): если у вас есть
`&T` и `T` реализует `Deref` для типа `U`, вы можете получить тип `&U` неявным
образом. Также для изменяемых ссылок.

Последний более сложный и немного странный: если у вас есть изменяемая ссылка, она
также может превратиться в неизменяемую. Обратное преобразование невозможно, т.к.
неизменяемая ссылка никогда не может быть превращена в изменяемую.

Причина, по которой типаж `Deref` важен для шаблона умных указателей, является
то что эти указатели могут обрабатываться как регулярные ссылки и использоваться в
местах, которые ожидают регулярных ссылок. Нам не нужно переопределять методы и
например, для явного указания умных указателей.