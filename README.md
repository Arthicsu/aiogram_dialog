**Первый код:** создание `текста` и `кнопки`.

```python
from aiogram import Bot, Dispatcher
from aiogram.types import Message
from aiogram.filters import Command

from aiogram.filters.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage

from aiogram_dialog import Dialog, DialogManager, setup_dialogs, StartMode, Window
from aiogram_dialog.widgets.kbd import Button
from aiogram_dialog.widgets.text import Const

storage = MemoryStorage()

BOT_TOKEN = 'Ваш токен'
bot = Bot(token=BOT_TOKEN)

# Определяем класс для группировки состояний
class MyStateG(StatesGroup):
    main = State()  # Определяем состояние "main"

# Создаем основное main_window окно диалога
main_window = Window(
    Const("Привет, мир!"),  # Постоянный текст, который будет отображаться в окне
    Button(Const("Полезная кнопка"), id="nothing"),  # Кнопка с текстом и уникальным id
    state=MyStateG.main,  # Указываем состояние, которое будет связано с этим окном
)

# Создаем диалог, содержащий окно main_window
dialog = Dialog(main_window)  # Объединяем окно в диалог для последующего использования

dp = Dispatcher(storage=storage)
dp.include_router(dialog)
setup_dialogs(dp)

@dp.message(Command("start"))
async def start(message: Message, dialog_manager: DialogManager):
    await dialog_manager.start(MyStateG.main, mode=StartMode.RESET_STACK)


if __name__ == '__main__':
    dp.run_polling(bot)
```

### Второй код к заданию

```python
from aiogram import Bot, Dispatcher
from aiogram.types import Message, CallbackQuery
from aiogram.filters import Command
from aiogram.filters.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage
from aiogram_dialog import Dialog, DialogManager, setup_dialogs, StartMode, Window
from aiogram_dialog.widgets.kbd import Button, Back
from aiogram_dialog.widgets.text import Const, Format

# Хранилище состояний
storage = MemoryStorage()

# Токен бота
BOT_TOKEN = 'Ваш токен'
bot = Bot(token=BOT_TOKEN)

# Определяем класс для группировки состояний
class MyStateG(StatesGroup):
    window1 = State()
    window2 = State()

# Функция получения данных для первого окна
async def window1_get_data(**kwargs):
    return {
        "data": "Какие-то данные из первого окна",
    }

# Функция получения данных для второго окна
async def window2_get_data(**kwargs):
    return {
        "data": "Какие-то данные из второго окна",
    }

# Функция получения данных для диалога
async def dialog_get_data(**kwargs):
    return {
        "name": "Пользователь",
    }

# Обработчик нажатия на кнопку в первом окне
async def button1_clicked(callback: CallbackQuery, button: Button, manager: DialogManager):
    """ Добавить данные в dialog_data и перейти к следующему окну диалога """
    manager.dialog_data['user_input'] = 'Статические данные от пользователя'#можно доработать под динамические данные
    await manager.next()

# Создаем диалог с двумя окнами
dialog = Dialog(
    Window(
        Format("Привет, {name}!"),
        Format("data: {data}"),
        Button(Const("Второе окно"), id="button1", on_click=button1_clicked),
        state=MyStateG.window1,
        getter=window1_get_data,  # Получение данных для первого окна
    ),
    Window(
        Format("Привет, {name}!"),
        Format("data: {data}"),
        Format("User_input: {dialog_data[user_input]}"),
        Back(text=Const("Вернуться к первому окну")),
        state=MyStateG.window2,
        getter=window2_get_data,  # Получение данных для второго окна
    ),
    getter=dialog_get_data  # Получение данных для всего диалога
)

# Создаем диспетчер и настраиваем диалоги
dp = Dispatcher(storage=storage)
dp.include_router(dialog)
setup_dialogs(dp)

@dp.message(Command("start"))
async def start(message: Message, dialog_manager: DialogManager):
    await dialog_manager.start(MyStateG.window1, mode=StartMode.RESET_STACK)

if __name__ == '__main__':
    dp.run_polling(bot)
)
```
