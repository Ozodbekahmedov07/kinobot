from aiogram import Bot, Dispatcher, executor, types
from aiogram.contrib.middlewares.logging import LoggingMiddleware
from aiogram.dispatcher.filters import Command
from aiogram.dispatcher import FSMContext
from aiogram.contrib.fsm_storage.memory import MemoryStorage
from aiogram.dispatcher.filters.state import State, StatesGroup

API_TOKEN = '7969192606:AAHpPsoL4p53hgLV2oyLvNVmY32jAGuKX0k'
ADMIN_ID = 6580095286  # Admin Telegram ID  

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot, storage=MemoryStorage())
dp.middleware.setup(LoggingMiddleware())

kino_db = {}  # {"kod": {"nomi": "...", "tavsif": "...", "file_id": "..."}}

class AddMovie(StatesGroup):
    waiting_for_code = State()
    waiting_for_name = State()
    waiting_for_description = State()
    waiting_for_link = State()

@dp.message_handler(Command('start'))
async def start(message: types.Message):
    await message.answer("Salom! Kino kodini kiriting:")

@dp.message_handler(lambda message: message.text.startswith('/add') and message.from_user.id == ADMIN_ID)
async def add_movie(message: types.Message):
    await message.answer("Kino kodi?")
    await AddMovie.waiting_for_code.set()

@dp.message_handler(state=AddMovie.waiting_for_code)
async def add_code(message: types.Message, state: FSMContext):
    await state.update_data(code=message.text)
    await message.answer("Kino nomi?")
    await AddMovie.next()

@dp.message_handler(state=AddMovie.waiting_for_name)
async def add_name(message: types.Message, state: FSMContext):
    await state.update_data(name=message.text)
    await message.answer("Kino tavsifi?")
    await AddMovie.next()

@dp.message_handler(state=AddMovie.waiting_for_description)
async def add_description(message: types.Message, state: FSMContext):
    await state.update_data(description=message.text)
    await message.answer("Kino fayli yoki linki?")
    await AddMovie.next()

@dp.message_handler(state=AddMovie.waiting_for_link, content_types=[types.ContentType.DOCUMENT, types.ContentType.VIDEO, types.ContentType.TEXT])
async def add_file_or_link(message: types.Message, state: FSMContext):
    data = await state.get_data()
    if message.content_type == types.ContentType.DOCUMENT:
        file_id = message.document.file_id
        kino_db[data['code']] = {
            'nomi': data['name'],
            'tavsif': data['description'],
            'file_id': file_id
        }
        await message.answer("Kino muvaffaqiyatli fayl sifatida qo'shildi!")
    elif message.content_type == types.ContentType.VIDEO:
        file_id = message.video.file_id
        kino_db[data['code']] = {
            'nomi': data['name'],
            'tavsif': data['description'],
            'file_id': file_id
        }
        await message.answer("Kino muvaffaqiyatli video sifatida qo'shildi!")
    elif message.content_type == types.ContentType.TEXT:
        link = message.text
        kino_db[data['code']] = {
            'nomi': data['name'],
            'tavsif': data['description'],
            'link': link
        }
        await message.answer("Kino muvaffaqiyatli link sifatida qo'shildi!")
    else:
        await message.answer("Iltimos, hujjat, video yoki link yuboring.")
        return
    await state.finish()

@dp.message_handler(lambda message: message.text.startswith('/ochirsh') and message.from_user.id == ADMIN_ID)
async def delete_movie(message: types.Message):
    code = message.text.split(maxsplit=1)[-1].strip()
    if code in kino_db:
        del kino_db[code]
        await message.answer(f"✅ Kod '{code}' bo'lgan kino muvaffaqiyatli o'chirildi.")
    else:
        await message.answer(f"❌ Kod '{code}' bo'lgan kino topilmadi.")

@dp.message_handler()
async def search_movie(message: types.Message):
    code = message.text
    if code in kino_db:
        kino = kino_db[code]
        caption = f"📽 Kino nomi: {kino['nomi']}\n📝 Tavsif: {kino['tavsif']}\n🆔 Kod: {code}"
        if 'file_id' in kino:
            await message.answer_document(kino['file_id'], caption=caption)
        elif 'link' in kino:
            await message.answer(f"{caption}\n🌐 Link: {kino['link']}")
    else:
        await message.answer("❌ Bunday kod bilan kino topilmadi.")

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
