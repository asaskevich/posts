Moment.js: работа с датами в JavaScript
======

Очень часто необходимо работать с информацией типа времени и дат, обрабатывая их таким образом, как этого требует программа. Существует огромное множество решений, каждое из которых умеет выполнять какие-то информации с датами - одни лучше, другие хуже. Среди этого разнообразия существует библиотека [Moment.js](http://momentjs.com/), которая умеет больше, чем не умеет.

<!-- more -->
### Установка
Для использования библиотеки на стороне сервера нужно установить ее при помощи `npm` или `bower`:
```bash
bower install moment --save
npm install moment --save 
```
Для использования на стороне клиента нужно скачать необходимые файлы и подключить их на вашу страницу.

### Форматирование дат
Начнем с того, что научимся превращать текущее время в удобочитаемый вид. Библиотека поддерживает много локализаций, среди них русский и английский. Все делaется просто - получаем текущее время и дату и форматируем его вывод:
```javascript
moment().format('MMMM Do YYYY, h:mm:ss a'); // сентябрь 3-го 2014, 8:02:18 вечера
moment().format('dddd');                    // среда
moment().format("MMM Do YY");               // сен 3-го 14
moment().format('YYYY [, а месяц] MMMM');   // 2014 , а месяц сентябрь
moment().format();                          // 2014-09-03T20:02:18+03:00
```

### Относительное время
Moment.js поддерживает вывод локализированной информации о том, сколько времени прошло/осталось с какого-то события. Удобно, если нужно использовать, например, при выводе информации о времени создания публикации в блоге или личного сообщения, например `User оставил вам сообщение 5 минут назад`:
```javascript
moment("20111031", "YYYYMMDD").fromNow(); // 3 года назад
moment("20120620", "YYYYMMDD").fromNow(); // 2 года назад
moment().startOf('day').fromNow();        // 20 часов назад
moment().endOf('day').fromNow();          // через 4 часа
moment().startOf('hour').fromNow();       // 5 минут назад
```

### Работа с датами
Библиотека позволяет не только форматировать даты, а еще вычитать или изменять их:
```javascript
moment().subtract(10, 'days').calendar(); // 24.08.2014
moment().subtract(6, 'days').calendar();  // В прошлый четверг в 20:07
moment().subtract(3, 'days').calendar();  // В прошлое воскресенье в 20:07
moment().subtract(1, 'days').calendar();  // Вчера в 20:07
moment().calendar();                      // Сегодня в 20:07
moment().add(1, 'days').calendar();       // Завтра в 20:07
moment().add(3, 'days').calendar();       // В субботу в 20:07
moment().add(10, 'days').calendar();      // 13.09.2014
```

### Работа с часовыми поясами
Помимо всего прочего, библиотека имеет подмодуль [Moment Timezone](http://momentjs.com/timezone/), позволяющий работать с часовыми поясами:
```javascript
var jun = moment("2014-06-01T12:00:00Z");
var dec = moment("2014-12-01T12:00:00Z");

jun.tz('Australia/Sydney').format('ha z');     // 10pm EST
dec.tz('Australia/Sydney').format('ha z');     // 11pm EST

// Конвертация между часовыми поясами
var newYork    = moment.tz("2014-06-01 12:00", "America/New_York");
var losAngeles = newYork.clone().tz("America/Los_Angeles");
var london     = newYork.clone().tz("Europe/London");

newYork.format();    // 2014-06-01T12:00:00-04:00
losAngeles.format(); // 2014-06-01T09:00:00-07:00
london.format();     // 2014-06-01T17:00:00+01:00
```

### Ссылки
* [Moment.js](http://momentjs.com/)
* [Moment Timezone](http://momentjs.com/timezone/)
* [Содержание](README.md)
