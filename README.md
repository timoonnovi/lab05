<div align="center">
<h1><a id="intro">Лабораторная работа №5</a><br></h1>
<a href="https://docs.github.com/en"><img src="https://img.shields.io/static/v1?logo=github&logoColor=fff&label=&message=Docs&color=36393f&style=flat" alt="GitHub Docs"></a>
<a href="https://daringfireball.net/projects/markdown"><img src="https://img.shields.io/static/v1?logo=markdown&logoColor=fff&label=&message=Markdown&color=36393f&style=flat" alt="Markdown"></a> 
<a href="https://symbl.cc/en/unicode-table"><img src="https://img.shields.io/static/v1?logo=unicode&logoColor=fff&label=&message=Unicode&color=36393f&style=flat" alt="Unicode"></a> 
<a href="https://shields.io"><img src="https://img.shields.io/static/v1?logo=shieldsdotio&logoColor=fff&label=&message=Shields&color=36393f&style=flat" alt="Shields"></a>
<a href="https://img.shields.io/badge/Risk_Analyze-2448a2"><img src="https://img.shields.io/badge/Course-Risk_Analysis-2448a2" alt= "RA"></a> <img src="https://img.shields.io/badge/AppSec-2448a2" alt= "RA"></a> <img src="https://img.shields.io/badge/Contributor-Пасько_И._Д.-8b9aff" alt="Contributor Badge"></a></div>

***

Данная лабораторная работа посвещена изучению Docker и как с ним работать. Эта лабораторная работа послужит подпоркой для старта в выявлении и определении уязвимостей на уровне сканирования контейнеров при сборке приложений. 

Для сдачи данной работы также будет требоваться ответить на дополнительыне вопросы по описанным темам.

***

## Структура репозитория лабораторной работы

```bash
lab05
├── client
│   ├── client.py
│   ├── Dockerfile
│   └── requirements.txt
├── docker-compose.yml
├── README.md
├── server
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
└── source
    ├── Dockerfile
    ├── hello.py
    ├── image.tar
    └── requirements.txt
```

***

## Материал

- **Контейнеризация**

Сборка приложения включает создание контейнерного образа, в котором упаковано приложение с конфигурациями, что бы приложение функционировало. `Docker` основан на использовании общих функций ядра `ОС Linux` (`cgroups`, `namespace`) для изоляции и управления ресурсами.

> **Образ** — это статический, неизменяемый шаблон, на базе которого создаются контейнера с ОС, приложением, зависимостями, библиотекакм и конфигурационными файлами. Нужен для создания воспроизводимой, неизменяемой среды выполнения приложений в контейнерах.

Для сборки образов используется `Dockerfile`, где прописаны версии зависимостей и инструкции, минимизирующие разрешения и атаки. Это инструкции, где описывается, как собрать образ. Впоследствии собирается контейнер.

> **Контейнер** — это изолированная среда выполнения приложения с необходимыми зависимостями, кодом, системными утилитами, библиотеками и настройками. Исползует не собственну гостеву ОС, а ядро хостовой ОС и имеет своё собственное файловое пространство, процессы и сеть.

После сборки образа формируется контейнер, которые являются изолированными средами выполнения для достижения цели переносимости, воспроизведения.

> **Контейнеризация** — это технология, позволяющая упаковать приложение вместе со всеми его зависимостями, библиотеками, настройками и средой выполнения в единый изолированный виртуальный контейнер. 

- **Namespaces**

Необходимы для организации изолированных рабочих пространств, - контейнеров. Когда мы запускаем контейнер, `Docker` создает набор пространств имен для данного контейнера, что создает изолированный уровень в своем простанстве имен и не имеет доступа к внешней системе. Пространство имен:

> - pid: для изоляции процесса
> - net: для управления сетевыми интерфейсами
> - ipc: для управления IPC ресурсами. (ICP: InterProccess Communication)
> - mnt: для управления точками монтирования
> - utc: для изолирования ядра и контроля генерации версий(UTC: Unix timesharing system)

- **Cgroups**

Необходимы для контрольных групп в изоляции, где предоставляется приложению только те ресурсы, которые указываем. Позволяют разделять ресурсы железа и устанавливать пределы, ограничения.

```bash
$ docker container run d \
        —e NGINX_HOST xxx.xxx \
        —p 8080:80 \
        –-v "$PWD/html" usr/share/nginx/html \
        —memory=50m \
        —cpus="2.5" \
        nginx
```

-  **Основные проблемы**

    - образ может содержать устаревшие или уязвимые версии библиотек CVE (Common Vulnerabilities and Exposures)
    - поддельные и злонамеренные образы
    - отсутствие подписей и проверки целостности
    - ошибка конфигурации и избыток прав — образы с избыточными правами доступа, запуском от root или с небезопасными настройками
    - присутствие секретов и конфиденциальных данных в образах
    - отсутствие регулярного обновления из-за неподдерживаемых образов

-  **Контекст безопасности**

    - Не задавать пользователей с правами «root» для работы сервисов внутри контейнеров. 
        > Если для функционирования сервиса не требуются расширенные привилегии, то в Dockerfile необходимо явно прописать учетную запись пользователя с минимально необходимыми правами.
    - Не запускать контейнеры в привилегированном режиме. 
        > Ключ «--privileged» отключает все средства изоляции (наложенные cgroup – контроллером устройства) docker-контейнера. Запуск контейнера с таким ключом обеспечит ему доступ к файловой системе и блочным устройствам (например, жесткому диску) хоста. Контейнеры должны быть запущены в непривилегированном режиме. Если контейнеру нужны дополнительные привилегии для корректной работы, то необходимо явно прописать или удалить требуемые docker capabilities.
    - Не отключать профили безопасности Docker. 
        > По умолчанию для запуска контейнеров Docker использует профили безопасности Linux, лучше использовать профили AppArmor, SELinux, grsecurity, seccomp, - позволяют ограничить активности контейнера, обеспечивая контроль сети, использования дополнительных возможностей (docker capabilities), контроль обращений к файловой системе хоста и пр. Контейнеры должны работать с активными профилями безопасности Docker, не docker-default. Если необходимо использовать другой профиль, то это можно сделать с помощью --security-opt.
    - Не допускать запуск контейнеров, использующих тип сети «host»
        > В режиме Host контейнер использует ту же сеть, что и хост, т.е. контейнерная сеть не изолируется от сети Docker хоста и контейнер не получает собственный IP-адрес, что дает доступ к REST API daemon docker изнутри контейнера, а также к устройствам, расположенным в сети хоста. Для реализации сетевого взаимодействия между контейнерами, они должны запускаться в режиме bridge или none.
    - Не разрешать доступ к docker.socket изнутри контейнера. Не подключать docker socket в контейнер без необходимости, либо с использованием плагинов авторизации. 
    - Не использовать секреты в открытом виде в Docker-файлах образов. По возможности не использовать переменные окружения и не хранить секреты внутри контейнера. Хранение и управление секретами возложить на сторонний сервис.
    - Ограничивать и контролировать использование ресурсов контейнерами. Указывать ограничения на уровне самого ПО или на уровне контейнеров для использования ресурсов хоста.
    - Контролировать качество базовых образов контейнеров. Использовать официальные образы и использовать образы с минимально необходимым набором инструментов.
    - Сканировать образы на наличие уязвимостей и проверки требований ИБ (Compliance Checks)

- **Дополнительно**

В случае, если возникает проблема с вызовом `docker buildx` для macos `silicon`, следует использовать вот [это](https://gist.github.com/Aeonitis/cbd9f8b61eaec5a8a024c0a42f415ca3) описание из gistup для фикса `samelink`.

***

## Выполнение задания

- [✔] 1. Поставьте `Docker` и `buildkit`

```bash
$ brew install buildkit
$ brew install docker

i@i8:~/Desktop/lab05$ brew install buildkit
==> Fetching downloads for: buildkit
✔︎ Bottle Manifest buildkit (0.26.2)                [Downloaded    7.5KB/  7.5KB]
✔︎ Bottle buildkit (0.26.2)                         [Downloaded    8.4MB/  8.4MB]
==> Pouring buildkit--0.26.2.x86_64_linux.bottle.tar.gz
🍺  /home/linuxbrew/.linuxbrew/Cellar/buildkit/0.26.2: 23 files, 23.3MB
==> Running `brew cleanup buildkit`...
Disable this behaviour by setting `HOMEBREW_NO_INSTALL_CLEANUP=1`.
Hide these hints with `HOMEBREW_NO_ENV_HINTS=1` (see `man brew`).
i@i8:~/Desktop/lab05$ brew install docker
==> Fetching downloads for: docker
✔︎ Bottle Manifest docker (29.1.3)                  [Downloaded    8.4KB/  8.4KB]
✔︎ Bottle Manifest docker-completion (29.1.3)       [Downloaded    2.0KB/  2.0KB]
✔︎ Bottle docker-completion (29.1.3)                [Downloaded   71.4KB/ 71.4KB]
✔︎ Bottle docker (29.1.3)                           [Downloaded    9.7MB/  9.7MB]
==> Installing docker dependency: docker-completion
==> Pouring docker-completion--29.1.3.all.bottle.tar.gz
🍺  /home/linuxbrew/.linuxbrew/Cellar/docker-completion/29.1.3: 10 files, 386.9KB
==> Pouring docker--29.1.3.x86_64_linux.bottle.tar.gz
🍺  /home/linuxbrew/.linuxbrew/Cellar/docker/29.1.3: 13 files, 28.4MB
==> Running `brew cleanup docker`...
Disable this behaviour by setting `HOMEBREW_NO_INSTALL_CLEANUP=1`.
Hide these hints with `HOMEBREW_NO_ENV_HINTS=1` (see `man brew`).
```

- [✔] 2. Перейдите в `source` и выведите на терминале, далее проанализируйте следующие команды консоли

```bash
$ docker buildx build -t hellow-appsec-world . # buildx - подсистема для расширенной сборки, -t - флаг для указания мени образа, hello-appsec-world - имя образа, точка - путь к контексту сборки
$ docker run hello-appsec-world # запуск нового контейнера с именем образа из которого будет создан контейнер hello-appsec-world
$ docker run --rm -it hello-appsec-world # --rm - удаление контейнера после остановки, -it - запуск в интерактивном режиме с терминалом

i@i8:~/Desktop/lab05/source$ sudo docker buildx build -t hello-appsec-world .
[+] Building 0.7s (13/13) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 429B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim        0.5s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:158caf0e  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 63B                                           0.0s
 => CACHED [builder 2/4] WORKDIR /hello                                    0.0s
 => CACHED [builder 3/4] COPY requirements.txt .                           0.0s
 => CACHED [builder 4/4] RUN pip install --upgrade pip && pip wheel --whe  0.0s
 => CACHED [stage-1 3/6] COPY --from=builder /wheels /wheels               0.0s
 => CACHED [stage-1 4/6] COPY requirements.txt .                           0.0s
 => CACHED [stage-1 5/6] RUN pip install --no-index --find-links=/wheels   0.0s
 => CACHED [stage-1 6/6] COPY hello.py .                                   0.0s
 => exporting to image                                                     0.0s
 => => exporting layers                                                    0.0s
 => => writing image sha256:130feb571d6c458b006e170eb78ad08fd8cdc8df5217a  0.0s
 => => naming to docker.io/library/hello-appsec-world  

i@i8:~/Desktop/lab05/source$ sudo docker run hello-appsec-world
hello appsec world # тут цветной вывод

i@i8:~/Desktop/lab05/source$ sudo docker run --rm -it hello-appsec-world
hello appsec world # и тут цветной вывод

$ docker save -o hello.tar hello-appsec-world # запись образа в архив hello.tar
$ docker load -i hello.tar # загрузка образа из файла hello.tar
$ docker load -i image.tar # загрузка из файла image.tar, такого нет, ловим ошибку

i@i8:~/Desktop/lab05/source$ sudo docker save -o hello.tar hello-appsec-world
i@i8:~/Desktop/lab05/source$ sudo docker load -i hello.tar
Loaded image: hello-appsec-world:latest
i@i8:~/Desktop/lab05/source$ sudo docker load -i image.tar
open image.tar: no such file or directory
```
- [✔] 3. Откройте `Dockerfile` и сделайте его анализ. Сделайте `commit`
- [✔] 4. Замените в `Dockerfile`значение скрипта на `python` тем, который вы сделали ранее в прошлых лабораторных работах. Вложите свой файл `python` в директорию. Сделайте анализ своего измененного `Dockerfile` и внесите изменения. Сделайте `commit`. 

```dockerfile
# Этап 1: сборка зависимостей
FROM python:3.11-slim AS builder
WORKDIR /hello
# Копируем файл с зависимостями
COPY requirements.txt .
# Устанавливаем зависимости в отдельную директорию wheelhouse для кеширования
RUN pip install --upgrade pip && pip wheel --wheel-dir=/wheels -r requirements.txt

# Этап 2: запускаемый образ
FROM python:3.11-slim 
WORKDIR /hello
# Копируем файл с зависимостями
COPY --from=builder /wheels /wheels
COPY requirements.txt .
# Устанавливаем зависимости из wheel-пакетов
RUN pip install --no-index --find-links=/wheels -r requirements.txt
# Копируем исходный код приложения
COPY hello.py .

# Переменные окружения для улучшенной работы Python
ENV PYTHONUNBUFFERED=1
# Запускаем приложение
CMD ["python", "hello.py"]
```

- [✔] 5. Выведите на терминале и проанализируйте следующие команды консоли. Сравните хеш сумму вашего архива с `image.tar` из репозитория, выведите на терминал.

```bash
$ docker buildx build -t hellow-appsec-world .
$ docker run hello-appsec-world
$ docker save -o hello_ypur_project.tar hello-appsec-world

$ docker load -i hello_ypur_project.tar
$ docker run hello-appsec-world

$ docker load -i image.tar
$ docker run hello-appsec-world

i@i8:~/Desktop/lab05/source$ sudo docker buildx build -t hello-appsec-world .
[+] Building 0.7s (13/13) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 1.05kB                                     0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim        0.5s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 695B                                          0.0s
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:158caf0e  0.0s
 => CACHED [builder 2/4] WORKDIR /hello                                    0.0s
 => CACHED [builder 3/4] COPY requirements.txt .                           0.0s
 => CACHED [builder 4/4] RUN pip install --upgrade pip && pip wheel --whe  0.0s
 => CACHED [stage-1 3/6] COPY --from=builder /wheels /wheels               0.0s
 => CACHED [stage-1 4/6] COPY requirements.txt .                           0.0s
 => CACHED [stage-1 5/6] RUN pip install --no-index --find-links=/wheels   0.0s
 => [stage-1 6/6] COPY hello.py .                                          0.1s
 => exporting to image                                                     0.0s
 => => exporting layers                                                    0.0s
 => => writing image sha256:fa76fcc623b0139fbcb8609f5fb6c11c51d2c9154ae68  0.0s
 => => naming to docker.io/library/hello-appsec-world                      0.0s
i@i8:~/Desktop/lab05/source$ sudo docker run hello-appsec-world
Добрый день, None !
i@i8:~/Desktop/lab05/source$ sudo docker save -o hello_your_project.tar hello-appsec-world
```

- [✔] 6. Доработайте свой `python` скрипт подключаемыми библиотеками, далее их необходимо разместить в `requirements.txt`. Размещение библиотек в следующем формате:
Доработал в пункте 5, ниже вид requirements.txt:
```
typer==0.9.0
```

- [✔] 7. Сделайте `commit`. Повторите сборку приложения по вашему `Dockerfile` для доработанного скрипта `python`. Сохраните `image` в виде .`tar` архива. Сделайте `commit`.
```bash
i@i8:~/Desktop/lab05/source$ git add Dockerfile 
i@i8:~/Desktop/lab05/source$ git add hello.py 
i@i8:~/Desktop/lab05/source$ git add requirements.txt 
i@i8:~/Desktop/lab05/source$ git commit -m "124"
[master 1fceaf8] 124
 3 files changed, 13 insertions(+), 8 deletions(-)
```
- [✔] 8. Выведите на терминале и проанализируйте следующие команды консоли

```bash
$ docker login # логин в docker-hub
$ docker tag hello-appsec-world yourusername/hello-appsec-world # присвоение тега(ссылки) образу (сам образ не меняется)
$ docker push yourusername/hello-appsec-world # загрузка образа в удалённый реестр (например dockerhub)
$ docker inspect yourusername/hello-appsec-world # выводит информацию об объектах docker
$ docker container create --name first hello-appsec-world # выпишите id контейнера

i@i8:~/Desktop/lab05/source$ sudo docker login

USING WEB-BASED LOGIN

i Info → To sign in with credentials on the command line, use 'docker login -u <username>'
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate

Waiting for authentication in the browser…

WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded

i@i8:~/Desktop/lab05/source$ sudo docker tag hello-appsec-world timoonnovi/hello-appsec-world
i@i8:~/Desktop/lab05/source$ sudo docker push timoonnovi/hello-appsec-world
Using default tag: latest
The push refers to repository [docker.io/timoonnovi/hello-appsec-world]
aee94fb8a860: Pushed 
76d7e99755f5: Pushed 
7c5d2ebd1f00: Pushed 
d2537b0b6747: Pushed 
a3a20903d1ee: Pushed 
fa384bf02ac1: Mounted from library/python 
600af8de593b: Mounted from library/python 
424dc4972605: Mounted from library/python 
77a2b55fbe8b: Mounted from library/python 
latest: digest: sha256:79bb32e22c4dc6e43e37f8f81c3940859e046bbdb8937f50705f8ece31d4e985 size: 2200
i@i8:~/Desktop/lab05/source$ sudo docker inspect timoonnovi/hello-appsec-world
[
    {
        "Id": "sha256:fa76fcc623b0139fbcb8609f5fb6c11c51d2c9154ae68f26f08bcd8cc977c73b",
        "RepoTags": [
            "hello-appsec-world:latest",
            "timoonnovi/hello-appsec-world:latest"
        ],
        "RepoDigests": [
            "timoonnovi/hello-appsec-world@sha256:79bb32e22c4dc6e43e37f8f81c3940859e046bbdb8937f50705f8ece31d4e985"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2025-12-14T20:40:05.251682646+03:00",
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Env": [
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D",
                "PYTHON_VERSION=3.11.14",
                "PYTHON_SHA256=8d3ed8ec5c88c1c95f5e558612a725450d2452813ddad5e58fdb1a53b1209b78",
                "PYTHONUNBUFFERED=1"
            ],
            "Cmd": [
                "python",
                "hello.py"
            ],
            "WorkingDir": "/hello",
            "ArgsEscaped": true
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 136950073,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/tv37gqa1lbz7zs2ikbvczj5ti/diff:/var/lib/docker/overlay2/afvbhwc9pegsku4e1ydbgwuby/diff:/var/lib/docker/overlay2/nx57ggk1pd42q1ecc45nuxlpa/diff:/var/lib/docker/overlay2/5vqscdrfz9mljpyrm7f0cqlnu/diff:/var/lib/docker/overlay2/88ba0241d99ab6a0c51f5631ccd122d2deb476a399ed889cea4a191912e57faf/diff:/var/lib/docker/overlay2/5efd4c8250668bb72663b05eafcc003ae44f5ff3877e33b794f9932976b4bc05/diff:/var/lib/docker/overlay2/c8e22a8d4d6f69056531124ff1c14d64f511ddf79707e697c0d2348dd45815bc/diff:/var/lib/docker/overlay2/a02a080a124826010095ad6133e9303d76c41aa5b85770e86a3e3d9594aedd3d/diff",
                "MergedDir": "/var/lib/docker/overlay2/z2t4stpebgoj80i4yfrjzlk29/merged",
                "UpperDir": "/var/lib/docker/overlay2/z2t4stpebgoj80i4yfrjzlk29/diff",
                "WorkDir": "/var/lib/docker/overlay2/z2t4stpebgoj80i4yfrjzlk29/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:77a2b55fbe8b9984ce0af3ffc0b0ab62507668e63306ec161a585e587a3eb164",
                "sha256:424dc4972605239ec660864fe4cc7bcf6ebdadd752a7ee7ad065a83c34798378",
                "sha256:600af8de593b464a3642857b2dce39ad42474145771745962fb90ea9c276fa9d",
                "sha256:fa384bf02ac198a84ae5f0bbe085a6e4bd2de0be1b595833ba52e9780a936ba9",
                "sha256:a3a20903d1ee75edc2a8fc108b0c236032ef443475e9a21a8bfe80000948e567",
                "sha256:d2537b0b674787abbfa50677b4ee8dc89c588851714f6e15530710b1977b8edf",
                "sha256:7c5d2ebd1f0007cf52e6a1c868445a00e7dc1a80ee547c4108a87026114416ee",
                "sha256:76d7e99755f59047c43473ddf8350e630d03d7dd4b2d752b712d920e200ab50c",
                "sha256:aee94fb8a860cd3f56c2a5a28bade95d4645818929e40b3d0a1bf714f955fe3a"
            ]
        },
        "Metadata": {
            "LastTagTime": "2025-12-14T20:54:40.108752559+03:00"
        }
    }
]

i@i8:~/Desktop/lab05/source$ sudo docker container create --name first hello-appsec-world
5ccd53c8213480f63a8ec1ea9c71b90bd64307b097a5a04ca5cafad3372bf8f8

$ docker image pull geminishkv/hello-appsec-world # ошибка
$ docker inspect geminishkvdev/hello-appsec-world # ошибка
$ docker container create --name second hello-appsec-world
i@i8:~/Desktop/lab05/source$ sudo docker image pull geminishkv/hello-appsec-world
Using default tag: latest
Error response from daemon: pull access denied for geminishkv/hello-appsec-world, repository does not exist or may require 'docker login': denied: requested access to the resource is denied

i@i8:~/Desktop/lab05/source$ sudo docker inspect geminishkvdev/hello-appsec-world
[]
Error: No such object: geminishkvdev/hello-appsec-world

i@i8:~/Desktop/lab05/source$ sudo docker container create --name second hello-appsec-world
33db7adb1b385d298d04542d3e18b6b4313ca886899d4a88f7f5d43fba7e75f3
``` 

- [✔] 9. Выведите на терминале и проанализируйте в консоли процессы, которые запущены, владельцев по пользователям

```bash 
 $ docker container run -it ubuntu /bin/bash
 i@i8:~/Desktop/lab05/source$ sudo docker container run -it ubuntu /bin/bash
root@d605c6d35f65:/# ps
    PID TTY          TIME CMD
      1 pts/0    00:00:00 bash
      9 pts/0    00:00:00 ps
root@d605c6d35f65:/# 

``` 
 
- [✔] 10. Выведите оба контейнера first и second на терминал
```bash
- i@i8:~/Desktop/lab05/source$ sudo docker container inspect first
[
    {
        "Id": "5ccd53c8213480f63a8ec1ea9c71b90bd64307b097a5a04ca5cafad3372bf8f8",
        "Created": "2025-12-14T17:55:38.301607836Z",
        "Path": "python",
        "Args": [
            "hello.py"
        ],
        "State": {
            "Status": "created",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "0001-01-01T00:00:00Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:fa76fcc623b0139fbcb8609f5fb6c11c51d2c9154ae68f26f08bcd8cc977c73b",
        "ResolvConfPath": "",
        "HostnamePath": "",
        "HostsPath": "",
        "LogPath": "",
        "Name": "/first",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                24,
                80
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/interrupts",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "ID": "5ccd53c8213480f63a8ec1ea9c71b90bd64307b097a5a04ca5cafad3372bf8f8",
                "LowerDir": "/var/lib/docker/overlay2/9711f9f1bb1ab2cd2b54f50f644ff2f8433e17214ec9e4e8579808f13c648e82-init/diff:/var/lib/docker/overlay2/z2t4stpebgoj80i4yfrjzlk29/diff:/var/lib/docker/overlay2/tv37gqa1lbz7zs2ikbvczj5ti/diff:/var/lib/docker/overlay2/afvbhwc9pegsku4e1ydbgwuby/diff:/var/lib/docker/overlay2/nx57ggk1pd42q1ecc45nuxlpa/diff:/var/lib/docker/overlay2/5vqscdrfz9mljpyrm7f0cqlnu/diff:/var/lib/docker/overlay2/88ba0241d99ab6a0c51f5631ccd122d2deb476a399ed889cea4a191912e57faf/diff:/var/lib/docker/overlay2/5efd4c8250668bb72663b05eafcc003ae44f5ff3877e33b794f9932976b4bc05/diff:/var/lib/docker/overlay2/c8e22a8d4d6f69056531124ff1c14d64f511ddf79707e697c0d2348dd45815bc/diff:/var/lib/docker/overlay2/a02a080a124826010095ad6133e9303d76c41aa5b85770e86a3e3d9594aedd3d/diff",
                "MergedDir": "/var/lib/docker/overlay2/9711f9f1bb1ab2cd2b54f50f644ff2f8433e17214ec9e4e8579808f13c648e82/merged",
                "UpperDir": "/var/lib/docker/overlay2/9711f9f1bb1ab2cd2b54f50f644ff2f8433e17214ec9e4e8579808f13c648e82/diff",
                "WorkDir": "/var/lib/docker/overlay2/9711f9f1bb1ab2cd2b54f50f644ff2f8433e17214ec9e4e8579808f13c648e82/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "5ccd53c82134",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D",
                "PYTHON_VERSION=3.11.14",
                "PYTHON_SHA256=8d3ed8ec5c88c1c95f5e558612a725450d2452813ddad5e58fdb1a53b1209b78",
                "PYTHONUNBUFFERED=1"
            ],
            "Cmd": [
                "python",
                "hello.py"
            ],
            "Image": "hello-appsec-world",
            "Volumes": null,
            "WorkingDir": "/hello",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "SandboxKey": "",
            "Ports": {},
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "",
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
i@i8:~/Desktop/lab05/source$ sudo docker container inspect second
[
    {
        "Id": "33db7adb1b385d298d04542d3e18b6b4313ca886899d4a88f7f5d43fba7e75f3",
        "Created": "2025-12-14T17:57:07.711309829Z",
        "Path": "python",
        "Args": [
            "hello.py"
        ],
        "State": {
            "Status": "created",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "0001-01-01T00:00:00Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:fa76fcc623b0139fbcb8609f5fb6c11c51d2c9154ae68f26f08bcd8cc977c73b",
        "ResolvConfPath": "",
        "HostnamePath": "",
        "HostsPath": "",
        "LogPath": "",
        "Name": "/second",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                24,
                80
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/interrupts",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "ID": "33db7adb1b385d298d04542d3e18b6b4313ca886899d4a88f7f5d43fba7e75f3",
                "LowerDir": "/var/lib/docker/overlay2/aed5a5f9e856181495129197dd7e5214ccbbe5ceacc450ab1d8fbd7966e65220-init/diff:/var/lib/docker/overlay2/z2t4stpebgoj80i4yfrjzlk29/diff:/var/lib/docker/overlay2/tv37gqa1lbz7zs2ikbvczj5ti/diff:/var/lib/docker/overlay2/afvbhwc9pegsku4e1ydbgwuby/diff:/var/lib/docker/overlay2/nx57ggk1pd42q1ecc45nuxlpa/diff:/var/lib/docker/overlay2/5vqscdrfz9mljpyrm7f0cqlnu/diff:/var/lib/docker/overlay2/88ba0241d99ab6a0c51f5631ccd122d2deb476a399ed889cea4a191912e57faf/diff:/var/lib/docker/overlay2/5efd4c8250668bb72663b05eafcc003ae44f5ff3877e33b794f9932976b4bc05/diff:/var/lib/docker/overlay2/c8e22a8d4d6f69056531124ff1c14d64f511ddf79707e697c0d2348dd45815bc/diff:/var/lib/docker/overlay2/a02a080a124826010095ad6133e9303d76c41aa5b85770e86a3e3d9594aedd3d/diff",
                "MergedDir": "/var/lib/docker/overlay2/aed5a5f9e856181495129197dd7e5214ccbbe5ceacc450ab1d8fbd7966e65220/merged",
                "UpperDir": "/var/lib/docker/overlay2/aed5a5f9e856181495129197dd7e5214ccbbe5ceacc450ab1d8fbd7966e65220/diff",
                "WorkDir": "/var/lib/docker/overlay2/aed5a5f9e856181495129197dd7e5214ccbbe5ceacc450ab1d8fbd7966e65220/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "33db7adb1b38",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D",
                "PYTHON_VERSION=3.11.14",
                "PYTHON_SHA256=8d3ed8ec5c88c1c95f5e558612a725450d2452813ddad5e58fdb1a53b1209b78",
                "PYTHONUNBUFFERED=1"
            ],
            "Cmd": [
                "python",
                "hello.py"
            ],
            "Image": "hello-appsec-world",
            "Volumes": null,
            "WorkingDir": "/hello",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "SandboxKey": "",
            "Ports": {},
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "",
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
```
- [✔] 11. Перейдите в основной корень `lab05` и выведите на терминале, и проанализируйте

```bash 
$ docker-compose up --build # сборка сервисов из docker-compose.yml
i@i8:~/Desktop/lab05$ sudo docker-compose up --build
Creating network "lab05_app_net" with the default driver
Building server
[+] Building 14.8s (14/14) FINISHED                              docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 419B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim        1.2s
 => [auth] library/python:pull token for registry-1.docker.io              0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load build context                                          0.1s
 => => transferring context: 842B                                          0.0s
 => CACHED [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:1  0.0s
 => [builder 2/4] WORKDIR /app                                             0.2s
 => [builder 3/4] COPY requirements.txt .                                  0.0s
 => [builder 4/4] RUN pip install --upgrade pip && pip wheel --wheel-dir=  8.9s
 => [stage-1 3/6] COPY --from=builder /wheels /wheels                      0.0s 
 => [stage-1 4/6] COPY requirements.txt .                                  0.0s 
 => [stage-1 5/6] RUN pip install --no-index --find-links=/wheels -r requ  3.8s 
 => [stage-1 6/6] COPY app.py .                                            0.1s 
 => exporting to image                                                     0.2s 
 => => exporting layers                                                    0.2s 
 => => writing image sha256:8e42099817e32ff17c7e7655e61c22340e9f23370d211  0.0s 
 => => naming to docker.io/library/lab05_server                            0.0s 
Building client                                                                 
[+] Building 12.7s (13/13) FINISHED                              docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 425B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim        0.2s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 576B                                          0.0s
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:158caf0e  0.0s
 => CACHED [builder 2/4] WORKDIR /app                                      0.0s
 => [builder 3/4] COPY requirements.txt .                                  0.0s
 => [builder 4/4] RUN pip install --upgrade pip && pip wheel --wheel-dir=  8.2s
 => [stage-1 3/6] COPY --from=builder /wheels /wheels                      0.0s 
 => [stage-1 4/6] COPY requirements.txt .                                  0.0s 
 => [stage-1 5/6] RUN pip install --no-index --find-links=/wheels -r requ  3.7s 
 => [stage-1 6/6] COPY client.py .                                         0.1s 
 => exporting to image                                                     0.2s 
 => => exporting layers                                                    0.2s 
 => => writing image sha256:f8a6e4cc789b9205f86775824c5d75a0f63594ea4dab9  0.0s 
 => => naming to docker.io/library/lab05_client                            0.0s 
Creating lab05_server_1 ... done
Creating lab05_client_1 ... done
Attaching to lab05_server_1, lab05_client_1
server_1  |  * Serving Flask app 'app'
server_1  |  * Debug mode: off
server_1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
server_1  |  * Running on all addresses (0.0.0.0)
server_1  |  * Running on http://127.0.0.1:8000
server_1  |  * Running on http://172.18.0.2:8000
server_1  | Press CTRL+C to quit
server_1  | 172.18.0.3 - - [14/Dec/2025 18:21:48] "GET / HTTP/1.1" 200 -
client_1  | 
client_1  |     <html>
client_1  |     <head><title>Colorful Output</title></head>
client_1  |     <body style="font-family: monospace; font-size: 24px;">
client_1  |     <span style="color:red">h</span><span style="color:green">e</span><span style="color:yellow">l</span><span style="color:blue">l</span><span style="color:purple">o</span><span style="color:red"> </span><span style="color:green">a</span><span style="color:yellow">p</span><span style="color:blue">p</span><span style="color:purple">s</span><span style="color:red">e</span><span style="color:green">c</span><span style="color:yellow"> </span><span style="color:blue">w</span><span style="color:purple">o</span><span style="color:red">r</span><span style="color:green">l</span><span style="color:yellow">d</span>
client_1  |     </body>
client_1  |     </html>
client_1  |     
lab05_client_1 exited with code 0
server_1  | 172.18.0.1 - - [14/Dec/2025 18:25:08] "GET / HTTP/1.1" 200 -
server_1  | 172.18.0.1 - - [14/Dec/2025 18:25:08] "GET /favicon.ico HTTP/1.1" 404 -
``` 

- [✔] 12. Откройте соседнее окно терминала и и выведите на терминале

```bash 
$ open -a "Google Chrome" http://localhost:8000
сделано, открылась вкладка с цветной надписью @Hello appsec world"
```

- [✔] 13. Остановите работу `docker-compose`.

```bash 
$ docker ps -a
$ docker ps -q
$ docker images

$ docker ps -q | xargs docker stop
$ docker-compose down

i@i8:~$ sudo docker ps -a
[sudo] password for i: 
CONTAINER ID   IMAGE                COMMAND              CREATED          STATUS                        PORTS                                         NAMES
0a032a49ad3c   lab05_client         "python client.py"   5 minutes ago    Exited (0) 2 minutes ago                                                    lab05_client_1
a6c35351f6b9   lab05_server         "python app.py"      5 minutes ago    Up 5 minutes                  0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp   lab05_server_1
522ac4305f16   hello-appsec-world   "python hello.py"    11 minutes ago   Exited (0) 11 minutes ago                                                   infallible_chaum
d605c6d35f65   ubuntu               "/bin/bash"          27 minutes ago   Exited (0) 19 minutes ago                                                   priceless_aryabhata
8258920a68e7   ubuntu               "/bin/bash"          28 minutes ago   Exited (127) 27 minutes ago                                                 priceless_dirac
33db7adb1b38   hello-appsec-world   "python hello.py"    29 minutes ago   Created                                                                     second
5ccd53c82134   hello-appsec-world   "python hello.py"    31 minutes ago   Created                                                                     first
fba026cc68c8   hello-appsec-world   "python hello.py"    45 minutes ago   Exited (0) 45 minutes ago                                                   hopeful_johnson
946c2aa27593   hello-appsec-world   "python hello.py"    46 minutes ago   Exited (0) 46 minutes ago                                                   lucid_liskov
c1bb5e4ed9c0   8d57ac9e6ba3         "python hello.py"    49 minutes ago   Exited (1) 49 minutes ago                                                   exciting_kapitsa
35c98ce9ac14   1805048b8a1f         "python hello.py"    49 minutes ago   Exited (1) 49 minutes ago                                                   sharp_blackwell
01ec44be5f5d   82e1b7e4306c         "python hello.py"    49 minutes ago   Exited (1) 49 minutes ago                                                   fervent_galileo
40abfd10dbbe   82e1b7e4306c         "python hello.py"    49 minutes ago   Exited (1) 49 minutes ago                                                   interesting_dirac
a6e51c077881   09ec8100a710         "python hello.py"    50 minutes ago   Exited (1) 50 minutes ago                                                   sleepy_almeida
841dd1a7973c   d5eeef8f5cc8         "python hello.py"    54 minutes ago   Exited (1) 54 minutes ago                                                   goofy_diffie
37c5714ef0f9   d5eeef8f5cc8         "python hello.py"    56 minutes ago   Exited (1) 56 minutes ago                                                   hardcore_kirch
fd50812f4ced   130feb571d6c         "python hello.py"    2 hours ago      Exited (0) 2 hours ago                                                      wonderful_booth

i@i8:~$ sudo docker ps -q
a6c35351f6b9

i@i8:~$ sudo docker images
REPOSITORY                      TAG       IMAGE ID       CREATED          SIZE
lab05_client                    latest    f8a6e4cc789b   5 minutes ago    138MB
lab05_server                    latest    8e42099817e3   6 minutes ago    141MB
hello-appsec-world              latest    fa76fcc623b0   47 minutes ago   137MB
timoonnovi/hello-appsec-world   latest    fa76fcc623b0   47 minutes ago   137MB
<none>                          <none>    8d57ac9e6ba3   49 minutes ago   137MB
<none>                          <none>    1805048b8a1f   50 minutes ago   137MB
<none>                          <none>    82e1b7e4306c   50 minutes ago   137MB
<none>                          <none>    09ec8100a710   51 minutes ago   137MB
<none>                          <none>    d5eeef8f5cc8   56 minutes ago   135MB
hellow-appsec-world             latest    130feb571d6c   2 hours ago      135MB
ubuntu                          latest    c3a134f2ace4   8 weeks ago      78.1MB

^CGracefully stopping... (press Ctrl+C again to force)
Stopping lab05_server_1 ... done
i@i8:~/Desktop/lab05$ sudo docker ps -q | xargs docker stop
docker: 'docker stop' requires at least 1 argument

Usage:  docker stop [OPTIONS] CONTAINER [CONTAINER...]

See 'docker stop --help' for more information
i@i8:~/Desktop/lab05$ sudo docker-compose down
Removing lab05_client_1 ... done
Removing lab05_server_1 ... done
Removing network lab05_app_net
```
- [✔] 14. Доработайте `docker-compose` и скрипт, который вы подготовили ранее, что бы вы смогли воспроизвести шаги п.11 по п.13 с демонстрацией. Сделайте `commit`.
- [✔] 15. Залейте изменения в свой удаленный репозиторий, проверьте историю `commit`.
- [✔] 16. Подготовьте отчет `gist`.
 
***
