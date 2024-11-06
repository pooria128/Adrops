import telebot
from telebot import types
import random

TOKEN = "7858423682:AAFWG-R9H9lhgzh7Kkc5utQGdCcmq6VczMU"  # جایگزینی با توکن خود
bot = telebot.TeleBot(TOKEN)

# دیکشنری‌های مربوط به کاربران
user_balances = {}
user_clicks = {}
user_multiplier = {}
user_upgrade_cost = {}
user_in_bet = {}
user_ids = set()

ADMIN_ID = 6639761452

@bot.message_handler(commands=['start'])
def start_message(message):
    user_id = message.chat.id
    if user_id not in user_balances:
        user_balances[user_id] = 0
        user_clicks[user_id] = 0
        user_multiplier[user_id] = 1
        user_upgrade_cost[user_id] = 1000
        user_in_bet[user_id] = False
        user_ids.add(user_id)

    keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
    balance_button = types.KeyboardButton("موجودی حساب شما")
    work_button = types.KeyboardButton("کار کردن")
    transfer_button = types.KeyboardButton("انتقال پول به کاربران")
    user_id_button = types.KeyboardButton("شناسه کاربری من")
    upgrade_button = types.KeyboardButton("افزایش سکه با هر کلیک")
    bet_button = types.KeyboardButton("شرطبندی")
    about_button = types.KeyboardButton("درباره ربات")

    keyboard.add(balance_button, work_button, transfer_button, user_id_button, upgrade_button, bet_button, about_button)
    bot.send_message(message.chat.id, "🎉 به ربات ایردراپ خوش آمدید! 🎉", reply_markup=keyboard)

@bot.message_handler(func=lambda message: message.text == "انتقال پول به کاربران")
def transfer_coins(message):
    help_text = (
        "برای انتقال پول به کاربر، لطفاً از دستور زیر استفاده کنید:\n"
        "/pay [شناسه کاربری] [مقدار پول]\n\n"
        "مثال: /pay 123456789 50\n\n"
        "این دستور به شما امکان می‌دهد تا مقدار مشخصی از سکه‌های خود را به کاربر دیگری منتقل کنید."
    )
    bot.send_message(message.chat.id, help_text)

@bot.message_handler(commands=['pay'])
def process_pay(message):
    try:
        _, recipient_id_str, amount_str = message.text.split()
        recipient_id = int(recipient_id_str)
        amount = int(amount_str)
    except (ValueError, IndexError):
        bot.send_message(message.chat.id, "فرمت نادرست است. لطفاً از فرمت زیر استفاده کنید:\n/pay [شناسه کاربری] [مقدار پول]")
        return

    user_id = message.chat.id

    if user_balances.get(user_id, 0) < amount:
        bot.send_message(user_id, "موجودی کافی نیست.")
        return

    if recipient_id not in user_balances:
        bot.send_message(user_id, "کاربر مقصد وجود ندارد.")
        return

    user_balances[user_id] -= amount
    user_balances[recipient_id] += amount
    bot.send_message(user_id, f"{amount} danscoin به کاربر {recipient_id} انتقال داده شد.")
    bot.send_message(recipient_id, f"{amount} danscoin از کاربر {user_id} دریافت کردید.")

@bot.message_handler(func=lambda message: message.text == "موجودی حساب شما")
def check_balance(message):
    user_id = message.chat.id
    balance = user_balances.get(user_id, 0)
    bot.send_message(user_id, f"موجودی حساب شما: {balance} danscoin")

@bot.message_handler(func=lambda message: message.text == "افزایش سکه با هر کلیک")
def upgrade_multiplier(message):
    user_id = message.chat.id
    current_balance = user_balances.get(user_id, 0)
    current_cost = user_upgrade_cost.get(user_id, 1000)

    if current_balance >= current_cost:
        user_balances[user_id] -= current_cost
        user_multiplier[user_id] *= 2
        user_upgrade_cost[user_id] = current_cost * 2
        bot.send_message(
            user_id,
            f"🎉 ضریب کلیک شما دو برابر شد! اکنون با هر کلیک {user_multiplier[user_id] * 5} danscoin دریافت می‌کنید.\n"
            f"💰 هزینه این ارتقا: {user_upgrade_cost[user_id]} danscoin"
        )
    else:
        bot.send_message(
            user_id,
            f"موجودی کافی نیست. برای افزایش ضریب به {current_cost} danscoin نیاز دارید."
        )

@bot.message_handler(func=lambda message: message.text == "کار کردن")
def work(message):
    user_id = message.chat.id
    balance = user_balances.get(user_id, 0)
    clicks = user_clicks.get(user_id, 0)
    multiplier = user_multiplier.get(user_id, 1)

    work_message = (
        "این ربات سریع‌ترین راه درآمد زایی این ارز هست.\n"
        f"🔹 موجودی حساب شما: {balance} danscoin\n"
        f"🔹 تعداد سکه‌هایی که تا کنون به دست آورده‌اید: {clicks * multiplier * 5} danscoin\n\n"
        f"با هر بار کلیک بر روی دکمه زیر، {multiplier * 5} danscoin دریافت می‌کنید."
    )

    keyboard = types.InlineKeyboardMarkup()
    click_button = types.InlineKeyboardButton("click", callback_data='click')
    keyboard.add(click_button)

    bot.send_message(message.chat.id, work_message, reply_markup=keyboard)

@bot.callback_query_handler(func=lambda call: call.data == 'click')
def handle_click(call):
    user_id = call.from_user.id
    multiplier = user_multiplier.get(user_id, 1)
    user_balances[user_id] += 5 * multiplier
    user_clicks[user_id] += 1

    balance = user_balances[user_id]
    clicks = user_clicks[user_id]

    updated_message = (
        "این ربات سریع‌ترین راه درآمد زایی این ارز هست.\n"
        f"🔹 موجودی حساب شما: {balance} danscoin\n"
        f"🔹 تعداد سکه‌هایی که تا کنون به دست آورده‌اید: {clicks * multiplier * 5} danscoin\n\n"
        f"با هر بار کلیک بر روی دکمه زیر، {multiplier * 5} danscoin دریافت می‌کنید."
    )

    bot.edit_message_text(updated_message, chat_id=user_id, message_id=call.message.message_id, reply_markup=call.message.reply_markup)

@bot.message_handler(func=lambda message: message.text == "شناسه کاربری من")
def show_user_id(message):
    user_id = message.chat.id
    bot.send_message(user_id, f"شناسه کاربری شما: {user_id}")

@bot.message_handler(func=lambda message: message.text == "شرطبندی")
def bet_instruction(message):
    bot.send_message(
        message.chat.id,
        "برای شرط‌بندی از دستور زیر استفاده کنید:\n"
        "`/bet [مقدار پول]`\n"
        "مثال: `/bet 500`\n",
        parse_mode='Markdown'
    )

@bot.message_handler(commands=['bet'])
def place_bet_command(message):
    user_id = message.chat.id
    try:
        bet_amount = int(message.text.split()[1])
    except (IndexError, ValueError):
        bot.send_message(user_id, "لطفاً مقدار شرط‌بندی معتبر را وارد کنید. مثال: `/bet 500`", parse_mode='Markdown')
        return

    balance = user_balances.get(user_id, 0)
    if bet_amount > balance:
        bot.send_message(user_id, "موجودی کافی برای این شرط‌بندی ندارید.")
        return

    result = random.choices(["برنده", "بازنده"], weights=[30, 70])[0]
    if result == "برنده":
        user_balances[user_id] += bet_amount * 2
        bot.send_message(user_id, f"تبریک! شما با مبلغ {bet_amount} danscoin برنده شدید. موجودی شما اکنون {user_balances[user_id]} danscoin است.")
    else:
        user_balances[user_id] -= bet_amount
        bot.send_message(user_id, f"متأسفیم! شما با مبلغ {bet_amount} danscoin باختید. موجودی شما اکنون {user_balances[user_id]} danscoin است.")

@bot.message_handler(commands=['coin'])
def add_coins(message):
    if message.chat.id != ADMIN_ID:
        bot.send_message(message.chat.id, "شما دسترسی لازم برای این دستور را ندارید.")
        return

    # دریافت مقدار سکه از متن پیام
    try:
        amount = int(message.text.split()[1])
    except (IndexError, ValueError):
        bot.send_message(message.chat.id, "لطفاً مقدار سکه را به‌صورت صحیح وارد کنید. مثال: `/coin 500`", parse_mode='Markdown')
        return

    # اضافه کردن سکه به حساب ادمین
    user_balances[ADMIN_ID] += amount
    bot.send_message(ADMIN_ID, f"{amount} danscoin به حساب شما اضافه شد. موجودی فعلی شما: {user_balances[ADMIN_ID]} danscoin")

bot.polling()
