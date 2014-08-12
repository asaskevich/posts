JS: Sails.js + Backbone.js = Hello, TODO
======

Сегодня я решил - пора взяться за веб со стороны Node.js. А именно я решил присмотреться, каково вообще вести разработку на серверном JavaScript. В качестве серверного фреймворка я взял Sails.js - он мне приглянулся хорошей документацией и наличием некоторого количества примеров. В связке с ним я взял Backbone.js для клиентской части - после первого знакомства стоит поработать с ним поглубже. А общей целью поставил написать небольшой ToDo менеджер. Поехали!

#### Установка

Про установку и настройку серверной части говорить не интересно, потому приведу команды, которые я выполнил в консоли (предварительно установив node.js и npm):
```sh
npm install -g sails
sails new todo
cd todo
sails generate api Task
sails generate controller Main
```
Обращу внимание на то, что я сделал в конце: я создал новое приложение `todo`, sails любезно создал всю структуру файлов, затем я создал API: шаблон для модели `Task` и соответствующий ему RESTful контроллер, а также контроллер `Main`. Все это интересно и непонятно. `Task` будет хранить информацию о задачах, а контроллер выводить страницу с их списком. Создав модель, sails так же любезно создал нам набор RESTful API с CRUD методами так, что уже сейчас, запустив приложение и перейдя по адресу `localhost:1337/Task/` можем увидеть все модели `Task`. Скажем, не модели, а объекты.

#### Модель `Task`

Далее идем в `api\models\Task.js`, где создаем все поля для модели. Нам хватит поля текста задачи и логического поля, выражающего завершенность задачи:
```javascript
module.exports = {
	attributes: {
	  // Поле complete
		complete: {
			type: "boolean",
			defaultsTo: false
		},
		// Поле text
		text: {
			type: "string"
		}
	}
};
```
В принципе тут мы со всем разобрались. У нас есть два поля, мы определили их название и типы. Перейдем к контроллеру.

#### Контроллер `Main`

Идем в `api\controllers\MainController.js`. Там создаем следующее содержимое:
```javascript
module.exports = {
 	index: function (req, res) {
 		res.view();
 	}
};
```
Теперь заглянем в `confir\routes.js`. Здесь мы прописываем, что хотим вызывать функцию `index` из нашего контроллера по запросу корня сайта (За нас конечно уже прописан шаблон `homepage`, но ведь это не то, что мы хотим), потому делаем так:
```javascript
module.exports.routes = {
  '/' : {
    controller: 'main',
    action: 'index'
  }
};
```
Попробуем все это дело запустить. Выполняем `sails lift` и получаем ошибку, что у нас не стоит адаптер `sails-disk` для работы с локальной БД. Ставим и пробуем еще раз. Запустилось! Переходим на сайт и получаем ошибку - не задан шаблон для контроллера. 

#### Шаблоны

Создадим в папке `view\main\` файл `index.ejs` - файл нашего будущего шаблона. Собственно название шаблона мы могли прописать сразу в роутере, но через контроллер нагляднее и интереснее. Содержимое файла будет следующим:
```html
<!DOCTYPE html>
<html>
<head>
	<title>ToDo</title>
</head>
<body>
	<div>Записей: <%= count %></div>
	<div class="new_todo"><input id="new"></input><button id="add">Add</button></div>
	<div id="tasks">
	</div>
</body>
</html>
```
Обратим внимание на `<%= count %>` - это параметр, который мы передадим в контроллере вот так:
```javascript
res.view( {count: ___} );
```
Где вместо пропуска будет получение общего количества записей.

#### Клиентская часть

Для начала подключим все необходимые библиотеки:
```html
<script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script src="//cdnjs.cloudflare.com/ajax/libs/underscore.js/1.4.4/underscore-min.js"></script>
<script src="//cdnjs.cloudflare.com/ajax/libs/backbone.js/1.0.0/backbone-min.js"></script>
```
Создадим нашу модель, указав адрес, по которому она обрабатывается:
```javascript
var TaskModel = Backbone.Model.extend({
	urlRoot: '/Task',
});
```
Далее нужно описать коллекцию, которая хранит все модели:
```javascript
var TaskCollection = Backbone.Collection.extend({
	url: '/Task',
	model: TaskModel,
});
```
Проинициализируем коллекцию и загрузим ее с сервера:
```javascript
var tasks = new TaskCollection();
tasks.fetch();
```
Далее привяжем к нашей кнопке событие добавления записи:
```javascript
$("#add").click(function(){
	var taskText = $("#task").val();
	tasks.create({text: taskText}, {wait: true});
	$("#task").val("");
});
```
Теперь осталось описать вид и рендер коллекции. Шаблон будет таким: `{{  }}`, дабы не было конфликтов с шаблонами Sails.js:
```javascript
_.templateSettings = {
	interpolate : /\{\{(.+?)\}\}/g
};
```
Теперь осталось самую малость - описать вид:
```javascript
var TaskView = Backbone.View.extend({
	el: '#tasks',
	initialize: function () {
		this.collection.on('add', this.render, this);
		this.render();
	},
	template: _.template("<li>{{ text }}</li>"),
	render: function () {
		this.$el.html("");
		this.collection.each(function(msg){
			this.$el.append(this.template(msg.toJSON()));
		}, this)
	}
});
```
Все, создаем объект вида и тестируем: `var mView = new TaskView({collection: tasks});`. Кнопка добавления добавляет новую запись сразу в коллекцию, а также синхронизируется с сервером, потому, если перезагрузить страницу, данные никуда не пропадут. Конечно, стоит позаботиться о серверной части, дабы в реальном приложении нельзя было удалить записи из базы, просто перейдя по нужной ссылке API.

#### Заключение

Скажу просто - это круто. Никаких особых заморочек с БД, нету особых сложностей с клиентской частью, да и с серверной. По сути - описать контроллеры, модели, шаблоны, сделать их связку, описать работу моделей и коллекций в клиентской части - и все готово. Ради интереса гляну еще некоторые MVC фреймворки, не только для Node.js, но и для других языков.

#### Ссылки
* [Sails.js](http://sailsjs.org/#/)
* [Backbone.js](http://backbonejs.org/)
* [Working With Data in Sails.js](http://code.tutsplus.com/tutorials/working-with-data-in-sailsjs--net-31525)
* [Содержание](README.md)
