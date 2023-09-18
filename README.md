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

Всего 700M зарегистрированных пользователей, из них 17.7M - платящая адуитория, остальные 682.3M - пользуются беспатным акканутом. Бесплатный аккаунт предоставляет 2Gb бесплатно + 0.5 Gb в среднем за реферальные ссылки. 

Из платящей аудитории 65 % пользуются индивидульным аккаунтом, который предосталяет в среднем 2.5 TB.
Остальные 35 % пользуются командным аккаунтом, который в среднем предоставляет 4 Tb на человека.
Будем считать, что в среднем эта аудитория заполняет все пространство.

Общий размер хранилища:
- 682.2 M * 2.5 Gb + 17.7M * 0.65 * 2.5 Tb + 17.7M * 0.35 * 4 Tb = 55 248 Pb

Средний размер хранилища пользователя:
- 55 248 Pb / 700 M = 78 Gb

Всего загружено 600B файлов.
В среднем пользователь хранит: 600B / 700M = 857 файлов.

В среднем файл весит: 55 248 Pb / 600B = 92 Mb.

Средний размер картинки: 7 mb.\
Среднее количетсво картинок у пользователя: 1000.\
Средний объем картинок у пользователя: 7MB * 1000 = 7 GB.\

Средний размер видео: 1 gb.\
Среднее количетсво видео у пользователя: 70.\
Средний объем видео у пользователя: 1 GB * 70 = 70 GB.\

Средний размер документа: 1 MB.\
Среднее количетсво документов у пользователя: 1000.\
Средний объем документов у пользователя: 1 MB * 1000 = 1 gb.\

### Среднее количество действий пользователя день:

| Действие   | Среднее количество в день |
| -------- | ------- |
| Скачивание | 10 |
| Загрузка на диск  | 10 |
| Удаление | 5 |
| Просмотр | 20 |

### RPS

RPS = (10 + 10 + 5 + 20) * DAU / 86 400 = 2 864

Пиковый RPS = RPS * 2 = 5728

### Расчет сетевого траффика

Для бесплатного аккаунта ограничением является: 20Gb/day, загружать за раз можно макс 10Gb.
Для платного аккаунта ограничением является: 200Gb/day, загружать за раз можно макс 100Gb.

DAU = 5.5 M

Для суммарного дневного потребления будем считать, что дневные юзеры расходуют 30% разрешенного траффика b между загрузкой и удалением нагрузка распределяется 50 на 50:
Суммарное потребление для загрузки у бесплатных аккаунтов: 5.5 M * 0.65 * 0.3 * 10 Gb = 11.25 Pb/d
Суммарное потребление для скачивания у бесплатных аккаунтов: 5.5 M * 0.65 * 0.3 * 10 Gb = 11.25 Pb/d

Суммарное потребление для загрузки у платных аккаунтов: 5.5 M * 0.35 * 0.3 * 100 Gb = 57.75 Pb/d
Суммарное потребление для скачивания у платных аккаунтов: 5.5 M * 0.35 * 0.3 * 100 Gb = 57.75 Pb/d

### Продуктовые метрики:
| Метрика    | Значение |
| -------- | ------- |
| Monthly visits | 150M |
| DAU  | 5.5M    |
| MAU | 67M |
| Хранилище пользователя | 78 Gb |
| Хранилище пользователя (картинки) | 7 GB |
| Хранилище пользователя (видео) | 70 GB |
| Хранилище пользователя (документы) | 1 GB |

### Технические метрики:
| Метрика    | Значение |
| -------- | ------- |
| Общий размер хранилища  | 55 248 Pb |
| Суммарное потребление для загрузки у бесплатных аккаунтов | 11.25 Pb/d |
| Суммарное потребление для скачивания у бесплатных аккаунтов | 11.25 Pb/d |
| Суммарное потребление для загрузки у платных аккаунтов | 57.75 Pb/d |
| Суммарное потребление для скачивания у платных аккаунтов | 57.75 Pb/d |
| RPS | 2 864 |
| Пиковый RPS | 5728 |




