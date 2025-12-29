import telebot
from telebot import types

BOT_TOKEN = "8369945817:AAHY0fz1Uhk30pCN4y58IKm2TEpQWI3BmD4"
FORCE_JOIN = "@M1NXGAMINGHi"

bot = telebot.TeleBot(BOT_TOKEN)

likes = {1003624228790}
liked_users = {7765423734}

def is_member(user_id):
    try:
        status = bot.get_chat_member(FORCE_JOIN, user_id).status
        return status in ["member", "administrator", "creator"]
    except:
        return False

@bot.message_handler(commands=['start'])
def start(message):
    if not is_member(message.from_user.id):
        markup = types.InlineKeyboardMarkup()
        markup.row(
            types.InlineKeyboardButton(
                "🔔 Join Group",
                url="https://t.me/M1NXGAMINGHi"
            )
        )
        bot.send_message(
            message.chat.id,
            "❌ You must join our Telegram group first!",
            reply_markup=markup
        )
        return

    bot.send_message(
        message.chat.id,
        "🔥 Free Fire Like Bot (Demo)\n\n🔢 Send your FF UID:"
    )

# ✅ Works in GROUP + PRIVATE
@bot.message_handler(func=lambda m: m.text and m.text.isdigit())
def uid_handler(message):
    if not is_member(message.from_user.id):
        return

    uid = message.text
    if len(uid) < 6:
        return

    likes.setdefault(uid, 0)
    liked_users.setdefault(uid, set())

    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("👍 LIKE", callback_data=f"like_{uid}"),
        types.InlineKeyboardButton("📊 STATS", callback_data=f"stats_{uid}")
    )

    bot.send_message(
        message.chat.id,
        f"🎮 FF UID: {uid}\n❤️ Likes: {likes[uid]}",
        reply_markup=markup
    )

@bot.callback_query_handler(func=lambda call: call.data.startswith("like_"))
def like_handler(call):
    uid = call.data.split("_")[1]
    user_id = call.from_user.id

    if not is_member(user_id):
        bot.answer_callback_query(call.id, "❌ Join the group first!")
        return

    if user_id in liked_users[uid]:
        bot.answer_callback_query(call.id, "❌ You already liked this UID!")
        return

    liked_users[uid].add(user_id)
    likes[uid] += 1

    markup = types.InlineKeyboardMarkup()
    markup.row(
        types.InlineKeyboardButton("👍 LIKE", callback_data=f"like_{uid}"),
        types.InlineKeyboardButton("📊 STATS", callback_data=f"stats_{uid}")
    )

    bot.edit_message_text(
        f"🎮 FF UID: {uid}\n❤️ Likes: {likes[uid]}",
        call.message.chat.id,
        call.message.message_id,
        reply_markup=markup
    )

    bot.answer_callback_query(call.id, "✅ Like added successfully!")

@bot.callback_query_handler(func=lambda call: call.data.startswith("stats_"))
def stats_handler(call):
    uid = call.data.split("_")[1]
    bot.answer_callback_query(
        call.id,
        f"❤️ Total Likes: {likes.get(uid, 0)}",
        show_alert=True
    )

bot.polling()
