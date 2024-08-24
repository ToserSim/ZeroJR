# ZeroJR
Дискорд бот для сохранения структуры вашего сервера в файле формата **.pkl**

**На данный момент сохраняются:**
- Название сервера
- Категории
- Голосовые и текстовые каналы (**только** в категориях!)
- Ветки в каналах с полным списком пользователей
- Вся история текстовых сообщений
- Все эмодзи сервера

**НЕ подлежат сохранению:**
- Изображения (во вложениях, **ссылки** будут сохранены)
- Роли
- Любые настройки прав доступа (исключение - флаг **nsfw**)
- Голосовые сообщения



## Восстановление истории сообщений
Сохранённые текстовые сообщения отправляются в каналы и ветки через вебхук с никнеймом и аватаркой оригинального автора сообщения. В дискорде установлено ограничение для каждого канала по скорости отправки вебхуков (~ 30 сообщений в минуту. https://stackoverflow.com/questions/59117210/discord-webhook-rate-limits), поэтому восстановление истории собщений занимает некоторое время. Будьте готовы 30 минут наблюдать за появлением вашего сервера. Проблема частично решается объединением серии сообщений от одного автора в одно большое (до лимита по символам).



## Инструкция по запуску бота:

1. Добавить файл .env с содержимым вида "TOKEN = {Ваш токен}"
2. Установить зависимости
3. Запустить main.py



## Функции бота:

1. Дампить сервер в .pkl файл
2. Выгружать дамп на существующий Дискорд сервер из .pkl файла
3. Очищать ветки/каналы/категории/эмодзи
4. Выводить забавные сообщения в консоль во время работы



## Консольные команды:

1. **create** {id сервера}
2. **load** {id сервера} {файл с дампом}
3. **clear** {id сервера} all/threads/channels/categories/emojis



## Для безумца, который решит зайти в код:

Бот релизован на нескольких библиотеках, основные функции возможны благодаря:
- discord.py + документация pycorde (https://docs.pycord.dev/en/stable/)
- pickle (https://docs.python.org/3/library/pickle.html)
- dpyConsole (https://github.com/Mihitoko/discord.py-Console?ysclid=m07axn47qg974980974)
- Rich (https://github.com/Textualize/rich?ysclid=m07b3crzoa137533031) - используется для отладки.


Состоит из трёх основных частей:

- **main.py** - CLI

- **builder.py** - Представляет из себя набор классов, каждый из которых отвечает за создание на сервере соответствующих объектов (веток, каналов, категорий и т.д.).

- **record.py** - Главная задача - создание объектов для записи в файл. Стандартные классы Discord создают циклические отссылки друг на друга, а Pickle с ними не дружит и зависает.
Среди методов у объектов для записи в файл наиболее непонятным выглядит *avisitor()*. Одна из его задач - двигаться вниз по дереву (от категорий - в каналы, от каналов - в ветки) и взаимодействовать с объектами, требующими асинхронность. Пример из комментария в коде: "Это нужно для того, чтобы заполнить массив thread.members, вызвав *thread.fetch_members()*, которому нужен *await*". Указанная конструкция позволила добавлять в ветку пользователей, имеющих к ней отношение при сохранении этой ветки в файл.
*as_dict()* содержит информацию о полях, которые требуется передать в *edit()*. Пример - имя Дискорд сервера.