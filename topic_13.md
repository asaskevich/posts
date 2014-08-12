Scala: внедряемся в код
======

Тема не совсем верная, потому что я не буду сидеть с дизассамблером, декомпилятором и влезать в код посредством байткода. Я всего лишь хочу познакомиться с такими вещами, как трэйты `traits` и неявные классы `implicit classes`. Они позволяют дополнить уже существующие классы и объекты новыми функциями, так, как, например, это позволяет делать C#.

#### Трэйты

Трэйты позволяют доопределять классы новыми функциями или новыми вариантами старых. Однако есть одно замечание - трэйты ни в коем случае нельзя применить к классам с модификатором `final`. Давайте попробуем для начала расширить некоторый созданный нами класс, например такой маленький и простой:
```scala
scala> class User {
     |   var name = ""
     |   var age = 0
     | }
defined class User
```
Давайте добавим к классу функцию вывода фразы `Hello, my name is $name!`:
```scala
scala> trait SayHello extends User {
     |   def sayHi() : Unit = {
     |     println("Hello, my name is " + name + "!")
     |   }
     | }
defined trait SayHello
```
А теперь попробуем два способа - без использования трэйта и с его использованием (я делаю все в REPL, потому не стоит обращать на такую странную форму):
```scala
scala> var user_1 = new User
user_1: User = User@daa183
scala> user_1.name = "Bob"
user_1.name: String = Bob
scala> user_1.sayHi()
<console>:10: error: value sayHi is not a member of User
              user_1.sayHi()
                     ^
```
Как видим, в обычном классе данный метод отсутствует. Теперь попробуем применить трэйт к новому объекту:
```scala
scala> var user_2 = new User with SayHello
user_2: User with SayHello = $anon$1@64967f9e
scala> user_2.name = "John"
user_2.name: String = John
scala> user_2.sayHi()
Hello, my name is John!
```
Класс `User` был расширен трэйтом `SayHello`, потому мы смогли вызвать нужную нам функцию.

#### Неявные классы

Но что, если нам все таки очень нужно расширить класс, объявленный как `final`? Кажется, у нас есть выход! А выход вот в чем - мы можем описать новый тип с нужными функциями, полями, а затем описать способ конвертации между этим типом и расширяемым классом.
Попробуем расширить класс строк простой функцией печати самой строки. Для этого сначала опишем свой новый класс модифицированной строки:
```scala
scala> class MyString (s : String) {
     |   def print = println(s)
     | }
defined class MyString
```
Но мы пока еще не можем вызвать метод из обычного класса `java.lang.String`:
```scala
scala> "Hi".print
<console>:8: error: value print is not a member of String
              "Hi".print
                   ^
```
Для этого опишем способ конвертации `String <-> MyString`:
```scala
scala> implicit def str2MyStr(s : String) = new MyString(s)
warning: there were 1 feature warning(s); re-run with -feature for details
str2MyStr: (s: String)MyString
```
Все, теперь пробуем и радуемся:
```scala
scala> "Hi".print
Hi
scala> "Good Day".print
Good Day
```

#### Заключение 

Сейчас мы рассмотрели самый простой способ даже не внедрения, а скорее просто расширения уже готовых классов в Scala. Тесная связка с Java позволяет использовать многие возможности Scala, не прибегая к особым сложностям. Так, например, чтобы расширить строки новыми функциями, совершенно не нужно использовать ASM и реализовывать руками динамическую загрузку, модифицирование "на лету", и прочие сложные вещи. На мой взгляд, весьма отличная вещь, которой не хватает во многих языках.

#### Ссылки
* [Implicit Classes](http://docs.scala-lang.org/overviews/core/implicit-classes.html)
* [A Tour of Scala: Traits](http://docs.scala-lang.org/overviews/core/implicit-classes.html)
* [Содержание](README.md)
