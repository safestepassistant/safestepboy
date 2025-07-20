# safestepboy
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import logging
import asyncio
import threading
import requests
import time
from flask import Flask
from telegram import (Update, ReplyKeyboardMarkup, KeyboardButton,
                      InlineKeyboardButton, InlineKeyboardMarkup)
from telegram.ext import (Application, CommandHandler, MessageHandler, filters,
                          CallbackContext, CallbackQueryHandler)
from datetime import datetime

# Configure logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO)
logger = logging.getLogger(__name__)

# ✅ Sening bot tokening (ishlatishga tayyor)
TOKEN = '7793563820:AAFN_t1CXcIm_jBkX6FI3e45Aek81to9uZs'
TRUSTED_CHAT_ID = 7238635484

# 🌐 Har bir til uchun matnlar
messages = {
    'en': {
        'welcome':
        "Hello Uraimjonova Gulnoza, welcome to SafeStep.\n\n"
        "This bot is here to support you in difficult moments.\n\n"
        "You can type:\n"
        "- 'SOS' to send emergency alert\n"
        "- 'I feel unsafe' to get help\n"
        "- 'note: your message' to save a private note\n"
        "- 'proof' to get instructions for sending evidence\n"
        "- Send photos, videos, or voice messages as evidence\n"
        "- Or share your live 📍location using the button below",
        'menu': [
            '📍 Location', '☎️ Share Contact', '📞 Contact', '🆘 SOS', '📤 Proof',
            '🛡 About SafeStep', '⚠️ I feel unsafe', '🌐 Website'
        ],
        'about':
        "SafeStep is a trusted system to store and send emergency data securely.",
        'proof_info':
        "Please send your photo/video/voice as proof. It will auto-delete in 5 seconds and alert the trusted contact.",
        'unsafe_info':
        "Don't worry. Tap 'SOS' or send your location. Our trusted contact will come to help.",
        'contact_info':
        "You can reach out at:\n📧 azimovasarvinoz2007@gmail.com\n💬 @Azimovasarvinoz2007",
        'website':
        "🌐 Visit: https://safestep-site.vercel.app"
    },
    'uz': {
        'welcome':
        "Salom Uraimjonova Gulnoza, SafeStep botiga xush kelibsiz.\n\n"
        "Ushbu bot sizni qiyin paytlarda qo'llab-quvvatlash uchun yaratilgan.\n\n"
        "Siz yozishingiz mumkin:\n"
        "- 'SOS' xavfli holatda yordam so'rash uchun\n"
        "- 'I feel unsafe' xavotirda ekanligingizni bildiradi\n"
        "- 'note: sizning xabaringiz' maxfiy eslatma sifatida saqlanadi\n"
        "- 'proof' dalillar yuborish haqida ko'rsatma\n"
        "- Rasm, video, ovoz yuboring dalil sifatida\n"
        "- Yoki pastdagi tugmadan 📍 joylashuvingizni ulashing",
        'menu': [
            '📍 Joylashuv', '☎️ Kontaktni ulashish', '📞 Kontakt', '🆘 SOS',
            '📤 Dalil', '🛡 SafeStep haqida',
            '⚠️ O\'zimni xavfsiz his qilmayapman', '🌐 Vebsayt'
        ],
        'about':
        "SafeStep bu ishonchli tizim bo'lib, barcha ma'lumotlaringizni xavfsiz saqlaydi va kerakli joyga yuboradi.",
        'proof_info':
        "Dalil sifatida rasm, video, ovoz yuboring. U 5 soniyada o'chadi va ishonchli kontaktga yuboriladi.",
        'unsafe_info':
        "Xavotir olmang. 'SOS' ni yuboring yoki joylashuvingizni ulashing. Yordamga chiqiladi.",
        'contact_info':
        "Bog'lanish uchun:\n📧 azimovasarvinoz2007@gmail.com\n💬 @Azimovasarvinoz2007",
        'website':
        "🌐 Tashrif: https://safestep-site.vercel.app"
    },
    'ru': {
        'welcome':
        "Здравствуйте, Ураимджонова Гульноза, добро пожаловать в SafeStep.\n\n"
        "Этот бот поддержит вас в трудные моменты.\n\n"
        "Вы можете:\n"
        "- Написать 'SOS' для вызова помощи\n"
        "- 'I feel unsafe' если чувствуете опасность\n"
        "- 'note: сообщение' сохранить как личную заметку\n"
        "- 'proof' получить инструкции по отправке доказательств\n"
        "- Отправьте фото, видео или голос как доказательства\n"
        "- Или поделитесь местоположением 📍 с помощью кнопки ниже",
        'menu': [
            '📍 Локация', '☎️ Поделиться контактом', '📞 Контакт', '🆘 SOS',
            '📤 Доказательства', '🛡 О SafeStep', '⚠️ Я чувствую опасность',
            '🌐 Вебсайт'
        ],
        'about':
        "SafeStep — это система, которая надежно хранит и передает экстренные данные.",
        'proof_info':
        "Отправьте фото/видео/аудио как доказательство. Оно будет удалено через 5 сек и передано доверенному контакту.",
        'unsafe_info':
        "Не волнуйтесь. Нажмите 'SOS' или отправьте локацию. Помощь уже в пути.",
        'contact_info':
        "Свяжитесь с нами:\n📧 azimovasarvinoz2007@gmail.com\n💬 @Azimovasarvinoz2007",
        'website':
        "🌐 Перейти: https://safestep-site.vercel.app"
    }
}

user_languages = {}

# Flask app for keepalive
app = Flask(__name__)


@app.route('/')
def home():
    return "SafeStep Bot is running! 🛡️"


@app.route('/health')
def health():
    return {
        "status": "healthy",
        "bot": "SafeStep",
        "timestamp": datetime.now().isoformat()
    }


@app.route('/ping')
def ping():
    return {"pong": True, "time": datetime.now().isoformat()}


def run_flask():
    app.run(host='0.0.0.0', port=8080, debug=False)


def keepalive_ping():
    """Send periodic pings to keep the bot alive"""
    while True:
        try:
            time.sleep(300)  # Wait 5 minutes
            requests.get('http://127.0.0.1:8080/ping', timeout=10)
            logger.info("Keepalive ping sent")
        except Exception as e:
            logger.warning(f"Keepalive ping failed: {e}")
            time.sleep(60)  # Wait 1 minute on error


async def start(update: Update, context: CallbackContext):
    try:
        keyboard = [[InlineKeyboardButton("English", callback_data='en')],
                    [InlineKeyboardButton("O'zbek tili", callback_data='uz')],
                    [InlineKeyboardButton("Русский язык", callback_data='ru')]]
        reply_markup = InlineKeyboardMarkup(keyboard)
        await update.message.reply_text(
            "Please choose your language / Tilni tanlang:",
            reply_markup=reply_markup)
    except Exception as e:
        logger.error(f"Error in start command: {e}")
        await update.message.reply_text(
            "Sorry, an error occurred. Please try again.")


async def language_selected(update: Update, context: CallbackContext):
    try:
        query = update.callback_query
        user_id = query.from_user.id
        lang = query.data
        user_languages[user_id] = lang
        await query.answer()
        await query.edit_message_text(messages[lang]['welcome'])
        await show_menu(query, lang)
    except Exception as e:
        logger.error(f"Error in language selection: {e}")
        await query.answer("Sorry, an error occurred.")
        await query.message.reply_text("Please try again with /start")


async def show_menu(source, lang):
    try:
        keyboard = [[
            KeyboardButton(text=messages[lang]['menu'][0],
                           request_location=True)
        ],
                    [
                        KeyboardButton(text=messages[lang]['menu'][1],
                                       request_contact=True)
                    ], [messages[lang]['menu'][2], messages[lang]['menu'][3]],
                    [messages[lang]['menu'][4], messages[lang]['menu'][5]],
                    [messages[lang]['menu'][6], messages[lang]['menu'][7]]]
        markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
        await source.message.reply_text("📋 Menu:", reply_markup=markup)
    except Exception as e:
        logger.error(f"Error showing menu: {e}")
        await source.message.reply_text(
            "Error displaying menu. Please try /start again.")


async def handle_message(update: Update, context: CallbackContext):
    user_id = update.message.from_user.id
    lang = user_languages.get(user_id, 'en')
    text = update.message.text.lower()

    if 'sos' in text:
        user = update.message.from_user
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        # Store SOS info temporarily
        context.user_data['pending_sos'] = {
            'user': user,
            'timestamp': timestamp,
            'type': 'text_sos'
        }

        # Request location
        location_keyboard = [[
            KeyboardButton("📍 Share My Location", request_location=True)
        ]]
        location_markup = ReplyKeyboardMarkup(location_keyboard,
                                              resize_keyboard=True,
                                              one_time_keyboard=True)

        await update.message.reply_text(
            "🚨 SOS received! Please share your location immediately so we can send help:",
            reply_markup=location_markup)

    elif 'proof' in text:
        await update.message.reply_text(messages[lang]['proof_info'])

    elif 'i feel unsafe' in text:
        await update.message.reply_text(messages[lang]['unsafe_info'])

    elif 'note:' in text:
        note = text.replace('note:', '', 1).strip()
        await context.bot.send_message(
            chat_id=TRUSTED_CHAT_ID,
            text=f"📝 Note from {update.message.from_user.full_name}:\n{note}")
        await update.message.reply_text("📝 Note saved.")

    elif update.message.text == messages[lang]['menu'][2]:  # Contact
        contact_msg = "📞 You can contact with trusted person:\n\n"
        contact_msg += "📧 Email: azimovasarvinoz2007@gmail.com\n"
        contact_msg += "💬 Telegram: @Azimovasarvinoz2007"
        await update.message.reply_text(contact_msg)

    elif update.message.text == messages[lang]['menu'][5]:  # About SafeStep
        about_msg = "🛡️ About SafeStep:\n\n"
        about_msg += "SafeStep is a trustful system which helps users to save proofs securely. "
        about_msg += "Our system helps users as soon as we get a signal from them. "
        about_msg += "All your emergency data is stored safely and accessible only to trusted contacts when you need help."
        await update.message.reply_text(about_msg)

    elif update.message.text == messages[lang]['menu'][7]:  # Website
        await update.message.reply_text(
            "🌐 Visit our website: https://safestep-site.vercel.app/")

    elif update.message.text == messages[lang]['menu'][4]:
        await update.message.reply_text(messages[lang]['proof_info'])

    elif update.message.text == messages[lang]['menu'][6]:
        await update.message.reply_text(messages[lang]['unsafe_info'])

    elif update.message.text == messages[lang]['menu'][3]:  # SOS button
        user = update.message.from_user
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        # Store SOS info temporarily
        context.user_data['pending_sos'] = {
            'user': user,
            'timestamp': timestamp,
            'type': 'button_sos'
        }

        # Request location
        location_keyboard = [[
            KeyboardButton("📍 Share My Location", request_location=True)
        ]]
        location_markup = ReplyKeyboardMarkup(location_keyboard,
                                              resize_keyboard=True,
                                              one_time_keyboard=True)

        await update.message.reply_text(
            "🚨 Emergency SOS activated! Please share your location immediately:",
            reply_markup=location_markup)


async def handle_location(update: Update, context: CallbackContext):
    loc = update.message.location
    user = update.message.from_user
    user_id = update.message.from_user.id
    lang = user_languages.get(user_id, 'en')

    # Check if this is in response to SOS or proof request
    if context.user_data.get('pending_sos'):
        sos_data = context.user_data['pending_sos']

        # Create comprehensive SOS alert with location
        alert = f"🚨 EMERGENCY SOS ALERT 🚨\n\n"
        alert += f"👤 Full Name: {user.full_name}\n"
        alert += f"📱 Username: @{user.username if user.username else 'No username'}\n"
        alert += f"🆔 User ID: {user.id}\n"
        alert += f"📅 Date & Time: {sos_data['timestamp']}\n"
        alert += f"🔗 User Link: tg://user?id={user.id}\n"
        alert += f"📍 Location: https://maps.google.com/?q={loc.latitude},{loc.longitude}\n"
        alert += f"🗺️ Coordinates: {loc.latitude}, {loc.longitude}\n\n"
        alert += f"⚠️ IMMEDIATE ASSISTANCE REQUIRED!"

        await context.bot.send_message(chat_id=TRUSTED_CHAT_ID, text=alert)

        # Clear the pending SOS
        del context.user_data['pending_sos']

        # Show main menu again
        await show_menu(update, lang)
        await update.message.reply_text(
            "✅ Emergency alert with location sent to trusted contact!")

    elif context.user_data.get('pending_proof'):
        proof_data = context.user_data['pending_proof']

        # Create comprehensive proof report with location
        report = f"📤 PROOF EVIDENCE WITH LOCATION 📤\n\n"
        report += f"👤 Full Name: {user.full_name}\n"
        report += f"📱 Username: @{user.username if user.username else 'No username'}\n"
        report += f"🆔 User ID: {user.id}\n"
        report += f"📅 Date & Time: {proof_data['timestamp']}\n"
        report += f"🔗 User Link: tg://user?id={user.id}\n"
        report += f"📋 Media Type: {proof_data['media_type']}\n"
        report += f"📍 Location: https://maps.google.com/?q={loc.latitude},{loc.longitude}\n"
        report += f"🗺️ Coordinates: {loc.latitude}, {loc.longitude}\n\n"
        report += f"📎 Evidence attached above this message"

        await context.bot.send_message(chat_id=TRUSTED_CHAT_ID, text=report)

        # Clear the pending proof
        del context.user_data['pending_proof']

        # Show main menu again
        await show_menu(update, lang)
        await update.message.reply_text(
            "✅ Proof evidence with location sent to trusted contact!")

    else:
        # Regular location sharing
        await context.bot.send_message(
            chat_id=TRUSTED_CHAT_ID,
            text=
            f"📍 Location from {user.full_name} (@{user.username}):\nhttps://maps.google.com/?q={loc.latitude},{loc.longitude}"
        )
        await update.message.reply_text("📍 Location sent.")


async def handle_contact(update: Update, context: CallbackContext):
    contact = update.message.contact
    user = update.message.from_user
    await context.bot.send_message(
        chat_id=TRUSTED_CHAT_ID,
        text=
        f"📞 Contact from {user.full_name}:\nName: {contact.first_name}\nPhone: {contact.phone_number}"
    )
    await update.message.reply_text("📞 Contact shared.")


async def handle_media(update: Update, context: CallbackContext):
    user = update.message.from_user
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    file = None
    media_type = ""

    if update.message.photo:
        file = update.message.photo[-1]
        media_type = "Photo"
    elif update.message.video:
        file = update.message.video
        media_type = "Video"
    elif update.message.voice:
        file = update.message.voice
        media_type = "Voice message"

    if file:
        # Forward the media to trusted account first
        await update.message.forward(chat_id=TRUSTED_CHAT_ID)

        # Store proof info temporarily
        context.user_data['pending_proof'] = {
            'user': user,
            'timestamp': timestamp,
            'media_type': media_type
        }

        # Request location
        location_keyboard = [[
            KeyboardButton("📍 Share My Location", request_location=True)
        ]]
        location_markup = ReplyKeyboardMarkup(location_keyboard,
                                              resize_keyboard=True,
                                              one_time_keyboard=True)

        # Confirm to user and request location
        confirmation = await update.message.reply_text(
            f"✅ {media_type} evidence received! Please share your location to complete the report:",
            reply_markup=location_markup)

        # Auto-delete the original media after 5 seconds
        async def delete_proof():
            await asyncio.sleep(5)
            try:
                await context.bot.delete_message(
                    chat_id=update.message.chat_id,
                    message_id=update.message.message_id)
            except Exception as e:
                logger.warning(f"Could not delete proof message: {e}")

        # Run deletion in background
        asyncio.create_task(delete_proof())


def main():
    try:
        # Start Flask server in background thread
        flask_thread = threading.Thread(target=run_flask, daemon=True)
        flask_thread.start()
        logger.info("Flask keepalive server started on port 8080")

        # Start keepalive ping thread
        ping_thread = threading.Thread(target=keepalive_ping, daemon=True)
        ping_thread.start()
        logger.info("Keepalive ping thread started")

        application = Application.builder().token(TOKEN).build()

        application.add_handler(CommandHandler("start", start))
        application.add_handler(CallbackQueryHandler(language_selected))
        application.add_handler(
            MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
        application.add_handler(
            MessageHandler(filters.LOCATION, handle_location))
        application.add_handler(MessageHandler(filters.CONTACT,
                                               handle_contact))
        application.add_handler(
            MessageHandler(filters.PHOTO | filters.VIDEO | filters.VOICE,
                           handle_media))

        logger.info("SafeStep bot started successfully")
        application.run_polling()
    except Exception as e:
        logger.error(f"Error starting bot: {e}")
        print(f"Failed to start bot: {e}")


if __name__ == '__main__':
    main()
