Java & API: работа с iCalendar
======

Не так давно у меня возникла необходимость парсинга и последующего вывода нужной информации из [iCalendar](http://en.wikipedia.org/wiki/ICalendar). После упорных поисков я наткнулся на библиотеку iCal4J. Присмотревшись к её функционалу, я понял — это то, что мне нужно. Давайте же попробуем её применить на практике.

### Подготовка календаря

В Google Calendar я создал маленький календарь, состоящий из нескольких различных мероприятий длительностью от 30 минут до 6 часов. Затем я скачал календарь в формате iCal к себе на ноутбук. Если кто не знает — делается это так: заходим в настройки календаря Google и в разделе «Закрытый адрес календаря» жмём на зеленую кнопочку «ICAL». Всё, календарь загружен на устройство. 

### Подготовка Java

Следующим этапом стала загрузка библиотеки и подключение ее к проекту. Распаковал и указал среде разработки папку с файлами библиотек *.jar. Также в папку с проектом был скопирован мой скачанный файл календаря. Все готово к написанию кода!

### Загрузка календаря

Существует два варианта загрузки календаря — из файла и парсинг строки с содержимым календаря. Мы выберем первый способ (второй существенно не отличается от него).

```java
// Открываем входной поток из файла
FileInputStream fin = new FileInputStream("calendar.ics");
// И на основе этого потока парсим календарь в объект Calendar
CalendarBuilder builder = new CalendarBuilder();
Calendar calendar = builder.build(fin);
```

### Вывод всех мероприятий

Давайте теперь попробуем вывести куда — нибудь (ну хотя бы в консоль) все мероприятия из нашего календаря. 

```java
// Получаем список мероприятий
ComponentList listEvent = calendar.getComponents(Component.VEVENT);
for (Object elem : listEvent) {
    // Нам придется явно привести элемент списка к нужному типу, так как listEvent хранит элементы Object
    VEvent event = (VEvent) elem;
    // Получаем описание
    // getValue() делаем для того, чтобы извлечь только значение. Доступные методы getName() и toString() вернут в себе еще значение тега DESCRIPTION
    String description = event.getDescription().getValue();
    // Получаем заголовок мероприятия 
    String title = event.getSummary().getValue();
    System.out.println(title + " : " + description);
}
```

Стоит обратить внимание на первую строчку, а именно `Component.VEVENT`. Так мы указываем, что хотим получить именно мероприятия календаря. Если не указать явно, что мы хотим извлечь из календаря, то мы получим список, в котором также окажутся To-Do компоненты (`VTODO`), заметки (`VJOURNAL`), и прочие компоненты типа `VFREEBUSY`, `VALARM` и `VTIMEZONE`.

### Применение фильтров

Для того, чтобы вывести элементы, в моем случае мероприятия, в определенный период времени, можно применить фильтры. 

```java
java.util.Calendar today = java.util.Calendar.getInstance();
today.set(java.util.Calendar.HOUR_OF_DAY, 0);
today.clear(java.util.Calendar.MINUTE);
today.clear(java.util.Calendar.SECOND);
// Создаем период времени в 1 день
Period period = new Period(new DateTime(today.getTime()), new Dur(1, 0, 0, 0));
// На его основе создаем фильтр
Filter filter = new Filter(new PeriodRule(period));
// Применяем фильтр к календарю
List eventsToday = filter.filter(calendar.getComponents(Component.VEVENT));
// Всё, в eventsToday хранятся все мероприятия, запланированные именно на сегодня
```

На момент написания в моем календаре хранились такие записи (в формате «Заголовок»: «Описание» [«Время начала»]):
```java
Event 1: Event 1 — November 1 [7 Nov 2013 13:00:00 GMT]
Event 5: Event 5 — November 12 [12 Nov 2013 12:00:00 GMT]
Event 3: Event 3 — November 10 [10 Nov 2013 19:30:00 GMT]
Event 2: Event 2 — November 9 [9 Nov 2013 08:30:00 GMT]
Event 4: Event 4 — November 11 [11 Nov 2013 08:00:00 GMT]
Event 6: Event 6 — November 13 [13 Nov 2013 09:30:00 GMT]
```

После применения фильтра на вывод запланированных мероприятий (на 11 ноября) я получил только одно мероприятие — Event 4:
`Event 4: Event 4 — November 11 [11 Nov 2013 08:00:00 GMT]`

Увеличив период до трёх дней…
`Period period = new Period(new DateTime(today.getTime()), new Dur(3, 0, 0, 0));`

… я получил такую картину:
```java
Event 5: Event 5 — November 12 [12 Nov 2013 12:00:00 GMT]
Event 4: Event 4 — November 11 [11 Nov 2013 08:00:00 GMT]
Event 6: Event 6 — November 13 [13 Nov 2013 09:30:00 GMT]
```

### Создание пустого календаря

iCal4j поддерживает также создание календаря с нуля. Здесь всё просто. Создаем объект календаря, добавляем в него необходимые поля.

```java
Calendar calendar = new Calendar();
calendar.getProperties().add(new ProdId("-//habrahabr"));
calendar.getProperties().add(Version.VERSION_2_0);
calendar.getProperties().add(CalScale.GREGORIAN);
```

### Добавление нового мероприятия в календарь

Вот с созданием мероприятия все не так просто, но если разобраться, то все пойдет достаточно быстро.

```java
// Устанавливаем часовой пояс
TimeZoneRegistry registry = TimeZoneRegistryFactory.getInstance().createRegistry();
TimeZone timezone = registry.getTimeZone("Europe/Minsk");
VTimeZone tz = timezone.getVTimeZone();
 // Начало 10 ноября в 18:00
java.util.Calendar startDate = new GregorianCalendar();
startDate.setTimeZone(timezone);
startDate.set(java.util.Calendar.MONTH, java.util.Calendar.NOVEMBER);
startDate.set(java.util.Calendar.DAY_OF_MONTH, 10);
startDate.set(java.util.Calendar.YEAR, 2013);
startDate.set(java.util.Calendar.HOUR_OF_DAY, 18);
startDate.set(java.util.Calendar.MINUTE, 0);
startDate.set(java.util.Calendar.SECOND, 0);
 // Конец в этот же день в 20:00
java.util.Calendar endDate = new GregorianCalendar();
endDate.setTimeZone(timezone);
endDate.set(java.util.Calendar.MONTH, java.util.Calendar.NOVEMBER);
endDate.set(java.util.Calendar.DAY_OF_MONTH, 10);
endDate.set(java.util.Calendar.YEAR, 2013);
endDate.set(java.util.Calendar.HOUR_OF_DAY, 20);
endDate.set(java.util.Calendar.MINUTE, 0);	
endDate.set(java.util.Calendar.SECOND, 0);
// Создаем событие
String eventName = "Какое-то мероприятие";
DateTime start = new DateTime(startDate.getTime());
DateTime end = new DateTime(endDate.getTime());
VEvent meeting = new VEvent(start, end, eventName);
// Добавляем к нему информацию о часовом поясе
meeting.getProperties().add(tz.getTimeZoneId());
// Создаем сам календарь
Calendar icsCalendar = new Calendar();
icsCalendar.getProperties().add(new ProdId("-//habrahabr"));
icsCalendar.getProperties().add(CalScale.GREGORIAN);
// Добавляем к нему созданное событие
icsCalendar.getComponents().add(meeting);
```

### Сохранение календаря в файл

Поработав с календарем, нам конечно же нужно его сохранить. Лучше всего сохранить обратно в файл.

```java
FileOutputStream fout = new FileOutputStream("calendar.ics");
CalendarOutputter out = new CalendarOutputter();
out.output(calendar, fout);
```

### Заключение

Мы рассмотрели базовые возможности библиотеки iCal4j. Конечно, её возможности не ограничиваются просто парсингом календаря и созданием событий. Библиотека позволяет также прикреплять файлы к событиям, создавать мероприятия с указанием места проведения в координатах, создавать списки приглашенных и многое другое. Помимо этого библиотека может использоваться в разработке Android приложений.

### Ссылки:
* [Описание iCalendar](http://en.wikipedia.org/wiki/ICalendar)
* [Документация по iCal4j](http://build.mnode.org/projects/ical4j/apidocs/index.html)
* [iCal4j Wiki (здесь же можно и загрузить библиотеку)](http://wiki.modularity.net.au/ical4j/index.php?title=Main_Page)
* [Содержание](README.md)

###### Маленькая заметка
Данная статья была первоначально опубликована вот [здесь](http://habrahabr.ru/post/201660/).
