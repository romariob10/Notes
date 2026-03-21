## Материалы
![[API-05_new (1).pdf]]
![[API-06_new (1).pdf]]
![[API-07_справочник.pdf]]
![[API-09_Справочник.pdf]]
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
**bot >>>** Привет, я user
*user >>>* $Туц\,туц\,туц$
**bot >>>** Туц туц туц

---
## Фильтры

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
```~(F.text == 'HeHe')``` то же самое, что и ```F.text != 'HeHe'```

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
Та же кнопочная клавиатура, но есть свои особенности. 

```ReplyKeyboardBuilder``` $->$ ```InlineKeyboardBuilder``` 
```KeyboardButton``` $->$ ```InlineKeyboardButton``` 

Также появляется новое поле ```callback_data```, куда можно положить текст, который вернется при нажатии на кнопку. 

```Python 
@dp.message(Command("start")) 
async def start_message(message): 
	kb = InlineKeyboardBuilder() 
	kb.add(InlineKeyboardButton(text="Нравится", callback_data="+")) 
	kb.add(InlineKeyboardButton(text="Не нравится", callback_data="-")) 
	kb.adjust(2) keyboard = kb.as_markup(resize_keyboard=True) 
	await bot.send_message(message.chat.id, 'текст сообщения', reply_markup=keyboard) 
``` 

### Действия на кнопки 
Коллбеки обрабатываются методом ```@dp.callback_query()```, фильтры можно писать аналогично обработчику $message$. В качестве аргумента приходит объект ```CallbackQuery``` с информацией о коллбэке. 

```Python 
@dp.callback_query(F.data == "+") 
async def callback_plus(callback: CallbackQuery): 
	global haiku_score haiku_score += 1 
	await callback.answer("Отзыв записан") @dp.callback_query(F.data == "-") 
	async def callback_minus(callback: CallbackQuery): 
	global haiku_score haiku_score -= 1 
	await callback.answer("Отзыв записан") 
``` 

## CallbackData 
#### Создадим свой класс для удобной работы с коллбеками 

Мы будем использовать объект, который унаследован от ```CallbackData```. 

```prefix``` — это уникальный идентификатор, который используется для различения различных типов коллбеков. 
```action``` — это параметр, который указывает на конкретное действие, связанное с коллбеком. 

```Python 
class YourCallbackData(CallbackData, prefix="YourPrefix"):
	your_index: int # индекс 
	action: int # 1 или -1 в зависимости от кнопки 
``` 

В примере префикс задается как ```prefix="YourPrefix"```, что означает, что все коллбеки, созданные с использованием этого класса, будут начинаться с "YourPrefix", например, `YourPrefix:1` или `YourPrefix:-1`. 

В примере ```action``` может принимать значения $1$ или $-1$. Это позволяет нам легко обрабатывать различные действия в зависимости от того, какая кнопка была нажата.

##### Пример использования YourCallbackData Чтобы использовать класс `YourCallbackData`, вам нужно создать коллбеки и обработчики. Например: 

```Python 
callback_data_increase = YourCallbackData(your_index=1, action=1) 
callback_data_decrease = YourCallbackData(your_index=1, action=-1) 

@dp.callback_query_handler(YourCallbackData.filter()) 
async def process_callback(callback_query: CallbackQuery, callback_data: dict): 
	index = callback_data['your_index'] 
	action = int(callback_data['action']) 
	
	if action == 1: 
		await callback_query.answer("Увеличиваем значение!") 
	elif action == -1: 
		await callback_query.answer("Уменьшаем значение!") 
``` 

##### Создание инлайн-кнопок с использованием YourCallbackData 

Вы можете создать инлайн-кнопки, которые будут использовать ваши коллбеки: 

```Python
kb.add(InlineKeyboardButton(text="Увеличить", callback_data=YourCallbackData(your_index=index, action=1).pack())) 
kb.add(InlineKeyboardButton(text="Уменьшить", callback_data=YourCallbackData(your_index=index, action=-1).pack())) 
``` 

Теперь, когда пользователь нажимает на кнопки "Увеличить" или "Уменьшить", ваш бот будет обрабатывать соответствующие коллбеки и выполнять нужные действия.

---

## Состояния ботов

### Палитра команд
- `state.set_state()` — устанавливает текущее состояние.
- `state.update_data()` — сохраняет данные.
- `state.get_data()` — извлекает данные.
- `state.clear()` — завершает состояние.
- `StateFilter(None)` — реагирует на пользователей из вне

### Введение

Состояния позволяют управлять последовательностью действий пользователя, сохраняя данные между этапами. Это удобно, например, при создании анкет, опросов или оформления заказов.

### Определение состояний

Состояния управляется с помощью **StatesGroup**, который определяет набор возможных состояний.

```Python
from aiogram.fsm.state import State, StatesGroup

class AnketaStates(StatesGroup):  
    name = State()  
    surname = State()  
    birthday = State()
```

В этом примере создается три состояния: ввод имени, фамилии и года рождения.

### Установка состояния

При запуске анкеты необходимо установить пользователю первое состояние:

```Python
from aiogram.fsm.context import FSMContext

@dp.message(Command("start"))
async def start_command(msg: Message, state: FSMContext):
    await state.set_state(AnketaStates.name)
    await msg.answer("Напишите ваше имя")
```

Здесь команда `/start` переводит пользователя в состояние **AnketaStates.name**.

### Обработчики состояний

Для обработки сообщений в конкретном состоянии используются фильтры состояний.

```Python
@dp.message(AnketaStates.name)
async def process_name(msg: Message, state: FSMContext):
    await state.update_data(first_name=msg.text)
    await state.set_state(AnketaStates.surname)
    await msg.answer("Напишите свою фамилию")
```

`state.update_data(first_name=msg.text)` — сохраняет данные в текущем состоянии.
`state.set_state(AnketaStates.surname)` — переводит пользователя на следующий шаг.

### Завершение состояния

Когда пользователь ввел все данные, их можно извлечь и очистить состояние.

```Python
@dp.message(AnketaStates.birthday)
async def process_birthday(msg: Message, state: FSMContext):
    await state.update_data(birthday=msg.text)
    data = await state.get_data()

    await msg.answer(
        f"Ваши данные:\nИмя: {data['first_name']}\nФамилия: {data['surname']}\nГод рождения: {data['birthday']}"
    )
    
    await state.clear()  # Очистка состояния
```

### Отмена процесса

Для выхода из любого состояния можно использовать команду `/cancel`.

```Python
from aiogram.filters import Command

@dp.message(Command("cancel"))
async def cancel_handler(msg: Message, state: FSMContext):
    await state.clear()
    await msg.answer("Анкета отменена")
```

Команда `/cancel` очищает состояние и возвращает пользователя в обычный режим.

### Фильтр по состоянию

Если бот должен реагировать только на пользователей, находящихся **вне состояний**, используется `StateFilter(None)`.

```Python
from aiogram.filters import StateFilter

@dp.message(StateFilter(None))
async def handle_default(msg: Message):
    await msg.answer("Чтобы пройти анкету, напишите /start")
```

Это позволяет игнорировать случайные сообщения от пользователей, которые не находятся в процессе заполнения анкеты.
