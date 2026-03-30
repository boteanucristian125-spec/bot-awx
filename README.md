import telebot

TOKEN = "8718202864:AAH0zUNPaOtiK_2Yrcr3dZNzbB-bpzz9TyI"
bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    salut = (
        "Salut! 👋 Sunt asistentul digital AWX.\n\n"
        "Cum te pot ajuta astăzi? Scrie un cuvânt cheie:\n"
        "1️⃣ **Servicii** - ce pot face pentru tine\n"
        "2️⃣ **Preturi** - cât costă serviciile\n"
        "3️⃣ **Portofoliu** - vezi ce am lucrat\n"
        "4️⃣ **Contact** - cum vorbim direct"
    )
    bot.reply_to(message, salut, parse_mode='Markdown')

@bot.message_handler(func=lambda message: True)
def raspuns_clienti(message):
    text = message.text.lower()
    
    if "servicii" in text or "ce faci" in text:
        bot.reply_to(message, "✅ **Servicii oferite:**\n- Instruire pas cu pas\n- Apel audio.")

    elif "pret" in text or "costa" in text or "bani" in text:
        # Creăm meniul de plată unitar
        markup = telebot.types.InlineKeyboardMarkup()
        
        # Buton PayPal
        btn_paypal = telebot.types.InlineKeyboardButton("💳 PayPal (5€)", url="https://www.paypal.me")
        
        # Buton MIA / Card Moldova
        btn_card = telebot.types.InlineKeyboardButton("🇲🇩 Card Moldova / MIA", callback_data="detalii_card")
        
        # Buton Confirmare (pentru a primi fișierul)
        btn_confirm = telebot.types.InlineKeyboardButton("✅ Am plătit! Trimite fișierul", callback_data="confirm_plata")
        
        markup.add(btn_paypal)
        markup.add(btn_card)
        markup.add(btn_confirm)

        msg_text = (
            "💰 **Opțiuni de plată:**\n\n"
            "• **Standard:** 5€\n\n"
            "*Alege metoda de plată de mai jos. După ce ai efectuat transferul, apasă pe butonul verde de confirmare:* "
        )
        bot.send_message(message.chat.id, msg_text, reply_markup=markup, parse_mode='Markdown')

    elif "portofoliu" in text or "lucrari" in text:
        bot.reply_to(message, "📂 **Portofoliul meu:**\nPoți vedea proiectele mele aici: [TikTok](https://www.tiktok.com/@awxteammoney?_r=1&_t=ZS-956PqXuNKld)", parse_mode='Markdown')

    elif "contact" in text or "vorbim" in text:
        bot.reply_to(message, "📩 **Contact:**\nÎmi poți lăsa un mesaj aici: @awxsmerti")

    else:
        bot.reply_to(message, "Nu am înțeles întrebarea. Te rog scrie: **Servicii**, **Preturi** sau **Contact**.")

# Handler pentru Detalii Card / MIA
@bot.callback_query_handler(func=lambda call: call.data == "detalii_card")
def trimite_date_card(call):
    text_plata = (
        "📌 **Detalii Plată Moldova**\n\n"
        "💳 **Număr Card (Moldindconbank):**\n"
        "`5188 9403 5108 7408` (Apasă pe număr pentru copy)\n\n"
        "📱 **Transfer MIA (Număr Telefon):**\n"
        "`+37360263126` (Apasă pe număr pentru copy)\n\n"
        "După ce faci transferul, te rog trimite un screenshot aici @awxsmerti."
    )
    bot.send_message(call.message.chat.id, text_plata, parse_mode='Markdown')
    bot.answer_callback_query(call.id)

# Handler pentru Confirmare Plată (Trimite Fișierul)
@bot.callback_query_handler(func=lambda call: call.data == "confirm_plata")
def trimite_fisier(call):
    bot.answer_callback_query(call.id, "Se procesează cererea...")
    
    # ATENȚIE: Fișierul trebuie să existe în Pyto cu acest nume exact!
    nume_fisier = "Fisier.zip" 
    
    try:
        with open(nume_fisier, "rb") as doc:
            bot.send_document(call.message.chat.id, doc, caption="Mulțumim pentru plată! Iată fișierul tău. ✅")
    except FileNotFoundError:
        bot.send_message(call.message.chat.id, f"❌ Eroare: Fișierul `{nume_fisier}` nu a fost găsit. Contactează @awxsmerti.")
    except Exception as e:
        bot.send_message(call.message.chat.id, "❌ A apărut o eroare la trimitere. Contactează @awxsmerti.")

print("Botul AWX este ONLINE!")
bot.infinity_polling()
