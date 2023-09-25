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
RPD * DAU / 86 400

Скачивание
- **Скачивание видео**: 1/7 * DAU / 86 400 = 9.
- **Скачивание png**: 1 * DAU / 86 400 = 63.
- **Скачивание jpg**:  2 * DAU / 86 400 = 126.
- **Скачивание документов**: 1 * DAU / 86 400 = 63.

Загрузка
- **Загрузка на диск видео**: 2/7 * DAU / 86 400 = 18.
- **Загрузка на диск png**: 1 * DAU / 86 400 = 63.
- **Загрузка на диск jpg**: 4 * DAU / 86 400 = 252.
- **Загрузка на диск документов**: 1 * DAU / 86 400 = 9.

Удаление
- **Удаление**: 1 * DAU / 86 400 = 63.

Просмотр
- **Просмот директории***: 1 * DAU / 86 400 = 63.

Общее
- **RPS** = 791.
- **Пиковый RPS (x2)** = 1582

### Расчет сетевого траффика
DAU = 5.5 M

Просмотр директории не будет критически влиять на сетевой траффик, так как на фоне скачиваемых/загружаемых файлов это всего лишь несколько байт, поэтому мы не будем учитывать его при расчете сетевого траффика.\
Также мой диск не будет поддерживать частичную замену файла, в случае загрузки на диск файлов, отличающихся только частично.

#### Дневной траффик:
RPS * объем файлов, который требует дейcтвие * 86 400

- **Скачивание с диска**: (9 * 512 MiB + 63 * 5.12 MiB + 126 * 1.12 MiB + 63 * 2.5 MiB) * 86 400 = 441 212 GiB / day
- **Загрузка на диск**: (18 * 512 MiB + 63 * 5.12 MiB + 252 * 1.12 MiB + 9 * 2.5 MiB) * 86 400 = 830 528 GiB / day 
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
| DAU  | 5.5M    |
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
| RPS | 791 |
| Пиковый RPS | 1582 |

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






