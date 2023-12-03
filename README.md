# Проектирование высоконагружнных приложений

## Тема и целевая аудитория

Тема - "Проектирование сервиса по типу Google Drive, DropBox"

Функционал (MVP) :
- загрузка файла
- удаление файла
- посмотр информации о файле
- поиск по директории
- поделиться ссылкой на файл


Целевая аудитория:
- [700 m пользователей по всему миру](https://investors.dropbox.com/static-files/51a64b81-6879-4d70-9ed2-a05b8d785db4)
- Пользователи по странам [hypestat](https://hypestat.com/info/dropbox.com) и [Блог DropBox](https://blog.dropbox.com/topics/company/5-million-dropboxes):
  | Страна         | Процент от всех юзеров   |
  | -------------- | ------------------------ |
  | США            | ~35 %   |
  | Великобритания | ~6 %    |
  | Германия       | ~4 %    |
  | Япония         | ~4 %    |
  | Канада         | ~3.5 %  |
  | Испания        | ~3 %    |

  Большинство ссылок на ресурсы DropBox ведут с американских ресурсов
- Самая многочисленная возрастная группа — 25 - 34

## Расчет нагрузки

### Объем хранилища и аудитория

<!-- Всего 700M зарегистрированных пользователей, из них 17.7M - платящая адуитория, остальные 682.3M - пользуются бесплатным акканутом. Бесплатный аккаунт предоставляет 2Gb бесплатно + 0.5 Gb в среднем за реферальные ссылки. 

Из платящей аудитории 65 % пользуются индивидульным аккаунтом, который предосталяет в среднем 2.5 TB.
Остальные 35 % пользуются командным аккаунтом, который в среднем предоставляет 4 Tb на человека.
Будем считать, что в среднем эта аудитория заполняет все пространство. 

Общий размер хранилища:
- 682.2 M * 2.5 Gb + 17.7M * 0.65 * 2.5 Tb + 17.7M * 0.35 * 4 Tb = 55 248 Pb

Средний размер хранилища пользователя:
- 55 248 Pb / 700 M = 78 Gb -->

На основании собственного опыта и опроса знакомых посчитаем использование файлового хранилища:

PNG
- **Средний размер PNG картинки**: ( 1920px * 1080 px * 4b + 33b (header) ) * 0.66 (среднее сжатие DEFLATE) ~= 5.12 MiB.
- **Среднее количетсво картинок у пользователя**: 500.
- **Средний объем картинок у пользователя**: 5.12 MiB * 500 ~= 2.5 GiB.

JPG
- **Средний размер JPG картинки**: ( 1920px * 1080 px * 3b + 600b (header) ) * 0.2 (среднее сжатие) ~= 1.12 MiB.
- **Среднее количетсво картинок у пользователя**: 2000.
- **Средний объем картинок у пользователя**: 1.12 MiB * 2000 ~= 2.3 GiB.

Видео
- **Средний размер видео**: 512 MiB.
- **Среднее количетсво видео у пользователя**: 100.
- **Средний объем видео у пользователя**: 512 MiB * 100 = 50 GiB.

Текст
- **Средний размер текстового файла**: 5 KiB.
- **Среднее количетсво документов у пользователя**: 200.
- **Средний объем документов у пользователя**: 5 KiB * 200 ~= 1 MiB.

Документ
- **Средний размер документа**: 2.5 MiB.
- **Среднее количетсво документов у пользователя**: 500.
- **Средний объем документов у пользователя**: 2.5 MiB * 500 ~= 1.2 gb.

**Среднее количество файлов пользователя**: 3200
**Общее кол-во файлов**: 3200 * 700 m = 2240 MM

Общий размер
- **Общий размер пользовательского хранилища**: ~56 GiB.
- **Общий размер всего хранилища**: ~37 443 PiB.

В будущем не будем учитывать текстовые файлы при расчетах, так как они занимают слишком мало пространства по сравнению с остальным контентом.

Для статического контента будем использовать CDN, поэтому делигируем на эти сервисы ответственность доставки статического контента и не будем учитывать его при рассчетах.

### Среднее количество действий пользователя день:

(Просмотр файла также будем считать за скачивание.)
| Действие   | Среднее количество дейстивий |
| -------- | ------- |
| Скачивание видео | 2 в неделю |
| Скачивание png | 1 в день |
| Скачивание jpg | 2 в день |
| Скачивание документов | 1 в день |
| Загрузка на диск видео | 2 в неделю |
| Загрузка на диск png | 1 в день |
| Загрузка на диск jpg | 4 в день |
| Загрузка на диск документов | 1 в день |
| Удаление | 1 в день |
| Просмотр директории | 1 в день |

### RPS
DAU = 5.5 M
RPD * DAU / 86 400

Скачивание
- **Скачивание видео**: 1/7 * DAU / 86 400 = 9.
- **Скачивание png**: 1 * DAU / 86 400 = 63.
- **Скачивание jpg**:  2 * DAU / 86 400 = 126.
- **Скачивание документов**: 1 * DAU / 86 400 = 63.

Загрузка\
(Файлы будут загрузаться чанками по 10 MBi с клиента, поэтому любое действие загрузки будет домножать на среднее кол-во файлов объекта, чанкованию подвержены только видео.)
- **Загрузка на диск видео**: 2/7 * ( 512 / 10 )  * DAU / 86 400 = 921.
- **Загрузка на диск png**: 1 * DAU / 86 400 = 63.
- **Загрузка на диск jpg**: 4 * DAU / 86 400 = 252.
- **Загрузка на диск документов**: 1 * DAU / 86 400 = 9.

Удаление
- **Удаление**: 1 * DAU / 86 400 = 63.

Просмотр
- **Просмот директории***: 1 * DAU / 86 400 = 63.

Общее
- **RPS** = 1694.
- **Пиковый RPS (x2)** = 3388.

### Расчет сетевого траффика
DAU = 5.5 M

Просмотр директории не будет критически влиять на сетевой траффик, так как на фоне скачиваемых/загружаемых файлов это всего лишь несколько байт, поэтому мы не будем учитывать его при расчете сетевого траффика.\
Также мой диск не будет поддерживать частичную замену файла, в случае загрузки на диск файлов, отличающихся только частично.

#### Дневной траффик:
RPS * объем файлов, который требует дейcтвие * 86 400

- **Скачивание с диска**: (9 * 512 MiB + 63 * 5.12 MiB + 126 * 1.12 MiB + 63 * 2.5 MiB) * 86 400 = 441 212 GiB / day
- **Загрузка на диск**: (921 * 10 MiB + 63 * 5.12 MiB + 252 * 1.12 MiB + 9 * 2.5 MiB) * 86 400 = 830 528 GiB / day 
- **Дневной траффик**: 1 271 740 GiB / day

#### Пиковый траффик в секунду:
Дневной траффик / 86 400 * 2

- **Скачивание с диска**: 441 212 GiB / 86 400 * 2 = 10.2 GiB / s
- **Загрузка на диск**: 830 528 GiB / 86 400 * 2 = 19.2 GiB / s
- **Пиковый траффик в секунду**: 29.4 GiB / s

### Продуктовые метрики:
| Метрика    | Значение |
| -------- | ------- |
| Monthly visits | 150M |
| DAU  | 5.5M |
| MAU | 67M |
| Хранилище пользователя | 56 GiB |
| Хранилище пользователя (png) | 2.5 GiB |
| Хранилище пользователя (jpg) | 2.3 GiB |
| Хранилище пользователя (видео) | 50 GiB |
| Хранилище пользователя (документы) | 1.2 GiB |

### Технические метрики:
| Метрика    | Значение |
| -------- | ------- |
| Общий размер хранилища  | 37 443 PiB |
| Траффик в день | 1 271 740 GiB / day |
| Пиковый траффик в секунду | 29.4 GiB / s |
| RPS | 1694 |
| Пиковый RPS | 3388 |

## Глобальная балансировка

### Выбор расположения датацентров

Большая часть аудитории находится в США (35 %), а остальная часть аудитории достаточно равномерно распределена между Европой (в основном Западная Европа) и Азией (В основном Япония).
Поэтому 3 ЦОД'а будут расположены в США, 1 в Западной Европе и 1 в Японии.

### Выбор способа глобальная балансировки

Для балансировки клиентов между континентами будем использовать DNS балансировку.
Возможен вариант использования Geo Based Dns балансировки (например Amazon Route 53, так как они имеют достаточно большую карту маппинга ip и геопозиции) или Latency Based Dns (при использовании ЦОД'ов от AWS для наиболее точных показателй).

Так как показатель Latency при большой нагрузке на траффик файлового хранилища требует минимизации, в пределах США будем использовать BGP Anycast для выбора нужного ЦОД'а в Америке.
В США будет использоваться один IP для AS, который будет сопостовлять с IP настоящего ЦОД'а.
Разобьем все линки BGP сети условно на 3 группы, каждая из которых будет соответствовать одному из трех ЦОД'ов.
При загруженности (или при полному отказе одного из ЦОД'ов) мы можем скорректировать количство хопов для снижения нагрузки на ЦОД.

## Локальная балансировка
Балансировка будет осуществляться посредством роутинга и L7 балансировщиков.

После приземления клиента в цод маршрут выбирается на основании **Routing'а ECMP**.
Будем использовать **Per-Flow** алгоритм хэширования, чтобы пакеты с одного IP попадали на один и тот же балансировщик. А также использовать **Resilient/Consistent Hashing** для минимизации разрыва TCP при добавлении новых путей.

Каждый ECMP маршрут будет связан с репликасетом из L7 балансировщиков с одним вирутальным IP, резервируемым с помощью **keepalived VRRP** .

В качестве L7 балансировщика выберем **envoy**. 
Балансировщик будет осуществлять **liveness пробы**, чтобы проверять работоспособность бэкендов.
SSL терминация также будет происходить за счет балансировщика.

## Логическая схема данных

### Схема таблицы юзеров:
<img src="https://github.com/stepanpopov/highload-google-drive/assets/77172612/313c4572-1ea7-49bf-bda7-4307dcf5ad2e" width="300">

#### РАССЧЕТ:

- name = 30 * 2 = 60 байт
- password = 30 * 2 = 60 байт
В среднем email занимает 30 символов:
- email = 30 *2 = 60 байт

Общее: 
- 180 байт * 700 m = 116 GiB

### Схема таблицы для файлов:
<img src="https://github.com/stepanpopov/highload-google-drive/assets/77172612/b4c24de2-b4c7-4f51-8b35-c9983342b402" width="600">

Сами данные файлов будем хранить в стороннем S3 сервисе, поэтому в базе данных будем хранить только пути до файлов.
Названием файла в s3 бакете будет хэш файла, чтобы при дублировании файлов не делать дополнительные запросы к S3 апи.
Частичная загрузка файлов будет также происходить за счет s3.

#### РАССЧЕТ для Files:
- file_uuid = 16 байт
- В среднем пользователи хранят папки 3его уровня вложенности и название в 15 символов
  path = 3 * 15 * 2 = 90 байт
- file_size = 4 байт
- file_type = 30 байт
- s3_url = 256 байт
- modify_date = 8 байт
- create_date = 8 байт
- checksum = 20 байт

Общее: 
- (16 + 90 + 4 + 30 + 256 + 8 + 8 + 20 + 4) байт * 3200(среднее кол-во файлов у пользователя) * 700m = 880 Tib

### Схема таблиц для multipart загрузки
<img src="https://github.com/stepanpopov/highload-google-drive/assets/77172612/3407e28f-d8ec-4c81-b615-768c9e49556b" width="600">

Эта таблица будет использоваться для проверки прав доступа пользователя в процессе загрузки файла по частям. Каждая запись будет иметь TTL в 2 часа.

#### РАССЧЕТ:
- user_uuid = 16 байт
- multipart_id = 56 байт

В среднем пользователь загружает 6 + 2/7 файлов в день. Посчитаем максимальный размер с учетом, что юзер может загружать все файлы в одном время.
Общий максимальный размер:
- (16 + 56) * (6 + 2/7) * 700m = 295 Gib

### Схема таблицы для проверки прав:
<img src="https://github.com/stepanpopov/highload-google-drive/assets/77172612/ac61fdd8-9987-46c9-883e-f5aafa6ab7d2" width="300">

Ключом бакета шардирования будет хэш функция от file_uuid.

#### РАССЧЕТ:
- file_uuid = 16 байт
- user_uuid = 16 байт
- permissions = 1 байт (храним 0,1,2)

В среднем юзер делится каждым 10 файлом с тремя людьми.\
Общее: 
- (16 байт + 16 байт + 1 байт) * 700m * 3200(среднее кол-во файлов у пользователя) * 3 / 10 = 20 TiB.

## Физическая схема данных

#### PostgreSql
Postgresql поддерживает механизм ltree деревьев и представления в виде Materialized Path, 

Индексы для таблицы Files:
- file_uiid - hash unique index
- path - btree index для поиска от корня
- gin index на последнюю метку path для полнотекстового поиска

Шардирование для таблицы Files по file_uuid

#### S3
Данные файлов будем хранить в облачном S3 хранилище.
S3 бакет будем формировать по user_id. Частичная загрузка файлов будет также происходить за счет s3.

#### Tarantool
Для таблицы Users, ACL и Multipart upload будем использовать in memory db tarantool с фреймворком cartridge для шардирования.

Каждый шард будет иметь свой репликасет для резервирования.
Для доступа к шардам и маппинга бакета на шард будем использовать несколько stateless роутеров. 

Для таблицы Multipart Upload будет использовать встроенный механизм вытеснения данных c TTL в 2 часа.

Индексы для таблицы Users:
- user_uuid - hash unique index

Индексы для таблицы ACL:
- file_uuid и user_uiid - составной hash index

Индексы для таблицы Multipart Upload:
- multipart_id - hash index

Шардирование Users по по хэш функции от ключа user_uuid. (consistent hashing).\
Шардирование ACL по по хэш функции от ключа file_uuid. (consistent hashing).\
Шардирование Mutipart_Upload по хэш функции от ключа multipart_id (consistent hashing).

#### Дампы
Каждого шарда любой базы данных будет иметь дополнительную реплику, с которой будет сниматься dump и отправляться в S3 Cold storage (например S3 Standard-IA). 

Для хранения метрик будем использовать Prometheus и time-series db InfluxDB. 

## Аппаратное и программное обеспечение

Подключение дисков с технологией Raid 10 для обеспечения наибольшей произодительности при приемлемой надежности.
Raid будем делать на программном уровне для обеспечения независимости дисков от raid контроллера.

## Распределенные алгоритмы

#### Рассмотрим механизм синхронизации хранилища между разными устройствами.
Когда пользователь впервые загружает файл, эта информация отправляется на все синхронизируемые устройства, которые держат web socket соединение.
На клиенте у пользователя постоянно работает процесс Watcher, который проверяет изменения файлов на диске и локально сохраняет дерево файлов и время последнего изменения.

При запросе на синхронизацию с хранилищем клиент запрашивает листинг директорий, который сервис List dirs отправляет как сжатый json с деревом всех файлов пользователя и временем последнего изменения. Разница деревьев, на основании которой пользователь скачивает или загружает файлы, рассчитывается на клиенте.
Если автоматическая синхронизация включена, то с сервисом Centryfugo устанавливается web socket соединение и пользователю отправляется любое изменение.

По окончании загрузки нового файла, сервис Сoordinator отправляет событие об изменении real-time messaging сервису Centryfugo, который впоследствии отправляет его по веб сокет соединению клиенту.
При удалении файла сервис Delete file также отправляет событие Centryfugo. 

#### Рассмотрим механизм загрузки файлов по частям
Когда клиент хочет начать загрузку файла, он разбивает его на части по 100 MB (стандарт AWS) и отправляет запрос сервису Init Upload на загрузку файла. Сервис upload инициирует multipart загрузку файла в S3 и возвращает клиенту идентификатор multipart загрузки, а также записывает его в Tarantool.

Далее клиент включает его в каждый запрос на загрузку части файла, указывая сам номер части и чексумму части файла. Сервис Upload на каждый запрос загрузки проверяет доступ пользователя к multipart id и возвращает статус операции. 

Когда клиент загрузит все части, он отправляет сервису Upload запрос на окончание загрузки с чексуммой всего файла. Сервис Upload опять сверяет multipart id, делая запрос к Tarantool и передает его сервису Finish Upload, который проверяет чексумму файла и проводит распределенную транзакцию, заканчивая загрузку файла в S3 и доабвляет метаинформацию о файле в таблицу Files в Postgres. 

В S3 должно быть настроено удаление недогруженных файлов при multipart загрузке по истечению определенного времени, чтобы неиспользуемые файлы не копились в объектном хранилище.

## Технологии
|Технология| |
|----|----|
|Rust | Web Assembly модуль на Envoy для поддержки логгирования и запросов на авторизацию |
|Golang | для микросервисов |
| Envoy | локальный балансировщик |
|React + TS | фронтенд |
|ClickHouse | логи |
|Tarantool | inmemory db |
| Postgresql | основная субд |
| AWS Cloudfront | для для доставки статического контента и файлов из S3. AWS имеет поддержку работы с S3 хранилищами, а также сервис AWS Functions позволяет имплементировать кастомную логику, которая понадобится нам для авторизации перед скачиванием файлов.| 
|Jagger |для трейсинга микросервисных походов|
|Cenryfugo | для real-time оповещений клиентов при синхронизации |

## Архитектура
![image](https://github.com/stepanpopov/highload-google-drive/assets/77172612/0a600ff9-acc0-47bd-b7c6-cc76e97e01ed)

## Выбор оборудования и хостинг

|Серивис| Целевая нагрузка | Ram | CPU | Net | 
|----|----|----|----|----|

K8:
|Серивис| CPU/r	|CPU/l	|RAM/r	|RAM/l	|Cnt|
|----|----|----|----|----|----|

| Название |Хостинг |Конфигурация |	Cores |	Cnt |	Покупка |	Аренда |
|----|----|----|----|----|----|----|


