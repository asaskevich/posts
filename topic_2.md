JS & API: StackExchange API на пальцах
======
Иногда возникают ситуации, когда необходимо написать скрипт или приложение, взаимодействующее с каким-либо сервисом. Многие крупные сайты предлагают разработчикам свое API, которое они могут использовать в своих проектах. Однако у каждого сервиса свои функции и способы работы с API. Потому нередко сложно начать использовать сторонние API в своих проектах.

На примере небольшого приложения, авторизирующего пользователя и получающего его последние уведомления с данными профиля, мы рассмотрим, как пользоваться Stack Exchange API.

### 1. Регистрация нового приложения
Прежде чем создавать свое приложение, необходимо зарегистрировать его в StackApps. Это необходимо, чтобы получить ключи доступа для него. Переходим по [этой ссылке](stackapps.com/apps/oauth/register) и заполняем все поля, не помеченные пунктом optional. Для своего туториала я заполнил поля так:

![](http://habrastorage.org/getpro/habr/post_images/81f/0d0/9c7/81f0d09c7cb9bbdf98d797e4e9247b43.png)

Не забываем поставить галочку в `«Enable Client Side OAuth Flow»`. Это необходимо для OAuth — иначе пользователь просто не сможет войти в систему, пользуясь приложением. Когда все заполнили, жмем соответствующую кнопку, и переходим на страницу управления приложением. Там мы получим несколько ключей (`Client Id`, `Client Secret`, `Key`). Из них для самого приложения нужны только `ClientId` и `Key`. Сохраняем их и на этом мы завершаем процесс регистрации и переходим непосредственно к написанию самого приложения.

### 2. Начинаем
Создадим новый чистый HTML файл и сразу подключим в нем скрипт:
```
<script type='text/javascript' src='https://api.stackexchange.com/js/2.0/all.js'></script>
```
Теперь мы можем воспользоваться базовыми функциями инициализации и авторизации для дальнейшей работы. Помимо `key` и `clientId`, для инициализации нам необходимо передать два дополнительных параметра - домен и callback-функция:
```
SE.init({
    clientId: 1, // Здесь мы ставим выданный нам clientId
    key: 'YoUrAweSomeKey', // А здесь соответственно key
    channelUrl: 'http://example.com/', // Особое внимание стоит уделить этому полю. Здесь нужно указать домен, на котором хостится и крутится приложение
    complete: function (data) { alert("Я загрузился!"); } // Здесь мы указываем некоторую функцию, которая будет выполнена в случае успеха
});
```
Если вы все сделали верно, то при загрузке страницы должен выскакивать alert :) Иначе проверьте вывод консоли — нет ли там чего подобного на `Uncaught channelUrl must be under the current domain` ? Если есть — перепроверьте `channelUrl`. Иначе стоит проверить параметры приложения в панели управления на сайте.

### 3. Авторизация
Изменим предыдущий код, а именно: создадим прототип функции `auth(data)` и укажем ее в `SE.init` как функцию, вызываемую при успешной инициализации:
```
function auth(data) {}
...
SE.init({
    ...
    complete: auth
});
```
Теперь самое время писать авторизацию. Здесь тоже все просто:
```
SE.authenticate({
    success: function(data) { alert('Я получил доступ!'); }, // Приложение авторизовало пользователя
    error: function(data) {  alert('Я не получил доступ :('); }, // Приложение не авторизовало пользователя
}); 
```
Этот код я поместил в тело `auth(data)`. Теперь, при первой загрузке страницы должно появиться всплывающее окно. В нем вам будут предложены две кнопки — на разрешение и запрет доступа для приложения:

![](http://habrastorage.org/getpro/habr/post_images/f13/448/709/f13448709920a84c0913e44dbffdfd89.png)

Однако пока мы просто авторизируемся и ничего не делаем с полученными данными. А ведь в случае успеха нам будут отданы клиентские токены для дальнейшей работы. Давайте сохраним их где-нибудь. Например так — создадим некоторую глобальную переменную, которая будет хранить эти самые токены:
```
var tokens = null;
...
SE.authenticate({
    success: function(data) { alert('Я получил доступ!'); tokens = data; }, 
    ...
...
```
Теперь, когда мы умеем авторизировать пользователя, можно перейти к непосредственной работе с API.

### 4. Получение данных профиля
API возвращает нам ответ на запросы в формате JSON. Это значит, что данные, загруженные посредством JS, легко будут переведены из строкового формата в некоторую структуру. Запросы будут кросс-доменные, и потому я воспользуюсь jQuery.
Данные о пользователе будем получать вот здесь:
```
https://api.stackexchange.com/2.2/me?site=[название сайта]
```
Некоторые данные о пользователе (например рейтинг или значки) зависят от того, на каком сайте сети Stack Exchange запрашивается профиль. Данные, полученные с StackOverflow и с MathOverflow, могут различаться. Название сайта можно посмотреть в самой системе. Окей. Если просто запросить данные по ссылке, мы получим что-то вроде такого json'a:
```
{"error_id":401,"error_message":"This method requires an access_token","error_name":"access_token_required"}
```
Все возможные ошибки можно глянуть [тут](http://api.stackexchange.com/docs/error-handling)

А ведь же нам нужно указать токены доступа! Для этого к нашему адресу выше мы прицепим эти самые токены:
`https://api.stackexchange.com/2.2/me?site=[название сайта]&key=[ключ приложения key]&access_token=[tokens.accessToken]&callback=[method]`

Здесь `tokens.accessToken` как раз ранее полученный в процессе авторизации ключ пользователя, а method — колбэк, который будет вызван в конце загрузки. Ответ будет примерно таким:
```
{
    "items": [
        {
            "badge_counts": {
                "bronze": 5,
                "silver": 0,
                "gold": 0
            },
            "account_id": 2760756,
            "is_employee": false,
            "last_modified_date": 1396214504,
            "last_access_date": 1396268249,
            "reputation_change_year": 62,
            "reputation_change_quarter": 62,
            "reputation_change_month": 62,
            "reputation_change_week": 55,
            "reputation_change_day": 10,
            "reputation": 63,
            "creation_date": 1368447422,
            "user_type": "registered",
            "user_id": 2377708,
            "age": 18,
            "location": "Belarus",
            "link": "http://stackoverflow.com/users/2377708/alex-saskevich",
            "display_name": "Alex Saskevich",
            "profile_image": "http://i.stack.imgur.com/0Vz5q.jpg?s=128&g=1"
        }
    ],
    "has_more": false,
    "quota_max": 10000,
    "quota_remaining": 9905
}
```
С такими данными визуально сложно работать, и потому я создам callback функцию, которая будет рендерить полученную информацию в небольшую табличку: 
```
function renderProfileData(data)
{
    var items = data.items[0];
    $("#reputation").text(items.reputation);
    $("#login").text(items.display_name);
    $("#bronze_badges").text(items.badge_counts.bronze);
    $("#silver_badges").text(items.badge_counts.silver);
    $("#gold_badges").text(items.badge_counts.gold);
    $("#profile_image").attr("src", items.profile_image);
}
```
Саму информацию я получал через GET запросы примерно так: `$.get("https://api.stackexchange.com/2.2/me?site=stackoverflow&key=YoUrAwEsOmEKey&access_token=YoUrSecretToKEn&callback=profile");`

В итоге у меня получился такой результат:

![](http://habrastorage.org/getpro/habr/post_images/687/aa6/b74/687aa6b74036bf07e26f55bde783b56e.png)

### 5. Последние уведомления
Для того, чтобы наше приложение могло получать информацию о непрочитанных сообщениях, уведомлениях, ему нужно предоставить определенные права доступа. В случае с уведомлениями это `read_inbox`. Ставим его в блоке авторизации приложения:
```
SE.authenticate({
    scope: ['read_inbox'],
    ...
});
```
Данные об уведомлениях мы получим вот тут: `https://api.stackexchange.com//2.2/notifications?pagesize=[число последних уведомлений]&key=[ключ приложения]&access_token=[токен доступа пользователя]&callback=[callback-метод]`

Мы можем не указывать число последних уведомлений, но тогда мы получим сразу все уведомления. Запрос выполняется аналогично предыдущему, ответ будет примерно таким:
```
{
    "items": [
            "site": {
                "styling": {
                    "tag_background_color": "#FFF",
                    "tag_foreground_color": "#000",
                    "link_color": "#0077CC"
                },
                "related_sites": [
                    {
                        "relation": "meta",
                        "api_site_parameter": "meta.reverseengineering",
                        "site_url": "http://meta.reverseengineering.stackexchange.com",
                        "name": "Reverse Engineering Meta Stack Exchange"
                    },
                    {
                        "relation": "chat",
                        "site_url": "http://chat.stackexchange.com",
                        "name": "Chat Stack Exchange"
                    }
                ],
                "open_beta_date": 1364774400,
                "closed_beta_date": 1363651200,
                "site_state": "open_beta",
                "twitter_account": "StackReverseEng",
                "favicon_url": "http://cdn.sstatic.net/reverseengineering/img/favicon.ico",
                "icon_url": "http://cdn.sstatic.net/reverseengineering/img/apple-touch-icon.png",
                "audience": "researchers and developers who explore the principles of a system through analysis of its structure, function, and operation",
                "site_url": "http://reverseengineering.stackexchange.com",
                "api_site_parameter": "reverseengineering",
                "logo_url": "http://cdn.sstatic.net/reverseengineering/img/logo.png",
                "name": "Reverse Engineering",
                "site_type": "main_site"
            },
            "is_unread": false,
            "creation_date": 1396197666,
            "notification_type": "badge_earned",
            "body": "You've earned the "Autobiographer" badge."
        }
    ],
    "has_more": true,
    "quota_max": 10000,
    "quota_remaining": 9994
}
```
Как видим, в данном случае система сразу отдает нам массив уведомлений. И с ними достаточно просто работать:
```
function renderNotifications(data)
{
    var items = data.items;
    var html = "<center>Последние уведомления</center><br/>";
    for (var i = 0; i < items.length; i ++)
    {
        var site = items[i].site.name;
        var icon = items[i].site.icon_url;
        var body = items[i].body;
        var str = "<div class = 'item'><img src = '" + icon + "' height = '14px' /> " + site + ": " + body + "</div>";
        html += str;
    }
    $("#notifications").html(html);
}
```
В итоге страничка моего маленького приложения приобрела вот такой окончательный вид:

![](http://habrastorage.org/getpro/habr/post_images/fff/f2a/096/ffff2a096c4e34e9d927f65f026284db.png)

### 6. И что теперь?
API сети Stack Exchange позволяет нам не только получать уведомления или данные профиля. Мы также можем получить вопросы, комментарии, теги, причем мы сможем также взаимодействовать с ними (добавлять вопросы, редактировать свои комментарии). Мы можем работать с профилем пользователя, событиями и статистикой.

Кстати о статистике — в панели управления приложением можно просмотреть статистику авторизаций и запросов в виде графиков. И да, стоит иметь ввиду, что число запросов, которое может сделать приложение к отдельной функции, ограничено десятью тысячами. Оставшееся число запросов пересылается нам вместе с JSON ответом в поле `«quota_remaining»`.

В документации также можно будет поэкспериментировать с запросами. Для этого StackExchange предлагает консоль, которую можно найти под описанием каждой функции.

### Cсылки
* [Документация](http://api.stackexchange.com/docs)
* [Панель управления приложениями](http://stackapps.com/apps/oauth)
* [Приложение в действии](http://jsfiddle.net/asaskevich/6F5kd/embedded/result/)
* [Содержание](README.md)

###### Маленькая заметка
В оргиниале данная запись была опубликована [тут](http://habrahabr.ru/post/217753/)
