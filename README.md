# 🤖 Telegram Translator Bot

Бот для Telegram, который автоматически переводит текст между русским и португальским языками. Подходит для практики работы с Telegram API, обработкой текста, переводами и использованием переменных окружения.

---

## ?? Стек технологий

- Python 3
- Telegram Bot API (через `python-telegram-bot`)
- Google Translate API (библиотека `googletrans`)
- dotenv для хранения токена
- logging для отладки

---

## 📂 Структура проекта

```
translator-bot/
├── bot.py              # Основной скрипт бота
├── .env                # Файл с переменными окружения
├── requirements.txt    # Зависимости проекта
└── README.md           # Документация
```

---

## 🚀 Запуск бота

```bash
python bot.py
```

---

## 💡 Как пользоваться

- Напишите боту сообщение на русском или португальском языке
- Бот автоматически определит язык и переведёт текст:
  - 🇷🇺 Русский → 🇵🇹 Португальский
  - 🇵🇹 Португальский → 🇷🇺 Русский

Доступные команды:

- `/start` — приветствие
- `/help` — инструкция по использованию

---

## 🧾 Код бота

```python
import logging
import re
from googletrans import Translator
from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
from dotenv import load_dotenv
import os

# Загружаем переменные окружения ДО их использования
load_dotenv()

# Проверяем наличие токена
token = os.getenv("TELEGRAM_TOKEN")
if not token:
    raise ValueError("Токен бота не найден! Проверьте файл .env")

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

translator = Translator()

def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text(
        "Привет! Я бот для перевода текста. Отправь мне текст на русском или португальском языке, "
        "и я переведу его на другой язык.\nНапиши /help для дополнительной информации."
    )

def help_command(update: Update, context: CallbackContext) -> None:
    update.message.reply_text(
        "🤖 *Инструкция по использованию бота:*\n\n"
        "📌 Просто отправь мне текст на русском или португальском языке.\n"
        "📤 Я автоматически определю язык и переведу его на другой:\n"
        "• Русский → Португальский\n"
        "• Португальский → Русский\n\n"
        "🚫 Другие языки пока не поддерживаются.\n"
        "🔁 Команды:\n"
        "/start — Начать\n"
        "/help — Помощь",
        parse_mode='Markdown'
    )

def translate_text(update: Update, context: CallbackContext) -> None:
    try:
        user_text = update.message.text
        user_text = re.sub(r'[^\w\s.,!?а-яА-ЯёЁa-zA-Zà-üÀ-Ü]', '', user_text)

        if len(user_text) > 3000:
            update.message.reply_text("Текст слишком длинный. Максимум — 3000 символов.")
            return

        detected = translator.detect(user_text)
        detected_lang = detected.lang

        if detected_lang == 'ru':
            translated = translator.translate(user_text, src='ru', dest='pt')
            update.message.reply_text(f"Переведено на португальский: {translated.text}")
        elif detected_lang == 'pt':
            translated = translator.translate(user_text, src='pt', dest='ru')
            update.message.reply_text(f"Переведено на русский: {translated.text}")
        else:
            update.message.reply_text("Поддерживаются только русский и португальский языки. Используйте /help для инструкций.")

    except Exception as e:
        logger.error(f"Ошибка: {e}")
        update.message.reply_text("Произошла ошибка при обработке запроса. Попробуйте позже.")

def main() -> None:
    updater = Updater(token)
    dispatcher = updater.dispatcher

    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("help", help_command))
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, translate_text))

    logger.info("Бот запущен...")
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
```

---

## 📦 `requirements.txt`

```txt
python-telegram-bot==13.15
python-dotenv==1.0.1
googletrans==4.0.0-rc1
```

---

## ❗ Ограничения

- Перевод работает **только** между 🇷🇺 русским и 🇵🇹 португальским
- Не более **3000 символов** в одном сообщении
- Возможны задержки или сбои при использовании `googletrans`, так как это неофициальная обёртка

---

## 📜 Лицензия

Проект создан в учебных целях. MIT License.

