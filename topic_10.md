Python: Flask и GitHub
======

Сегодня я решил немного поэкспериментировать с Python. Работать с декстоп-приложениями не так весело, потому я решил испытать какой-нибудь web-фреймворк. Пока размышлял, что выбрать, пришел к выводу, что для начала стоит попробовать Flask - такой вот микрофреймворк. А вместе с ним испытать плагин Flask-Github, предоставляющий авторизацию, а после позволяющий выполнять запросы к API.

#### Подготовка к работе

Сначала загрузим все необходимое:
```sh
pip install Flask
pip install GitHub-Flask
```
Библиотеки работают на Python 3, потому мне не пришлось переключаться в IDE на Python 2.7. Следующим этапом нужно создать приложение в профиле GitHub, а оттуда забрать `Client ID` и `Client Secret`. 

#### Каркас приложения

Теперь переходим непосредственно к коду. Инициализируем приложение, указываем маршруты:
```python
from flask import Flask

app = Flask(__name__)

@app.route('/login')
def login():
    ""
    
@app.route('/info')
def info():
    ""
    
@app.route('/')
def index():
    ""

if __name__ == "__main__":
    app.run()
```
В корне (`'/'`) сайта мы будем предлагать ссылку на авторизацию (`'/login'`), либо ссылку на просмотр информации о профиле (`'/info'`). Мы используем декораторы, чтобы связать функцию и соответствующий маршрут.

#### Настройка 

Далее нам нужно указать параметры приложения, полученые в профиле на GitHub:
```python
app.config["GITHUB_CLIENT_ID"] = 'XXX'
app.config["GITHUB_CLIENT_SECRET"] = 'YYY'
app.config["GITHUB_CALLBACK_URL"] = 'http://localhost:5000/callback'
```
Колбэк не забываем указать в профиле приложения. После всего нам нужно заняться модулем GitHub-Flask.

#### GitHub-Flask

После того, как мы указали в конфигурации Flask-приложения необходимые параметры, можно создать отдельное GitHub-приложение:
```python
app = Flask(__name__)
...
github = GitHub(app)
```
В маршруте `/login` вызываем "авторизатор":
```python
@app.route('/login')
def login():
    return github.authorize()
```
Теперь конфигурируем колбэк:
```python
@app.route('/callback')
@github.authorized_handler
def authorized(oauth_token):
    next_url = request.args.get('next') or url_for('index')
    # Если токен не получен:
    if oauth_token is None:
        print("Auth Error")
        return redirect(next_url)
    # А если получен, то его нужно сохранить
    ...
    return redirect(next_url)
```
В примерах, которые я разбирал, токен доступа для отдельного пользователя сохранялся в базу данных. Однако, так как я просто экспериментирую, мне достаточно сохранить токен одного единственного пользователя. На одного пользователя не жалко и глобальной переменной, хоть это и не очень красиво:
```python
token = None
...
@app.route('/callback')
@github.authorized_handler
def authorized(oauth_token):
    ...
    global token
    token = oauth_token
    return redirect(next_url)
```
Если мы собираемся выполнять запросы к GitHub API, нам нужно реализовать функцию, которая будет получать токен доступа, который будет использовать при обмене с сервером:
```python
@github.access_token_getter
def token_getter():
    if token is not None:
        return token
```
Ну и напоследок, сделаем запрос на получение информации об авторизированном пользователе:
```python
@app.route('/info')
def info():
    result = github.get("user")
    return str(result)
```
Вот и всё.

#### Заключение

Собственно все оказалось весьма просто и легко. Единственное, что немного разочаровало - запрос на авторизацию или запрос к API выполнялись весьма долго (несколько секунд). Может на реальном хосте все будет работать гораздо быстрее. С самой библиотекой в принципе разобрался, однако изучению несколько препятствовало наличие весьма малого количества знаний о Python. В данном случае знание базового синтаксиса недостаточно, необходимо иметь понятие об устройстве многих build-in библиотек.

#### Ссылки
* [Flask.py](http://flask.pocoo.org/)
* [GitHub-Flask](http://github-flask.readthedocs.org/en/latest/)
* [GitHub API](http://developer.github.com/v3/)
* [Содержание](readme.md)
