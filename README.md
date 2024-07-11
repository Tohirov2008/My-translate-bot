import logging
import os
import speech_recognition as sr
from googletrans import Translator
from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext

# Loggingni sozlash
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levellevel)s - %(message)s', level=logging.INFO
)
logger = logging.getLogger(__name__)

# Start komandasi
def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text('Salom! Menga video yuboring, men uning audiosini tarjima qilaman.')

# Videoni qabul qilish va tarjima qilish funksiyasi
def handle_video(update: Update, context: CallbackContext) -> None:
    video_file = update.message.video.get_file()
    video_file_path = 'video.mp4'
    audio_file_path = 'audio.wav'

    # Videoni yuklab olish
    video_file.download(video_file_path)

    # Videoni audio formatga aylantirish
    os.system(f'ffmpeg -i {video_file_path} -q:a 0 -map a {audio_file_path}')

    # Audio faylni matnga aylantirish
    recognizer = sr.Recognizer()
    translator = Translator()
    with sr.AudioFile(audio_file_path) as source:
        audio_data = recognizer.record(source)
        try:
            text = recognizer.recognize_google(audio_data)
            translated_text = translator.translate(text, dest='uz').text
            update.message.reply_text(translated_text)
        except sr.UnknownValueError:
            update.message.reply_text('Audioni tushunib bo\'lmadi.')
        except sr.RequestError as e:
            update.message.reply_text(f'Xatolik yuz berdi: {e}')

    # Fayllarni o'chirish
    os.remove(video_file_path)
    os.remove(audio_file_path)

def main() -> None:
    # O'z API tokeningizni kiriting
    updater = Updater("YOUR_API_TOKEN")
    dispatcher = updater.dispatcher

    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(MessageHandler(Filters.video, handle_video))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()# My-translate-bot
Tg bot
