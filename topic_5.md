Java & API: Легкий старт с Jsoup
======

Иногда необходимо обработать данные, представленные в виде HTML. Конечно, проще всего работать с такими данными в JavaScript, однако в других языках также существуют отличные решения, решающие данную проблему. Так, например, в Java это [Jsoup](http://jsoup.org/). Рассмотрим самые базовые операции, которые мы можем выполнить при помощи библиотеки.

### Загрузка HTML страницы

Веб-страницу мы можем запустить из файла, напрямую из текста или из Интернета.
```java
Document doc = Jsoup.connect("http://github.com/").get();
...
String html = "<html><head><title>First parse</title></head><body><p>Hello, world!</p></body></html>";
Document doc = Jsoup.parse(html);
...
File input = new File("/input.html");
Document doc = Jsoup.parse(input, "UTF-8", "http://example.com/");
```

### Работа с селекторами

Для выбора отдельных элементов документа мы можем использовать селекторы, форма записи которых эквивалентна селекторам JS:
```java
Elements links = doc.select("a[href]");
Elements forms = doc.select(".form");
```
Селекторы можно комбинировать: `doc.select("#mp-itn b a");`

### Выделение аттрибутов, текста и прочего из элементов

В следующем примере мы можем увидеть, как при помощи библиотеки получить все ссылки, выбрать первую, ее текст, аттрибуты, ее код и прочее: 
```java
String html = "<p>An <a href='http://example.com/'><b>example</b></a> link.</p>";
Document doc = Jsoup.parse(html);
Element link = doc.select("a").first();

String text = doc.body().text(); // "An example link"
String linkHref = link.attr("href"); // "http://example.com/"
String linkText = link.text(); // "example""

String linkOuterH = link.outerHtml(); 
    // "<a href="http://example.com"><b>example</b></a>"
String linkInnerH = link.html(); // "<b>example</b>"
```

### Обработка URL'ов

Тут тоже ничего сложного. Добавив параметр `abs:`, мы можем получить полный путь для URL. Данный прием полезен, например, для обработки веб-страниц, ссылки которых обычно являются не абсолютными, а относитльными:
```java
Document doc = Jsoup.connect("http://jsoup.org").get();

Element link = doc.select("a").first();
String relHref = link.attr("href"); // == "/"
String absHref = link.attr("abs:href"); // "http://jsoup.org/"
```

### Изменение аттрибутов элемента

Делается это так (мне даже нечего тут прокомментировать):
```java
doc.select("div.masthead").attr("title", "jsoup").addClass("round-box");
```

### Работа с HTML элементов

Изменить содержимое элемента, добавить перед ним и после него элементы можно так:
```java
Element div = doc.select("div").first(); // <div></div>
div.html("<p>lorem ipsum</p>"); // <div><p>lorem ipsum</p></div>
div.prepend("<p>First</p>");
div.append("<p>Last</p>");
// теперь: <div><p>First</p><p>lorem ipsum</p><p>Last</p></div>
```
Обернуть элемент внешним элементов мы можем так:
```java
Element span = doc.select("span").first(); // <span>One</span>
span.wrap("<li><a href='http://example.com/'></a></li>");
// теперь: <li><a href="http://example.com"><span>One</span></a></li>
```

### Заключение

Как видим, работа с библиотекой не доставит нам трудностей. Потому она может использоваться для обработки HTML, XML документов. Также при помощи библиотеки можно написать клиент для любого сайта - библиотека позволяет работать с http-запросами, куками и прочим. Пример можно увидеть в моем репозитории [VK_Small_API](http://github.com/asaskevich/vk-small-api), где я делал попытку создать библиотеку для работы с сайтом [vk.com](http://vk.com).

### Ссылки:
* [Официальный сайт](http://jsoup.org/)
* [Документация по Jsoup](http://jsoup.org/apidocs/)
* [Полезные приемы при работе с библиотекой](http://jsoup.org/apidocs/)
* [Try Jsoup](http://try.jsoup.org/)
* [Содержание](README.md)
