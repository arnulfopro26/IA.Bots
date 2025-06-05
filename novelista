from telegram import Update, ReplyKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, CallbackContext, filters
import requests
import logging

# Diccionario de traducciones
TRANSLATIONS = {
    'es': {
        'welcome': (
            "🌟 ¡Bienvenido a NovelistaBot! 🌟\n\n"
            "🌐 **Selección de idioma:**\n"
            "• Usa /language para cambiar entre Español e Inglés\n\n"
            "🎭 **Crea novelas cortas personalizadas para tus redes sociales.** 🎭\n\n"
            "📜 **Instrucciones:**\n"
            "1️⃣ Escribe los personajes separados por comas.\n"
            "2️⃣ Selecciona la categoría: ficción, no ficción, terror, romance, etc.\n"
            "3️⃣ Define el tono: humorístico, dramático, misterioso, etc.\n"
            "4️⃣ Elige el tamaño de la novela (150 a 500 palabras).\n\n"
            "✍️ Ejemplo: `Juan, María, un perro | terror | misterioso | 300`\n\n"
            "🔄 Usa /language en cualquier momento para cambiar el idioma\n"
            "✨ ¡Deja que la magia de las palabras comience! ✨"
        ),
        'format_error': "⚠️ Formato incorrecto. Por favor, sigue el ejemplo: `Juan, María, un perro | terror | misterioso | 300`",
        'word_count_error': "⚠️ El tamaño debe ser uno de los siguientes valores: 100, 150, 200, 250, 300, 350, 400, 450, 500.",
        'story_prefix': "📖 **Tu novela corta:**\n\n",
        'api_error': "⚠️ Error de conexión con la API. Por favor, intenta más tarde.",
        'unexpected_error': "⚠️ Ocurrió un error inesperado. Por favor, intenta más tarde.",
        'generation_error': "⚠️ No se pudo generar la novela. Intenta nuevamente más tarde.",
        'select_language': "🌐 Selecciona tu idioma / Select your language:",
        'language_changed': "✅ Idioma cambiado a Español"
    },
    'en': {
        'welcome': (
            "🌟 Welcome to NovelistBot! 🌟\n\n"
            "🌐 **Language Selection:**\n"
            "• Use /language to switch between English and Spanish\n\n"
            "🎭 **Create customized short stories for your social media.** 🎭\n\n"
            "📜 **Instructions:**\n"
            "1️⃣ Write the characters separated by commas.\n"
            "2️⃣ Select the category: fiction, non-fiction, horror, romance, etc.\n"
            "3️⃣ Define the tone: humorous, dramatic, mysterious, etc.\n"
            "4️⃣ Choose the story length (150 to 500 words).\n\n"
            "✍️ Example: `John, Mary, a dog | horror | mysterious | 300`\n\n"
            "🔄 Use /language at any time to change the language\n"
            "✨ Let the magic of words begin! ✨"
        ),
        'format_error': "⚠️ Incorrect format. Please follow the example: `John, Mary, a dog | horror | mysterious | 300`",
        'word_count_error': "⚠️ The size must be one of these values: 100, 150, 200, 250, 300, 350, 400, 450, 500.",
        'story_prefix': "📖 **Your short story:**\n\n",
        'api_error': "⚠️ API connection error. Please try again later.",
        'unexpected_error': "⚠️ An unexpected error occurred. Please try again later.",
        'generation_error': "⚠️ Could not generate the story. Please try again later.",
        'select_language': "🌐 Selecciona tu idioma / Select your language:",
        'language_changed': "✅ Language changed to English"
    }
}

# Configuración del bot y OpenRouter
BOT_TOKEN = "7628252959:AAFwWErA6xI5xRHJtgaRB-o-VDHu86C2nFk"
API_KEY = "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"
MODEL_NAME = "qwen/qwen3-235b-a22b:free"

# Configuración del logger
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Agregar al inicio del archivo, después de las importaciones
def get_user_language(context: CallbackContext) -> str:
    if not context.user_data.get('language'):
        context.user_data['language'] = 'es'
    return context.user_data['language']

# Nuevo comando para cambiar el idioma
async def change_language(update: Update, context: CallbackContext) -> None:
    keyboard = [
        ['🇪🇸 Español', '🇬🇧 English']
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, one_time_keyboard=True)
    lang = get_user_language(context)
    await update.message.reply_text(
        TRANSLATIONS[lang]['select_language'],
        reply_markup=reply_markup
    )

# Función para manejar el comando /start
async def start(update: Update, context: CallbackContext) -> None:
    logger.info("Comando /start recibido")
    
    # Mostrar selector de idioma inicial si no hay idioma seleccionado
    if not context.user_data.get('language'):
        keyboard = [
            ['🇪🇸 Español', '🇬🇧 English']
        ]
        reply_markup = ReplyKeyboardMarkup(keyboard, one_time_keyboard=True)
        await update.message.reply_text(
            "🌟 ¡Bienvenido! Welcome! 🌟\n\n"
            "🌐 Por favor, selecciona tu idioma / Please select your language:",
            reply_markup=reply_markup
        )
        return

    # Si ya hay un idioma seleccionado, mostrar el mensaje de bienvenida
    lang = get_user_language(context)
    await update.message.reply_text(TRANSLATIONS[lang]['welcome'])

# Función para generar la novela
async def generate_story(update: Update, context: CallbackContext) -> None:
    logger.info("Generando historia...")
    user_input = update.message.text
    lang = get_user_language(context)

    try:
        parts = user_input.split('|')
        if len(parts) != 4:
            await update.message.reply_text(TRANSLATIONS[lang]['format_error'])
            return

        characters = parts[0].strip()
        category = parts[1].strip()
        tone = parts[2].strip()
        try:
            word_count = int(parts[3].strip())
            if word_count not in [100, 150, 200, 250, 300, 350, 400, 450, 500]:
                raise ValueError
        except ValueError:
            await update.message.reply_text(TRANSLATIONS[lang]['word_count_error'])
            return

        # Llamada a la API de OpenRouter para generar la novela
        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers={
                "Authorization": f"Bearer {API_KEY}",
                "Content-Type": "application/json"
            },
            json={
                "model": MODEL_NAME,
                "messages": [
                    {"role": "system", "content": "Eres un novelista experto en crear historias cortas."},
                    {
                        "role": "user",
                        "content": (
                            f"Escribe una novela corta de {word_count} palabras.\n"
                            f"Personajes: {characters}.\n"
                            f"Categoría: {category}.\n"
                            f"Tono: {tone}."
                        )
                    }
                ]
            }
        )

        response.raise_for_status()
        result = response.json()

        # Validar la respuesta generada
        if 'choices' in result and isinstance(result['choices'], list) and len(result['choices']) > 0:
            story = result['choices'][0]['message']['content']
            await update.message.reply_text(f"{TRANSLATIONS[lang]['story_prefix']}{story}")
        else:
            await update.message.reply_text(TRANSLATIONS[lang]['generation_error'])

    except requests.exceptions.RequestException as e:
        logger.error(f"Error de conexión con la API: {e}")
        await update.message.reply_text(TRANSLATIONS[lang]['api_error'])
    except Exception as e:
        logger.error(f"Error inesperado: {e}")
        await update.message.reply_text(TRANSLATIONS[lang]['unexpected_error'])

# Agregar función para manejar la selección de idioma
async def handle_language_selection(update: Update, context: CallbackContext) -> None:
    text = update.message.text
    if text == '🇪🇸 Español':
        context.user_data['language'] = 'es'
        await update.message.reply_text(
            "✅ Idioma establecido: Español\n\n"
            "🎭 **Crea novelas cortas personalizadas para tus redes sociales.** 🎭\n\n"
            "📜 **Instrucciones:**\n"
            "1️⃣ Escribe los personajes separados por comas.\n"
            "2️⃣ Selecciona la categoría: ficción, no ficción, terror, romance, etc.\n"
            "3️⃣ Define el tono: humorístico, dramático, misterioso, etc.\n"
            "4️⃣ Elige el tamaño de la novela (150 a 500 palabras).\n\n"
            "✍️ Ejemplo: `Juan, María, un perro | terror | misterioso | 300`\n\n"
            "🔄 Usa /language para cambiar el idioma en cualquier momento"
        )
    elif text == '🇬🇧 English':
        context.user_data['language'] = 'en'
        await update.message.reply_text(
            "✅ Language set: English\n\n"
            "🎭 **Create customized short stories for your social media.** 🎭\n\n"
            "📜 **Instructions:**\n"
            "1️⃣ Write the characters separated by commas.\n"
            "2️⃣ Select the category: fiction, non-fiction, horror, romance, etc.\n"
            "3️⃣ Define the tone: humorous, dramatic, mysterious, etc.\n"
            "4️⃣ Choose the story length (150 to 500 words).\n\n"
            "✍️ Example: `John, Mary, a dog | horror | mysterious | 300`\n\n"
            "🔄 Use /language to change the language at any time"
        )

# Función principal para iniciar el bot
def main() -> None:
    application = Application.builder().token(BOT_TOKEN).build()

    # Registrar comandos y manejadores
    application.add_handler(CommandHandler('start', start))
    application.add_handler(CommandHandler('language', change_language))
    application.add_handler(MessageHandler(
        filters.Regex('^(🇪🇸 Español|🇬🇧 English)$'),
        handle_language_selection
    ))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, generate_story))

    # Iniciar el bot
    application.run_polling()

if __name__ == '__main__':
    main()
