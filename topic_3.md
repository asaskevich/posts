Go: Блокнот с графическим интерфейсом на языке Go
======
Несмотря на то, что язык Go существует уже не один год, информация о том, как создавать приложения с графическим интерфейсом на этом языке, практически отсутствует. Возможно это вызвано тем, что среди официальных библиотек до сих пор нет библиотеки для работы с GUI. Однако это не значит, что мы не можем создать приложение с пользовательским интерфейсом: существуют библиотеки, предоставляющие такую возможность. Приведу их список. Но есть еще несколько библиотек, не указанных в этом списке. Среди них — Walk, название которого расшифровывается как «Windows Application Library Kit». С его помощью я попробую создать небольшое приложение с пользовательским интерфейсом.

Постановка задачи

Предположим, что я хочу создать небольшой блокнот, позволяющий мне сохранять введенный текст в файл, загружать этот текст обратно, копировать текст в буфер обмена и извлекать из него же. 

Установка библиотеки Walk

Walk работает только с Go версии 1.1.x и выше. На первом этапе нужно загрузить и установить библиотеку с GitHub. Запускаем CMD и выполняем команду :
go get github.com/lxn/walk

Если Go ругается и выдает что-то типа:
go: missing Git command ...
Нужно посетить git-scm.com/downloads и установить Git. Если после установки его ошибка не исчезла, нужно убедиться, что в %PATH% прописан путь до Git.

Создание пустой формы

Создадим исходный файл gopad.go с таким содержимым:
package main

import (
    . "github.com/lxn/walk/declarative"
)

func main() {
    MainWindow{
        Title:   "GoPad", // Ставим заголовок формы
        MinSize: Size{600, 400}, // И ее минимальные размеры
    }.Run()
}

Теперь скомпилируем наш исходник. Создадим bat-скрипт build.bat с таким содержимым:
go build -ldflags="-H windowsgui"
pause

Мы могли бы не прописывать ключ ldflags, но тогда бы форма запускалась вместе с консолью на заднем плане. Запустим наш скрипт и убедимся, что все скомпилировалось корректно. Но запускать полученный exe-файл пока что рано. Нужно создать еще один файл — манифест. Создадим файл gopad.exe.manifest (вместо gopad.exe должно быть название полученного бинарника) с таким содержимым:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
    <assemblyIdentity version="1.0.0.0" processorArchitecture="*" name="SomeName" type="win32"/>
    <dependency>
        <dependentAssembly>
            <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls" version="6.0.0.0" processorArchitecture="*" publicKeyToken="6595b64144ccf1df" language="*"/>
        </dependentAssembly>
    </dependency>
</assembly>

Манифест обязательно должен присутствовать. Без него бинарник не запустится!
Когда все сделано, пора запустить нашу пустую форму, чтобы убедиться, что все работает. У меня полученная программа выглядит так:


Добавляем необходимые кнопки и поле ввода

Для начала добавим поле ввода. Добавим в import еще одну библиотеку:
import (
    "github.com/lxn/walk"
    . "github.com/lxn/walk/declarative"
)

Потом создадим поле ввода, переменную-указатель и привяжем его к полю:
func main() {
    var edit *walk.TextEdit
    MainWindow{
        Title:   "GoPad",
        MinSize: Size{600, 400},
        Layout:  VBox{},
        Children: []Widget{
            TextEdit{AssignTo: &edit},
        },
    }.Run()
}

Запустим и убедимся, что появилось поле ввода, которое отображается на всю форму.
Таким же образом пропишем в MainWindow четыре кнопки: «Copy», «Paste», «Load», «Save»:
MainWindow{
     Title:   "GoPad",
     MinSize: Size{600, 400},
     Layout:  VBox{},
     Children: []Widget{
            TextEdit{AssignTo: &edit},
            HSplitter{
        	MinSize: Size{600, 30},
                Children: []Widget{
                	PushButton{
            		    Text: "Copy",
            		},
            		PushButton{
                	    Text: "Paste",
            		},
            		PushButton{
                	    Text: "Load",
            		},
            		PushButton{
                	    Text: "Save",
           		},
                },
           },
      },
}.Run()

В результате получаем вот такую форму:

Когда все элементы расположились на ней, пора добавить немного функционала.

Работа с буфером

Чтобы добавить обработчик нажатия на кнопку, достаточно прописать необходимую функцию, которая будет запускаться при нажатии кнопки, в свойствах самой же кнопки:
PushButton{
      Text: "Save",
       OnClicked: func() {
              // ...
       },
},

Мы можем либо создать анонимную функцию, либо прописать уже существующую.

Walk позволяет нам работать с буфером обмена напрямую через данные функции:
walk.Clipboard().SetText("text"); // Вставляем текст в буфер обмена
walk.Clipboard().Text(); // Извлекаем его

Чтобы получить или заменить текст в поле ввода, используем такие функции:
edit.Text(); // Чтобы получить текст из поля
edit.SetText("text"); // Чтобы его установить в поле

Теперь содержимое наших кнопок станет таким:
PushButton{
        Text: "Copy",
        OnClicked: func() {
                 walk.Clipboard().SetText(edit.Text());
        },
},
PushButton{
       Text: "Paste",
       OnClicked: func() {
                  if text, err := walk.Clipboard().Text(); err == nil {
                           edit.SetText(text)
                  }
       },
},


Сохранение в файл и загрузка из него

Давайте вынесем загрузку и сохранение в файл в отдельные функции. Для этого сделаем переменную edit глобальной. Создадим функцию readFromFile, подгружающую в поле текст из текстового файла file.txt (не забываем, что в import нужно добавить "io/ioutil"):
var edit *walk.TextEdit

func readFromFile() {
    contents,_ := ioutil.ReadFile("file.txt")
    edit.SetText(string(contents))
}

Аналогично пропишем сохранение в файл:
func saveToFile() {
    ioutil.WriteFile("file.txt", []byte(edit.Text()), 0x777)
}

Теперь нам нужно привязать функции к соответствующим кнопкам:
PushButton{
        Text: "Load",
        OnClicked: readFromFile,
},
PushButton{
        Text: "Save",
        OnClicked: saveToFile,
},

P.S. Содержимое текстового файла программа читает и пишет в юникоде.

Итоги

Мы создали небольшой блокнот с графическим интерфейсом на языке Go с использованием библиотеки Walk. При желании функционал можно расширить: добавить выбор файла, контекстное меню, работу с вкладками и шрифтами и многое другое. Walk активно развивается и предоставляет доступ к большому числу компонент WinUI. Тут находится много примеров с использованием библиотеки. 

Исходные коды файлов:

gopad.go
gopad.exe.manifest
build.bat

Ссылки

GUIs and Widget Toolkits
Walk
Walk Examples
