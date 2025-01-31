---
title: 'Распаковка Docker, часть 1'
date: 2021-02-08
author: igor-korovchenko
editors:
    - vadim-makeev
tags:
    - article
    - js
hero:
    src: images/cover.png
    cover: true
featured: true
---

Труд наш в основном творческий, интеллектуальный, и мы посвящаем довольно много времени размышлениям о том, что и как делаем. В условиях быстро развивающейся платформы веб-разработчикам приходится принимать сложные, часто компромиссные, решения. Хороший программист всегда стремится к совершенству, и такое положение вещей раздражает. Однако мы можем существенно облегчить свою участь с помощью Docker, который в худшем случае сократит время работы перед релизом, а в лучшем — добавит оснований нам быть счастливыми.

Есть и другая сторона. Какое впечатление создает ваш продукт у заказчика и его клиентов? В Docker вы создаете упаковку, создаете, если хотите, свой «дизайнерский язык», уникальный первичный пользовательский опыт. Отношение к вам часто формируется через него. Представьте продукт в физическом, а не в виртуальном мире. Пользователь снимает пленку, открывает крышку коробки, внимательно изучает макулатуру, достает провода и зарядку, вдыхает неповторимый аромат новой техники… Скачивая программу, пользователь ожидает нечто подобное. Восхищается ли он простотой, удобством и лаконичностью?

Материал, в начале которого вы находитесь, накладывает определенные требования и к упаковке самого себя. Из соображений удобства весь текст разделен на три части:

1. **Первая часть** описывает терминологию и основные инструменты Docker — _вы находитесь здесь._
2. [Вторая часть](https://web-standards.ru/articles/docker-unboxing-2/) посвящена практическим примерам, которые могут быть полезны веб-разработчикам.
3. [Третья часть](https://web-standards.ru/articles/docker-unboxing-3/) содержит интересности для организации процесса разработки.

_Третья часть выйдет 10 февраля — прим. редактора._

## Навигация

- [Технология виртуализации](#section-2)
- [Основные понятия Docker](#section-3)
    - [Docker container (контейнер)](#section-4)
    - [Docker image (образ)](#section-5)
    - [Union File Systems](#section-6)
- [Cистема команд Linux](#section-7)

## Технология виртуализации

В 7 главе [знаменитой книги Эндрю Таненбаума](https://www.piter.com/collection/A20865/product/sovremennye-operatsionnye-sistemy-4-e-izd), ставшей хрестоматией для современного поколения инженеров в области Computer Science, понятие виртуализации рассматривается достаточно подробно. Вся концепция сводится к существованию менеджера виртуальных машин (Virtual Machine Manager), или, по-другому, гипервизора (hypervisor), который позволяет запускать ОС так, как будто она существует сама по себе и запущена на отдельном компьютере. Гипервизоры бывают двух типов: функционирующие на уровне «железа» и программные, работающие уже как слой в базовой ОС (host OS).

Применение виртуализации связано как с развитием самой вычислительной техники и программ, так и с построением сложных инфраструктурных решений для обеспечения работы компаний разного уровня. Выделенный сервер у хостинг-провайдера, к примеру, будет запускаться на «железном» гипервизоре, а у вас, скорее всего — на программном типа VMWare или VirtualBox.

Эти технологии не новые и существуют уже более 50 лет. Однако после небольшого периода забвения они получили в последнее десятилетие новую жизнь. Почему мы говорим про виртуализацию в случае Docker?

На ОС, которые не относятся к семейству Linux, служба Docker запускается как отдельная, хоть и очень быстрая, виртуальная машина. Потери ресурсов ОС, хоть и небольших, не избежать. Об этом лучше знать заранее. Подробнее об архитектуре Docker можно почитать в [Anatomy of Docker](https://medium.com/sysf/getting-started-with-docker-1-b4dc83e64389), мы же рассмотрим лишь основные моменты.

## Основные понятия Docker

В основе Docker лежит понятие контейнера, которое появилось в Linux довольно давно. [Linux containers](https://linuxcontainers.org/) (LXC) в основном использовались и продолжают использоваться для разделения прав пользователей и создания изолированных окружений для разработки, сборки и запуска программ. Простого разграничения прав пользователей не хватало, поскольку требовались наборы разного софта с иногда противоположными значениями в настройках для того или иного случая. Все это существенно приводило к нестабильности базовой ОС, дырам в безопасности и багам в программах.

Технология LXC позволяла использовать единое ядро ОС и поверх запускать изолированные экземпляры набора служб со своими настройками, пользователями, подключенными (доступными) внешними устройствами (в понимании архитектуры фон Неймана), с собственным пространством процессов и собственным сетевым стеком. Понятное дело, что все это вынесли в отдельную абстракцию, которая стала называться контейнером.

LXC сейчас применяются, например, у хостинг-провайдеров, позволяя клиентам арендовать часть ресурсов какого-то сервера для размещения своего сайта (услуга виртуального хостинга). При этом на одной физической или виртуальной машине могут работать одновременно очень много контейнеров.

Какие шаги нужно выполнить для того, чтобы создать и запустить контейнер (независимое окружение, «песочницу») с точки зрения администратора? Вот примерный список действий:

1. Сгенерировать LXC;
2. Выделить специальную файловую систему;
3. Примонтировать новый слой ОС;
4. Создать виртуальный интерфейс для работы с сетью;
5. Настроить таблицы маршрутизации;
6. Настроить туннелирование сетевых пакетов через протокол NAT;
7. Запустить нужный процесс и перенаправить вывод на вывод родительской ОС.

Реализация каждого из семи шагов занимает довольно много времени. Вы же понимаете, что с первого раза редко удаётся что-то хорошо настроить для большого количества пользователей? Словом, настройка и поддержание такой системы в рабочем состоянии — не простая задача.

В день рождения Docker, 21 марта 2013 года, Соломон Хайкс (Solomon Hykes) довольно изящно [представил](https://www.youtube.com/watch?v=wW9CAH9nSLs) совершенно новое решение, достоинства которого легко продемонстрировать в терминале. Он загрузил из Интернета вариант сборки контейнера, собрал, запустил внутри контейнера приложение и получил его результаты единственной командой:

```bash
docker run -dp 80:80 docker/getting-started
```

Естественно, перед тем, как использовать команду docker, необходимо установить кое-какие специальные службы (пара-тройка команд, которые выполняются один раз при настройке ОС сервера). А сейчас уже существуют готовые сборки серверов с Docker, когда ничего вообще не надо настраивать. К тому же, Docker теперь отлично работает на всех популярных и на некоторых не очень популярных ОС, обеспечивая полную совместимость. Программисты могут свободно разрабатывать программы на базе своей ОС и быть уверенными, что все запустится с первого раза на любом компьютере, на любом сервере, где установлены службы Docker.

Таким образом, написав программу один раз и поместив ее внутрь контейнера, можно запускать ее на большинстве ОС. Я описал лишь основной сценарий использования Docker. Для программиста существует еще несколько сценариев, которые мы разберем по ходу. Однако сейчас немного остановимся на терминологии.

### Docker container (контейнер)

В статье [LXC vs Docker: Why Docker is Better](https://www.upguard.com/blog/docker-vs-lxc) приводится сравнение возможностей технологий Docker и LXC, которое оказывается явно не в пользу последней. В LXC под контейнером подразумевается изолированное окружение пользователя, которое формируется за счет выставления прав (ограничения доступа) для пользователя. Используется установленная на компьютере ОС, но для программ и пользователей контейнер оказывается полностью изолированным. Складывается ощущение, что он совершенно независим, но это не так. Например, LXC-контейнер не может использовать ОС, отличную от базовой. Технология LXC решает задачу изоляции окружения, для которой и была создана, но иногда нужно немного больше.

Под контейнером в Docker (Docker container) подразумевается эмуляция ОС на основе специальной службы Docker daemon, которая устанавливается на родительскую ОС и является сервером. Она использует не только механизм LXC, но и включает cgroups, прослойку для коммуникации со службами ОС и ядро Linux.

Docker daemon осуществляет управление объектами (Docker objects) и позволяет установить и запустить на Linux, macOS или Windows виртуальную копию окружения большого списка разных дистрибутивов Linux. На Linux используется ядро базовой ОС. На macOS и Windows — ядро оптимизированной виртуальной машины с ОС Linux.

С точки зрения пользователя контейнер становится аналогом отдельной ОС, на которой установлено нужное окружение и запускаемое приложение. С точки зрения родительской ОС, запуская контейнер, вы запускаете просто отдельное приложение.

Получается, что ОС работает в обычном режиме, а пользователь получает все преимущества отдельной виртуальной машины и ОС контейнера. При этом ресурсы компьютера на обеспечение механизма работы Docker практически не тратятся и идут на обеспечение работы целевого приложения.

### Docker engine (движок)

[Движок Docker](https://docs.docker.com/get-started/overview/#docker-engine) состоит из Docker daemon, выступающего в роли сервера, прослойки REST API и клиента (Docker CLI), на котором осуществляется управление образами (Docker image) и томами (Docker volumes), управление жизненным циклом контейнеров, обеспечивается эмуляция локальной сети для связи запущенных контейнеров друг с другом.

### Docker image (образ)

Это образ (прототип) контейнера, который в конечном итоге будет запущен с помощью Docker CLI. Образ представляет собой конфигурационный файл, который содержит набор инструкций для сборки контейнера. Каждую новую инструкцию можно сравнить с новым слоем. Например, на первом слое описывается, какой должен использоваться дистрибутив ОС, на втором слое устанавливаются какие-нибудь специфические для контейнера службы или сервисы, а на последнем слое собирается или просто устанавливается целевое приложение (система управления базами данных (СУБД) / сайт / веб-сервис).

### Union File Systems

В контейнерах используется особая стековая файловая система (Union File Systems), в которой файлы и каталоги одного слоя представляют собой отдельную файловую систему (ветвь). Каждая новая ветвь может быть наложена на предыдущую, образуя единую файловую систему собранного контейнера. Содержимое каталогов, имеющих одинаковый путь внутри наложенных ветвей, рассматривается как единый объединенный каталог, что позволяет избежать необходимости создавать отдельные копии каждого слоя. За подробностями можно обратиться к статьям и видео:

- [A Beginner-Friendly Introduction to Containers, VMs and Docker](https://www.freecodecamp.org/news/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b/)
- [Изучаем Docker, часть 1: основы](https://habr.com/p/438796/)
- [Изучаем Docker, часть 2: термины и концепции](https://habr.com/p/439978/)
- [The future of Linux Containers](https://youtu.be/wW9CAH9nSLs)
- [Основы Docker. Большой практический выпуск](https://youtu.be/QF4ZF857m44)

### Система команд Linux

Итак, задача стоит перед нами исследовательская. Например, вы хотите попробовать разобраться с новой для вас системой команд конкретной версии ОС Linux или с настройкой какой-то определенной службы.

Чтобы воспользоваться Docker для этой цели, нам необходимо в первую очередь его установить. [На официальном сайте](https://docs.docker.com/get-docker/) найдите дистрибутив Docker Desktop для macOS и Windows или инструкции для установки Docker для вашей версии ОС Linux. В результате установки вы получите Docker engine, Docker daemon и Docker CLI, то есть все самое необходимое для использования Docker.

Следующий этап представляется намного более творческим. Для работы контейнера с «установленной» внутри него версией ОС Linux, необходимо его из чего-то собрать. Для этого используется базовый образ (Docker image). Необходимо найти такой образ, на который вы потом будете «наслаивать» специфическое окружение. Для выполнения поставленной задачи необходимо выбрать образ ОС Linux и получить доступ к терминалу.

Сначала проверим, что после установки и запуска все работает корректно:

```bash
docker --version

> Docker version 19.03.13, build 4484c46d9d
```

Командой выше можно протестировать, что Docker готов, может скачивать, устанавливать и запускать образы (Docker image). Образы должны где-то храниться, и Docker предлагает хранить их с помощью удаленной платформы [Docker Registry](https://docs.docker.com/registry/) или в общедоступном хранилище [Docker Hub](https://hub.docker.com/) (бесплатный общедоступный реестр образов).

Кроме простых образов существует понятие репозиториев [Docker repository](https://docs.docker.com/docker-hub/repos/), которые представляют собой набор образов с одинаковым именем, но с разными тегами (идентификаторами образов). Обычно с помощью тегов организуется хранение образов с разными версиями программного обеспечения и/или ОС.

Попробуем запустить образ. Для этого у команды Docker есть специальный тестовый образ, который можно скачать, установить и запустить следующим образом:

```bash
docker run hello-world

> Unable to find image 'hello-world:latest' locally
> latest: Pulling from library/hello-world
> 0e03bdcc26d7: Pull complete
> Digest: sha256:e7c70bb24b462baa86c102610182e3efcb12a04854e8c582838d92970a09f323
> Status: Downloaded newer image for hello-world:latest
>
> Hello from Docker!
```

Сначала Docker CLI попытался найти указанный образ hello-world среди уже скачанных и не нашел его («Unable to find … locally»). Затем он обратился к реестру Docker Hub, который установлен по умолчанию, нашел, скачал и установил последнюю версию: «latest: Pulling from…». В итоге мы получили вывод в терминале сообщения «Hello from Docker!»

Если сейчас посмотреть список запущенных контейнеров с помощью команды:

```bash
docker ps

> CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

Вы увидите, что список пуст (присутствуют только заголовки таблицы). Если запустить эту команду с определенным ключом `--all`, то мы получим список всех запущенных контейнеров и контейнеров, которые уже отработали, но не были удалены из списка (операция удаления производится вручную или при остановки службы Docker daemon):

```
docker ps --all

> CONTAINER ID  IMAGE        COMMAND   CREATED        STATUS                   PORTS  NAMES
> 93cb27f68163  hello-world  "/hello"  3 minutes ago  Exited (0) 3 minutes ago        happy_goldberg
```

Приведу список основных команд со ссылками на оригинальную документацию, которыми вы будете пользоваться большую часть времени при использовании Docker:

- [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
- [docker run](https://docs.docker.com/engine/reference/commandline/run/);
- [docker image](https://docs.docker.com/engine/reference/commandline/image/);
- [docker container](https://docs.docker.com/engine/reference/commandline/container/);
- [docker volume](https://docs.docker.com/storage/volumes/).

Вернемся к нашей задаче. Самый простой способ воплотить ее в жизнь, найти готовый образ желаемой ОС в реестре и запустить. Для этого переходим на страницу [Docker Hub](https://hub.docker.com) и ищем необходимый нам образ (пусть это будет [образ с CentOS 8](https://hub.docker.com/_/centos)). Вы наверняка увидите подсказку с командой для установки данного образа, после выполнения которой вам будет выведена следующая информация:

```bash
docker pull centos

> Using default tag: latest
> latest: Pulling from library/centos
> 3c72a8ed6814: Pull complete
> Digest: sha256:76d24f3ba3317fa945743bb3746fbaf3a0b752f10b10376960de01da70685fbd
> Status: Downloaded newer image for centos:latest
> docker.io/library/centos:latest
```

Любопытно, что образ чистой ОС занимает всего около 70 Мб, гораздо меньше, чем полноценная ОС Linux. Удивляться не надо, ведь в образе нет, например, ядра ОС.

В интерфейсе Docker Hub есть три вкладки с полезной информацией: Description, Reviews, Tags. Наличие вкладки Tags свидетельствует о том, что перед нами не образ, а репозиторий образов. На вкладке Reviews есть полезные комментарии от пользователей, на которые стоит обратить внимание при выборе образа или версии образа. Также вы можете заметить справа над строкой, которую мы с вами скопировали, селектор с выбором платформы (ОС и архитектура процессора).

Надо понимать, что под архитектурой процессора подразумевается архитектура процессора, для которой собран образ ОС. По умолчанию используется amd64 (синоним x86-64). В первой строке вывода мы также видим, что по умолчанию присваивается тег latest. Это означает, что версию ОС, которая будет установлена в контейнере нужно смотреть в реестре под этим тегом.

Информация о том, какие слои должны присутствовать в образе (другими словами — «какое окружение»), хранится в специальном файле Dockerfile. На главной странице репозитория ссылка [latest, centos8, 8](https://github.com/CentOS/sig-cloud-instance-images/blob/12a4f1c0d78e257ce3d33fe89092eee07e6574da/docker/Dockerfile) в списке тегов (Supported tags and respective Dockerfile links) ведет на содержимое файла Dockerfile с указанием четырех слоев контейнера, чуть позже мы рассмотрим эти конфигурационные файлы подробнее. Для используемого нами образа это всего три слоя (базовый слой, слой операционный системы, запуск терминала), которые устанавливаются командами:

```docker
FROM scratch
ADD centos-8-x86_64.tar.xz /
LABEL org.label-schema.schema-version="1.0" org.label-schema.name="CentOS Base Image" org.label-schema.vendor="CentOS" org.label-schema.license="GPLv2" org.label-schema.build-date="20200809"
CMD ["/bin/bash"]
```

Используется базовая ОС для Docker. С помощью команды `ADD` добавляются файловая система образа интересующей нас ОС, затем устанавливаются именованные константы для контейнера командой `LABEL` (важный шаг для управления контейнерами), а на последнем шаге запускается терминал командой `CMD`. Кажется, пора запустить контейнер.

Давайте посмотрим список загруженных образов с помощью команды:

```bash
docker image ls

> REPOSITORY   TAG     IMAGE ID      CREATED       SIZE
> centos       latest  0d120b6ccaa8  3 months ago  215MB
> hello-world  latest  bf756fb1ae65  11 months ago 13.3kB
```

Обратите внимание на то, что при распаковке размер образа увеличился почти в три раза (215MB против ~ 70MB). Запуск контейнера осуществляется командой:

```bash
docker run centos
```

Если вы выполните эту команду, то увидите, что на консоль ничего не вывелось. Почему? Вроде бы вы скачали правильный образ, вроде бы запустили… Помните, что вы запускаете какое-то приложение. После окончания работы этого приложения контейнер автоматически останавливается и выгружается из памяти. Так и получилось.

Как же быть, если нам нужно запустить терминал для работы с консолью контейнера? Необходимо выполнить команду со специальными ключами:

```bash
docker run -it --entrypoint bash centos
```

Ключ `--entrypoint` перезаписывает значение точки вхождения по умолчанию. Это приложение, которое запускается внутри контейнера после его сборки. Два ключа `-i` и `-t` служат для открытия стандартного потока ввода и использования одного терминала соответственно. С подробностями вы можете ознакомиться в [разделе документации](https://docs.docker.com/engine/reference/commandline/run/), относящейся к команде, [в статье на Хабре](https://habr.com/p/439978/), в Википедии: [Стандартные потоки](https://ru.wikipedia.org/wiki/Стандартные_потоки) и [TTY-абстракция](https://ru.wikipedia.org/wiki/TTY-абстракция).

Чтобы выйти из терминала bash, как обычно, необходимо выполнить команду `exit`. Это приведет к остановке контейнера, поскольку программа терминала будет завершена.

Взаимодействие с внешними хранилищами — папками базовой ОС, на которой запущен Docker, или с содержимым других контейнерами — обеспечивается путем использование томов [Docker volumes](https://docs.docker.com/storage/volumes/). Тома монтируются в папки контейнера, которые можно задавать с помощью конфигурационных файлов или напрямую в терминале установкой определенных флагов.

Тома могут быть как самостоятельными ресурсами, недоступными для базовой ОС (обычный тип), так и связанными с папками базовой ОС (тип bind). Если используется независимый том, его необходимо предварительно создать и подключить к контейнеру. Для работы с томами используются следующие команды:

- **Создание тома:** `docker volume create my_volume`
- **Список всех томов:** `docker volume ls`
- **Информация о томе:** `docker volume inspect my_volume`
- **Удаление тома:** `docker volume rm my_volume`

Когда вы создадите обычный том и просмотрите информацию о нем, вы увидите, в какой именно папке будет содержаться файл с содержимым тома в базовой ОС. Чтобы подключить том при запуске контейнера и иметь доступ к содержимому этого тома в определенной папке контейнера (например, /tmp/volume), достаточно выполнить команду `run` с параметром `--mount`:

```bash
docker run --mount 'source=my_volume,target=/tmp/volume' -it --entrypoint bash centos
```

Если к контейнеру необходимо подключить папку из базовой ОС, необходимо выполнить команду со следующими ключами:

```bash
docker run --mount 'type=bind,source=/tmp,target=/tmp/volume' -it --entrypoint bash centos
```

Связывать подобным образом папки базовой ОС оказывается довольно удобно для работы приложений внутри контейнера. Вы можете разобраться с использованием томов подробнее [в документации](https://docs.docker.com/storage/) или кратко [в руководстве по работе с данными](https://habr.com/p/441574/).

Имея доступ к терминалу ОС контейнера, вы можете изучить интересный для вас дистрибутив ОС Linux, не устанавливая его к себе на компьютер и не арендуя сервер. Например, теперь вы можете экспериментировать со специфичным пакетным менеджером или создать файлы настроек нужных вам служб. И все это после выполнения одной команды в терминале!

***

Кажется, теперь мы прошлись по всему самому необходимому и готовы попробовать использовать технологию на практике. [Поговорим об этом во второй части](https://web-standards.ru/articles/docker-unboxing-2/).
