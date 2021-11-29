# Проектирование высоконагруженных систем. Discord.

# 1. Тема и целевая аудитория

## Тип сервиса

Discord - популярный мессенджер с поддержкой VoIP и видеоконференций.

## Функционал MVP

1. Регистрация
2. Авторизация
3. Создание сервера
    * Создание текстовых каналов (чатов) сервера
    * Создание голосовых каналов (лобби) сервера
4. Отправка сообщений в чат сервера (текст + 1 прикреплённый файл)
    * Максмимальный размер сообщения - 400 символов
    * Максимальный размер прикреплённого файла - 8 МБ
5. Голосовое общение в голосовом канале сервера
6. Трансляция экрана со звуком/веб-камеры
6. Личные сообщения

## Целевая аудитория
### Размер 

Источник: https://www.businessofapps.com/data/discord-statistics/
* 300 млн. зарегистрированных пользователей
* 140 млн. активных в течение месяца пользователей 
* 14 млн. активных в течение дня пользователей
* 10 млн. - пик онлайна

### Расположение

Не удалось найти точные данные, но по [количеству переходов](https://www.similarweb.com/website/discord.com/#overview) на сайт Discord, можно сделать вывод, что 30% пользователей приходится на Северную Америку, а остальные пользователи равномерно распределены по всему миру:
1. Северная Америка (30%)
2. Южная Америка (23%)
3. Европа (23%)
4. Азия (23%)

# 2. Расчет нагрузки

## Продуктовые метрики

### Месячная аудитория

**140 млн.**

### Дневная аудитория

**14 млн.**

### Средний размер хранилища пользователя

[963 млн. сообщений в день](https://influencermarketinghub.com/discord-stats/)
=> один пользователь отправляет 963 / 14 = **69 сообщений в день** 

Из личного опыта, размер сообщений колеблется от 29 символов (обычное сообщение, наиболее часто) до 400 символов (цитата, фрагмент кода - редко). Будем считать средний размер сообщения равным 100 символам, пусть 1 символ занимает 2 байта => **200 байт на одно сообщение**

Из личного опыта, наиболее распространённый тип отправляемых файлов - изображения. Обычно, пользователь отправляет 2 изображения/файла в день. Средний размер изображения - 0.5 МБ. Очень редко отправляются и файлы больших размеров (до 8 МБ): гифки, видео, программы. Будем считать, что изображения отправляются в 95% случаев, а файлы размером 5 МБ в 5% случаев. Тогда средний размер файла: 0.5 * 0.95 + 5 * 0.05 = **0.725 МБ**

Большую популярность дискорд получил в [2018 году](https://www.businessofapps.com/data/discord-statistics/) 
=> средний возраст аккаунта - 3 года.

Аватарка весит в среднем 1 МБ.

Аватарка: 1 МБ

Сообщения: (69 * 200 * 365 * 3) / (1024^2) = 14.5 МБ

Файлы: (2 * 0.725 * 1024^2 * 365 * 3) / (1024^2) = 1587.75 МБ

Итого: 1 + 14.5 + 1587.7 = **1603.25 МБ** на 1 пользователя

### Среднее количество действий пользователя по типам в день

Обычно, пользователи запускают дискорд 2 раза в день каждый день => 2 авторизации в день. С учётом смены лобби, и нескольких эпизодов общения каждый пользователь заходит в лобби 3 раза в день. Очень часто в этом лобби, начинается несколько трансляций (~2), к которой пользователь присоединяется.

Сервера и каналы создаются очень редко (~раз в полгода) => можно не учитывать.

Будем считать, что пользователь читает сообщения и просматривает файлы в 5 раз чаще, чем отправляет.

| Действие                   | Кол-во в день      |
| :---                       |                ---:|
| Авторизация                | 2                  |
| Отправка сообщения         | 69                 |
| Отправка файла             | 2                  |
| Просмотр сообщений         | 345                |
| Просмотр файлов            | 10                 |
| Вход в голосовой канал     | 3                  |
| Присоединение к трансляции | 6                  |
| Выход из голосового канала | 3                  |

## Технические метрики

### Размер хранения в разбивке по типам данных (в Тб)

Будем учитывать только активных в течение месяца пользователей (и всех пользователей для аватарки). Подсчитаем используя данные на 1 пользователя, приведённые выше:
* Аватарки: (1 * 300000000) / (1024^2) = 286 ТБ
* Сообщения: (14.5 МБ * 140000000) / (1024^2) = 1936 ТБ
* Файлы: (1587.75 МБ * 140000000) / (1024^2) = 211988 ТБ
* Итого: 286 + 1936 + 211988 = ~**214 210 ТБ** на хранение существующих данных с момента запуска Discord (2017 год)

### Сетевой трафик

#### Пиковое потребление в теченнии суток (в Гбит/с)

Чаще всего Discord используется в вечерние часы (19:00 - 23:59) для общения/игр с друзьями после работы/учебы. Т. е. пиковое потребление приходится на 5 вечерних часов каждый день.

**Установка аватарок новыми пользователями (входящий траффик)**

Каждый год кол-во зарегистрированных пользователей увеличивается [на 50 млн.](https://influencermarketinghub.com/discord-stats/) => 50 / 365 ~ 140 тыс. новых пользователей в день, 1 МБ - средний размер аватарки

* 140000 * 1 / 1024 = **136.72 ГБайт/сутки**
* 136.72 * 8 / (5 * 60 * 60) = **0.06 Гбит/с**

**Отправка сообщений (входящий траффик):**

* 963 млн. сообщений/день (200 байт/сообщение)
* 963000000 * 200 / 1024^3 = **179.37 ГБайт/сутки**
* 179.37 * 8 / (5 * 60 * 60) = **0.08 Гбит/c**

**Отправка файлов (входящий траффик):**

Каждый пользователь отправляет 2 файла в день (см. таблицу с RPS), ежедневно 14 млн. активных пользователей, средний размер файла 0.725 МБ

* 2 * 14 млн. = 28 млн. файлов/день
* 28000000 * 0.725 / 1024 = **19824.22 ГБайт/сутки**
* 19824.22  * 8 / (5 * 60 * 60) = **8.8 Гбит/c** 

**Чтение сообщений и просмотр файлов (исходящий траффик):**

Выше мы предположили, что пользователь просматривает сообщения и файлы в 5 раз чаще, чем отправляет.
Используя траффик на отправку, получим траффик на чтение:

Сообщения:
* 179.37 * 5 = **896.85 ГБайт/сутки**
* 0.08 * 5 = **0.04 Гбит/с**

Файлы:
* 19824.22 * 5 = **99121.1 ГБайт/сутки**
* 8.8 * 5 = **44 Гбит/c**

**Голосовое общение и трансляции (входящий и исходящий траффик)**

[4 млрд. мин./день голосового общения](https://influencermarketinghub.com/discord-stats/)

Голосовой поток от каждого пользователя [объединяется в один на сервере и раздаётся всем остальным участникам](https://blog.discord.com/how-discord-handles-two-and-half-million-concurrent-voice-users-using-webrtc-ce01c3187429). Т.е. у каждого пользователя при голосовом общении 1 входящий поток с сервера и 1 исходящий поток на сервер.

64 Кб/c - среднее качество аудиопотока (стандартная настройка лобби)

Стандартные настройки трансляции:
* Видео: 720p, 3 bytes/px, 30 fps => 1280 * 720 * 3 * 30 / 8 / 1024 = 10125 Кб/с
* Аудио: 64 Кб/с

*Из личного опыта:*
* в лобби обычно общаются 4 человека
* трансляции обычно длятся ~1/4 времени общения 
* трансляцию обычно смотрят 2 человека

Рассчитаем исходящий аудио-поток:
* 4 человека * 4 млрд. = 16 млрд. мин. аудиопотока в день
* 16000000000 * 60 * 64 / 8 / 1024^2 = **7324218.75 ГБайт/сутки**
* 7324218.75 * 8 / (5 * 60 * 60) = **3255.2 Гбит/c**

Аналогично, входящий аудио-поток:
* **7324218.75 ГБайт/сутки** и **3255.2 Гбит/c**

Рассчитаем входящий поток трансляции:
* 4 * 1/4 = 1 млрд. мин. трансляций в день.
* (10125 + 64) * 1000000000 * 60 / 8 / 1024^2 = **72877407.07 ГБайт/сутки**
* 72877407.07 * 8 / (5 * 60 * 60) = **32390 Гбит/с**

Аналогично, исходящие потоки трансляции (2 зрителя):
* 72877407.07 * 2 = **145754814.14 ГБайт/сутки**
* 32390 * 2 = **64780 Гбит/с**

#### Таблица входящей нагрузки
| Нагрузка                         | Пиковая (Гбит/с)                  | Суточная (ГБайт/сутки)         |
| :---                             | :----:                                |                     ---:|
| Аватарки новых пользователей | 0.06 | 136.72 |
| Отправка сообщений | 0.08 |179.37|
| Отправка файлов | 8.8 | 19824.22 |
| Передача голоса | 3255.2 | 7324218.75 |
| Передача трансляции | 32390 | 72877407.07 |

#### Таблица исходящей нагрузки

| Нагрузка                         | Пиковая (Гбит/с) | Суточная (ГБайт/сутки) |
| :---                             | :----:           |                    ---:|
| Просмотр сообщений               | 0.04             | 896.85                 |
| Просмотр файлов                  | 44               | 99121.1                |
| Получение голоса                 | 3255.2           | 7324218.75             |
| Получение трансляции             | 64780            | 145754814.14           |


#### RPS в разбивке по типам запросов

Умножаем значение из таблицы с действиями пользователей на кол-во активных пользователей в день и делим на 24 * 60 * 60

Суммарное кол-во действий пользователя в день:
2 + 69 + 2 + 345 + 10 + 3 + 6 + 3 = **440**

Общее кол-во активных пользователей в день - 14 млн.

Total RPS:
(440 * 14000000) / (24 * 60 * 60) = **71296 RPS**

В расчёте на каждое действие:

| Действие                   | RPS                  |
| :---                       |                  ---:|
| Отправка сообщения         | 11180                |
| Просмотр сообщений         | 55902                |
| Просмотр файлов            | 1620                 |
| Присоединение к трансляции | 972                  |
| Прочее\*                   |  1620                |

Прочее\* - авторизация, отправка файлов, вход и выход из голосового канала
= 324 + 324 + 486 + 486 = 1620
