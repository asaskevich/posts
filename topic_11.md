JS: Backbone.js, say "Hello"
======

Пусть для JavaScript'a существует достаточное число фреймворков, создавать динамические веб-приложения все же достаточно сложно. А когда дело доходит до работы с DOM, событиями и прочими прелестями HTML, JS и CSS, то код становится нереально большим и сложным в поддержке. Backbone призван решить эту проблему. Как написано на сайте русской документации Backbone:
```
Backbone.js придает структуру веб-приложениям с помощью моделей с биндингами по ключу и пользовательскими событиями, коллекций с богатым набором методов с перечислимыми сущностями, представлений с декларативной обработкой событий; и соединяет это все с вашим существующим REST-овым JSON API.
```
Что ж, было бы неплохо попробовать его в действии.

#### Основа

За HTML-основу один из сайтов предлагает следующее:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
</head>
<body>
  <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js" type="text/javascript"></script>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.3.3/underscore-min.js" type="text/javascript"></script>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/backbone.js/0.9.2/backbone-min.js" type="text/javascript"></script>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/backbone-localstorage.js/1.0/backbone.localStorage-min.js" type="text/javascript"></script>  
</body>
</html>
```
Все необходимое подключено и мы можем начать наши эксперименты.

#### Связывание вида и его рендера

Фреймворк позволяет назначать DOM элементам свои шаблоны, рендер и многое другое. Давайте попробуем.
Для начала создадим DOM элемент:
```html
<div id="example">Example</div>
```
Перейдем к JS. Для начала создадим пустой вид:
```javascript
var AppView = Backbone.View.extend({
  // el - наш элемент, с которым ассоциирован вид, в данном случае <div> с id "example"
  el: '#example'
});
```
Затем нам необходимо указать функцию рендера:
```javascript
var AppView = Backbone.View.extend({
    // el - наш элемент, с которым ассоциирован вид, в данном случае <div> с id "example"
    el: '#example',
    // $el - объект jQuery, потому к нему могут быть применены любые функции jQuery,
    // так например html()
    render: function () {
        this.$el.html("Hello World");
    }
});
```
Но сама себя эта функция не вызовет, ее нужно вызвать, например при инициализации:
```javascript
var AppView = Backbone.View.extend({
    // el - наш элемент, с которым ассоциирован вид, в данном случае <div> с id "example"
    el: '#example',
    // Эта функция вызывается при инициализации вида
    initialize: function () {
        this.render();
    },
    // $el - объект jQuery, потому к нему могут быть применены любые функции jQuery,
    // так например html()
    render: function () {
        this.$el.html("Hello World");
    }
});
```
Осталось создать объект вида для дальнейшей работы:
```javascript
var appView = new AppView();
```

#### Шаблоны

Так как Backbone жестко завязан на underscore.js, мы можем воспользоваться множеством различных утилит и функций из нее.  Так, например, мы можем использовать шаблоны. Синтаксис вызова такой: `_.template(шаблон, [данные], [параметры])`.
Итак, добавим в наш вид поле шаблона:
```javascript
var AppView = Backbone.View.extend({
  ...
  template: _.template("<h3>Hello <%= who %><h3>"),
  ...
});
```
Обратите внимание на `<%= who %>`. Это параметр, который будет загружен из аргумента "данные". Все, что нам нужно, вызвать шаблон в рендере. Например так:
```javascript
render: function () {
    this.$el.html(this.template({who: 'man!'}));
}
```
Разницы между `<%= %>` и `<%- %>` нету. Однако при помощи `<% %>` мы можем выполнять любой JS код.

#### Заключение

Изначально кажется, что мы проделали слишком много действий лишь для того, чтобы вывести стандартную надпись `Hello World`. Да, это так. Но продолжив изучение фреймворка и используя его в крупном веб-приложении, мы несомненно увидим, что мы сохранили много времени и сил, не пытаясь изобретать велосипед при создании динамического приложения. Далее в планах познакомится с фреймворком глубже, а также обратить внимание на Angular.js - фреймворк с похожей задачей - упростить создание динамических веб-приложений.

#### Ссылки
* [Документация Backbone.js](http://backbonejs.ru/)
* [Документация underscore.js](http://underscorejs.ru/)
* [Backbone.js for Absolute Beginners](http://adrianmejia.com/blog/2012/09/11/backbone-dot-js-for-absolute-beginners-getting-started/)
* [Содержание](README.md)
