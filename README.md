# Игровая платформа SuzenEscape

Призвана для обучения в нестандартной форме командам Linux.
В готовом и развернутом виде поиграть можно тут: [escape.myctf.ru](http://escape.myctf.ru)

Этот репозиторий предназначен именно для разработки! Если вы совсем-совсем новичок в Linux, пробуйте сначала поиграть,
а потом уже заняться разбором платформы и созданию новых заданий.

## Структура

1) **ansible** предназначена для выкатки платформы на некоторый игровой сервер
2) **chain** содержит данные для сборки уровней
3) **suzen_website** содержит веб интерфейс для игры (жюрейная система не является открытым продуктом и не входит в этот репозиторий)

## БЫСТРЫЙ СТАРТ

### Требования к системе

* Ваша операционная система должна поддерживать 64-разрядные виртуальные машины.
* Теоретически платформу можн поднять на любой ОС, которая поддерживает `VirtualBox` и `Vagrant`, но проверялось только на `Debian` и `Ubuntu`

### Порядок действий

* Установить следующий набор ПО:
  * [vagrant](https://www.vagrantup.com/downloads.html)
  * [virtualbox](https://www.virtualbox.org/wiki/Linux_Downloads)
  * [rsync](https://rsync.samba.org/download.html) (или apt install rsync)
  * [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* Выполнить поднятие виртуальной машины

  ```shellsession
  vagrant up
  ```

* В случае получения ошибки выполнить команду:

  ```shellsession
  vagrant provision
  ```

* Платформа развернута и готова. Задания доступы через `ssh`, где `XX` - номер уровня

  ```shellsession
  ssh -p 2222 suzenXX@127.0.0.1
  ```

* Доступ к веб интерфейсу:

  ```text
  http://127.0.0.1:8080
  ```

## Используемые роли

Обратите внимание, что некоторые роли можно контролировать, например включать и отключать. Это существенно ускоряет разворачивание проекта при отладке. См. "Используемые переменные".

| Роль                 | Описание                                                                         |
| -------------------- | ---------------------------------------------------------------------------------|
| `install_docker`     | Устанавливает docker engine на указанный сервер                                  |
| `registry`           | Разворачивает локальный Docker image registry                                    |
| `build_containers`   | Собирает контейнеры с задачами и выгружает их в Docker image registry            |
| `deploy_web`         | Собирает контейнер с веб интерфейсом и разворачивает его на указанном сервере    |
| `deploy_tasks`       | Разворачивает задания на указанном сервере                                       |

## Сборка контейнеров

Сборку осуществляет корневой скрипт `build.py`, который принимает в качестве аргумента номер собираемого уровня. Аргумент может быть ключевым словом `all` — сборка всех уровней, либо перечисление номеров через пробел.

* Выполнить сборку всех уровней:

  ```shellsession
  ./build.py all
  ```

* Выполнить сборку определенных уровней:

  ```shellsession
  ./build.py 1 3 7
  ```

* Сборка веб-интерфейса:

  ```shellsession
  docker build -t 127.0.0.1:5000/suzenescape/web . && docker push 127.0.0.1:5000/suzenescape/web
  ```

## Registry и системные контейнеры

В случае если вы хотите разворачивать платформу с использованием стороннего docker registry, укажите его в
`ansible/values.yaml` в переменной `docker_registry`.

Также в рамках перехода на единый сборщик, существует еще системный контейнер: `bykva/busybinaries`,
он собирается в директории `busybox-custom`. Я разместил его на Docker Hub для уменьшения количества шагов в быстром старте.

## Используемые переменные

Все переменные проекта собраны в файле `ansible/values.yaml`

Конфигурационные параметры сборки платформы:

| Параметр                                           | Описание                                                                                     | Значение по-умолчанию                                   |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `deploy_registry`                                  | Контролирует сборку и установку контейнера Docker image registry                             | `true`                                                  |
| `registry_proto`                                   | Global Docker image registry protocol                                                        | `http`                                                  |
| `registry_url`                                     | Global Docker image registry url                                                             | `127.0.0.1:5000`                                        |
| `vagrant`                                          | Указатель среды в которой происходит работа ansible                                          | `vagrant`                                               |
| `deploy_web`                                       | Контролирует сборку и установку контейнера с веб интерфейсом                                 | `true`                                                  |
| `build_containers`                                 | Контролирует сборку и установку контейнеров с задачами                                       | `true`                                                  |
| `tasks_to_build`                                   | Устанавливает номер(а) задания для сборки. Строка. Может принимать значения: `"1 2 3 ... N"` | `all`                                                   |
| `install_docker`                                   | Контролирует установку Docker engine на указанный сервер                                     | `true`                                                  |
| `is_online`                                        | Используется для сборки в оффлайн режиме. Применимо ТОЛЬКО после сборки в онлайн режиме.     | `yes`                                                   |

Переменные специфичные для задач:

| Параметр                                           | Описание                                                                                     | Пример                                                  |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `name`                                             | Порядковый номер уровня. Используется для ssh авторизации                                    | `suzen5`                                                |
| `password`                                         | Пароль, используемый при ssh-авторизации для доступа к уровню.                               | `suzen5`  или   `dmVlNFdvaE42ZWVoMFpvN3dhcGgK`          |
| `sault`                                            | Строка данных, для защиты пароля от перебора. Принимает значение `rand([:alnum:]{8})`        | `Yy9aP4bg`                                              |
| `config`                                           | Список исполняемых файлов, которые будут доступны пользователю на уровне                     | `cd\|ls\|mknod\|sh`                                        |
| `rohome`                                           | Устанавливает доступность для записи домашней директории пользователя                        | `true`                                                     |
| `chain`                                            | Принадлежность задачи цепочке                                                                | `2`                                                  |
| `servers`                                          | Словарь, описывающий серверную часть задания при ее наличии                                  | `[]`                                                    |
| `flag`                                             | Устанавливаемый при сборке флаг для конкретного уровня                                       | `dmVlNFdvaE42ZWVoMFpvN3dhcGgK`                          |

## Виды задач

* Задачи с флагом внутри: флаг дается за то что игрок догадался куда смотреть или выполнил набор действий,
  который привел его к флагу.
* Задачи с внешней проверкой: используется `cronjob`, которые прописываются в `ansible\templates`.
  (Также нужно прописать в `site.yaml`). Согласно cron будет выполняться достижение некоторого заданного состояния,
  за который игроку обещан флаг. Состояние и проверка выдумываются автором такого задания.
* Задачи с подключением к стороннему контейнеру. см таски `5` и `9`.
