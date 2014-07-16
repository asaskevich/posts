Для размещения своего web-приложения в облаке уже существует не мало различных сервисов и хостингов, однако лишь немногие поддерживают работу с Go. Среди них можно обратить внимание на следующие:
— Google App Engine
— Heroku
Некоторые другие сервисы также предлагают поддержку Go, однако на платной основе, что не всегда выгодно для разработчика, например, если он проводит различные эксперименты, изучая особенности языка. Выбрав такие критерии выбора, как простоту развертывания, скорость и удобство, я остановился на Heroku.

Для одного аккаунта Heroku предлагает до 5 приложений на бесплатной основе. Каждому приложению система выделяет 750 бесплатных часов работы в месяц, также следует учитывать, что после часа «простоя» приложение уходит в режим «сна» (Но оно автоматически будет «разбужено» при поступлении запроса к нему).

1. Регистрация в системе и авторизация

Если у вас нету профиля, создайте его, перейдя вот по этой ссылке. Далее нужно загрузить и установить Heroku Toolbelt. Установив, убедитесь, что у вас в консоли работает команда heroku. Если все работает, открываем терминал и вводим следующее:
$ heroku login
Enter your Heroku credentials.
Email: user@server.com
Password:
Could not find an existing public key.
Would you like to generate one? [Yn]
Generating new SSH public key.
Uploading ssh public key /Users/user/.ssh/id_rsa.pub


2. Создание приложения

Цель поста — показать, как развернуть приложение в облаке, потому я обойдусь простейшим «Hello, World», используя фреймворк martini:
package main

import "github.com/go-martini/martini"

func main() {
    m := martini.Classic()
    m.Get("/", func() string {
        return "Hello World"
    })
    m.Run()
}

Исходный код я разместил в файле $GOPATH/github.com/user/hello/server.go. 

3. Создание файла Procfile

Procfile нужен Heroku для того, чтобы знать, как запускать сервер. Разместим там одну маленькую строчку:
web: hello

Обратите внимание, что если ваш исходник расположен в папке, отличной от папки hello, то и содержимое будет несколько другим:
web: <название папки, в которой расположен код приложения> 


4. Создание локального репозитория

В папке $GOPATH/github.com/user/hello/ выполняем следующие команды:
$ git init
$ git add -A .
$ git commit -m "code"

В дальнейшем мы будем производить push из локального репозитория на репозиторий Heroku.

5. Godep — сохранение зависимостей

godep — специальный инструмент для управления зависимостями пакета. Он позволит сохранить информацию о пакетах, которые использует наш проект, и их исходный код.
Устанавливаем:
$ go get github.com/kr/godep

Переходим в нашу папку $GOPATH/github.com/user/hello/ и выполняем:
$ godep save

В итоге будет создана папка Godep, в которой вы найдете файл Godep.json со списком зависимостей, а также папку _workspace с исходными кодами сторонних пакетов.
Делаем коммит:
$ git add -A .
$ git commit -m "dependencies"


6. Создание приложения на Heroku и развертывание

Теперь начинается самое интересное. Если вы ушли из папки $GOPATH/github.com/user/hello/, то вернитесь. Теперь в терминале выполняем следующее:
$ heroku create -b https://github.com/kr/heroku-buildpack-go.git
Creating secure-beyond-6735... done, stack is cedar
BUILDPACK_URL=https://github.com/kr/heroku-buildpack-go.git
http://secure-beyond-6735.herokuapp.com/ | git@heroku.com:secure-beyond-6735.git
Git remote heroku added

Команда создаст наше приложение и, используя Go Heroku Buildpack, сохранит информацию о том, как его нужно собирать и развертывать.
Делаем push:
$ git push heroku master
Initializing repository, done.
Counting objects: 11, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (11/11), 1.29 KiB | 0 bytes/s, done.
Total 11 (delta 0), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> Go app detected
-----> Installing go1.3... done
-----> Running: godep go install -tags heroku ./...
-----> Discovering process types
       Procfile declares types -> web
 
-----> Compressing... done, 1.7MB
-----> Launching... done, v4
       http://secure-beyond-6735.herokuapp.com deployed to Heroku

To git@heroku.com:secure-beyond-6735.git
 * [new branch]      master -> master

Почти все, мы выполняем еще одну команду, Heroku запустит приложение, затем откроет браузер и перейдет по адресу работающего приложения:
$ heroku open
Opening secure-beyond-6735... done

Все, приложение запущено на Heroku. В будущем вам нужно будет только поправить зависимости (если начнете использовать новые библиотеки), сделать коммит и push. На мой взгляд весьма быстро, просто и удобно. Вот тут описан похожий способ, но на мой взгляд он немного сложнее. 

Все ссылки:
— Heroku Signup
— Heroku Toolbelt 
— Godep 
— Go Heroku Buildpack
