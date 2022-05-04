## Laboratory work VIII

Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [ ] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [ ] 2. Ознакомиться со ссылками учебного материала
- [ ] 3. Выполнить инструкцию учебного материала
- [ ] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Report

Клонируем репозиторий с прошлой лр
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```
Создаем файл для Docker
```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
Устанавливаем компиляторы и cmake
```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
Копируем все полученные файлы в новую директорию print и делаем ее активной (аналог cd)
```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
Выполняем сборку нашего приложения
```sh
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
Добавляем переменную окружения для работы программы demo/main.cpp
```sh
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```
Указываем каталог, изменения в котором будут сохранены
```sh
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```
Переходим в созданную на этапе сборки директорию
```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
Запускаем исполняемый файл приложения
```sh
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```
Собираем образ
```sh
$ docker build -t logger .
```
Команда для просмотра доступных образов
```sh
$ docker images
```
Вызов нашей программы из образа
(Флаг -i — это сокращение для --interactive. Благодаря этому флагу поток STDIN поддерживается в открытом состоянии даже если контейнер к STDIN не подключён. Флаг -t — это сокращение для --tty. Благодаря этому флагу выделяется псевдотерминал, который соединяет используемый терминал с потоками STDIN и STDOUT контейнера. Параметр -v задает каталог аналогично volume)
```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
#Вводим текст и жмем CTRL-D
```
Команда для просмотра состояния образа
```sh
$ docker inspect logger
```
Проверим, что программа вывела наш текст в лог
```sh
$ cat logs/log.txt
```
Создадим .yml файл для создания образа
```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
Пуш всех изменений
```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
