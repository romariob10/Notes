## Начало работы

## Материалы
![[API-05_new (1).pdf]]
![[API-06_new (1).pdf]]
![[API-07_справочник.pdf]]
## Установка библиотек

``` bash  
pip install aiogram
```

``` bash
pip install asyncio
```

## Первый бот

``` Python
from aiogram import Bot, Dispatcher, F, types # импортируем классы из aiogram 

bot = Bot("сюда пишем токен") # создаем объект бота 
dp = Dispatcher() # создаем объект обработчика сообщений
```

``` Python
#Обработчик ТЕКСТОВЫХ сообщений

@dp.message(F.text) #F.text это фильтр, на то что это текстовое
async def nameOfFunction(message): #Название функци не на что не влияет (нужно для удобства понимания кода)
	await bot.send_message(message.chat.id, message.text) # второй аргумент - строка которую выводит бот, в примере идет "эхо"
```
#### Результат выполнения:
*user >>>* $Привет,\,я\,user$
**bot >>>** $Привет,\,я\,user$
*user >>>* $Туц\,туц\,туц$
**bot >>>** $Туц\,туц\,туц$
---
## Обработка фильтров

### Обработка команд (начинающиеся на /)
```Python
from aiogram.filters import Command
```
Подключаем класс работы с фильтрами

### Фильтры для текста
``` Python
@dp.message(F.text)
async def nameOfFunction(message): 
	await message.send_message("Привет! Как дела?")
```
#### Обратите внимание!
Если под сообщение подходит несколько функций обработчиков, то будет применяться первая из них! Таким образом, лучше частные случае (команды) обрабатывать в начале, а более общие — в конце (когда все остальные обработчики не подошли).

#### Более "деликатная" выборка для текста

Текст сообщения *РАВЕН*   $чему\,должен\,быть\,равен$
```F.text == 'чему должен быть равен'```

Текст сообщения *НЕ РАВЕН* $чему\,не\,должен\,быть\,равен$
```F.text != 'чему должен быть не равет'```

Текст сообщения *СОДЕРЖИТ* слово $что\,содержит$
```F.text.contains('Привет')```

Текст сообщения *СОДЕРЖИТСЯ В МНОЖЕСТВЕ ВАРИАНТОВ*
```F.text.in_({'первыйВариант', 'второйВариант'})```

Текст сообщения *СОДЕРЖИТ В МАЛОМ РЕГИСТРЕ* слово $что\,содержит$.
```F.text.lower().contains('что содержит')```

Текст сообщения *НАЧИНАЕТСЯ* со слова $Что\,тебе\,необходимо$
```F.text.startswith('Что тебе необходимо')```

Текст сообщения *ЗАКАНЧИВАЕТСЯ* словом $Что\,тебе\,необходимо$
```F.text.endswith('Что тебе необходимо')```

Это означает *ИНВЕРТИРОВАНИЕ* результата операции с помощью побитового отрицания ~.
```~(F.text == 'HeHe)``` то же самое, что и ```F.text != 'HeHe```

### Фильтр команды, такие как /help, /start или /info
``` Python
from aiogram.filters import Command
#Какой-то код......
@dp.message(Command("ТвояКомандаБезСлеша"))
```
### Фильтр фото
``` Python
@dp.message(F.photo)
```
### Фильтр сообщения с  видео
``` Python
@dp.message(F.video)
```
### Фильтр сообщения с анимацией (гифки)
``` Python
@dp.message(F.animation)
```
### Фильтр сообщение с отправкой контактных данных (очень полезно для FSM)
``` Python
@dp.message(F.contact)
```
### Фильтр сообщение с файлом (тут может быть и фото, если оно отправлено документом)
``` Python
@dp.message(F.document)
```
### Фильтр сообщения с CallData
``` Python
@dp.message(F.data)
```

---
## Отправка сообщений и медиа (будет дополнятся)

### Отправка ОБЫЧНОГО сообщения
``` await bot.send_message(message.chat.id, message.text)  ```
первый аргумент - в какой чат отправлять
второй аргумент - строка которую выводит бот, в примере идет "эхо"

### Отправка сообщения ОТВЕТА
```await message.reply(message.text)```
единственный аргумент - строка которую выводит бот, в примере идет "эхо"
---
## Виртуальные клавиатуры
Чтобы отобразить клавиатуру, вы должны составить разметку кнопок на экране, по строкам и столбцам. 
Каждая кнопка представлена объектом KeyboardButton. Собираем кнопки в двумерный массив.
``` Python
markup = ReplyKeyboardMarkup(keyboard=keyboard) 
await bot.send_message(message.chat.id, "Привет!", reply_markup=markup)
```

Чтобы очистить клавиатуру используйте объект ReplyKeyboardRemove.
``` Python
await bot.send_message(message.chat.id, "Спасибо!", reply_markup=ReplyKeyboardRemove())
```

#### Пример
``` Python
@dp.message(F.text.in_({"Весна", "Лето", "Осень", "Зима"})) 
async def echo_message(message: Message): 
	await bot.send_message(message.chat.id, "Спасибо!", reply_markup=ReplyKeyboardRemove()) 

@dp.message(F.text) 
async def echo_message(message: Message): 
	keyboard = [ [KeyboardButton(text="Весна"), KeyboardButton(text="Лето")], [KeyboardButton(text="Осень"), KeyboardButton(text="Зима")] ] 
	markup = ReplyKeyboardMarkup(keyboard=keyboard) 
	await bot.send_message(message.chat.id, "Привет!", reply_markup=markup)
```

### ReplyKeyboardBuilder
Если в ответе может быть различное количество кнопок, то красивую верстку получится сделать через построитель клавиатур *ReplyKeyboardBuilder*.

Сначала нужно создать объект ReplyKeyboardBuilder и добавить туда нужные кнопки через add.
``` Python
@dp.message() 
async def handler(message): 
	kb = ReplyKeyboardBuilder() 
	kb.add(KeyboardButton(text="1"))
	kb.add(KeyboardButton(text="2"))
	kb.add(KeyboardButton(text="3"))
```

Теперь устанавливаем размеры строк
`kb.adjust(num)`  - кнопки будут ставиться в num столбцов.
`kb.adjust(num1, num2, num3)` - в 1 строке num1 кнопок, во второй  num2 кнопок, дальше по num3 кнопок в строке.
`kb.adjust(num1, num2, repeat=True)`  repeat это чередование (повтор)

##### Чтобы построить клавиатуру используем функцию `as_markup()`, потом клавиатуру можно вернуть как обычную клавиатуру из прошлого занятия.
``` Python
keyboard = kb.as_markup()
```

По умолчанию кнопки занимают максимальную высоту. Если мы хотим сделать их минимальной высоты, используем аргумент keyboard_resize у функции 
``` Python
kb.as_markup(resize_keyboard=True)
```

## Инлайн-кнопки

