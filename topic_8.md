Clojure: второе знакомство
======

Продолжая знакомство с языком, я решил воспользоваться книгой ["Clojure for the Brave and True"](http://www.braveclojure.com/). Удобно, понятно, самое главное - обилие примеров, простых и сложных. Что ж, продолжим статью [Clojure: первое знакомство](topic_7.md).

#### Условный оператор

Структура оператора вполне привычна для других языков, за исключением того, что скобки располагаются не в зоне условия, а вокруг всего блока оператора:
```clojure
(if условие
  то-действие
  иначе-действие
)
```
Напишем маленький пример:
```clojure
user=> (if false "True" "False")
"False"
user=> (if true "True" "False")
"True"
```
Единственное ограничение состоит в том, что здесь мы можем использовать только одно действие (написать сложную конструкцию из других операторов и функций не получится). Специально для этого в языке есть оператор `do`

#### Оператор `do`

Оператор служит для объединения вызова нескольких команд вместе:
```clojure
user=> (do
        (println "One")
        (println "Two")
        (println "Three")
       )
One
Two
Three
nil
```
Теперь мы можем переписать условие так, чтобы оно вывело, например результат умножения `5 * 5` и строку "Hi":
```clojure
user=> (if (= 10 10)
          (do
                  (println (* 5 5))
                  (println "Hi")
          )
          (println "Nothing")
        )
25
Hi
nil
```
Красота да и только!

#### Оператор `when`

Данный оператор как бы является комбинацией `if` и `do`, однако без формы `else`:
```clojure
user=>  (when (= 10 10)
         (println (* 5 5))
         (println (+ 5 5))
        )
25
10
nil
```

#### Объявление переменных

Вообще, большинство переменных в языке не совсем переменные - они неизменяемые. Однако это не помешает нам переприсваивать новое значение старой переменной:
```clojure
user=> (do
        (def x 10)
        (println x)
        (def x "Some Text")
        (println x)
       )
10
Some Text
nil
```
Мы объявили переменную `x`, присвоили ей значение `10`, а потом изменили ее значение и тип на строку. Можно сделать и так:
```clojure
user=> (def x 10)
#'user/x
user=> (def x (+ x 1))
#'user/x
user=> x
11
```
Объявили переменную и увеличили ее значение на единицу. Не исключаю, что есть более элегантная форма решения этого действия.

#### Заключение

Рассмотрели, всего-то, операторы `if`, `def`, `do`, `when`. В книге, ссылка которой указана в самом начале, следующей темой будут структуры данных. Тема весьма крупная, потому я выделю под нее отдельную статью.

#### Ссылки
* [Clojure](http://clojure.org/)
* [Clojure Docs](https://clojuredocs.herokuapp.com/)
* [Clojure cheatsheet](http://clojure.org/cheatsheet)
* [Try Clojure](http://tryclj.com)
* [Почему стоит изучить Clojure? / Хабрахабр](http://habrahabr.ru/post/173071/)
* [Содержание](README.md)
