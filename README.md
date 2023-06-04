import requests
import telebot
import json

TOKEN = '5741584233:AAGq57TN85IlE2zB6wAfcY-l7228w5MML18'
API_KEY = '5682bca392624b73ad61786609f19e7b'

bot = telebot.TeleBot(TOKEN)

keys = {'EUR', 'USD', 'RUB'}  # Define the currency keys as strings

@bot.message_handler(commands=['start', 'help'])
def help(message: telebot.types.Message):
    text = 'Чтобы начать работу, введите команду боту в следующем формате: \n<имя валюты> <в какую валюту перевести> \
\ <количество переводимой валюты>\nЧтобы увидеть список всех доступных валют нажмите на ссылку:/values'
    bot.reply_to(message, text)

@bot.message_handler(commands=['values'])
def values(message: telebot.types.Message):
    text = 'Доступные валюты'
    for key in keys:
        text = '\n'.join((text, key))
    bot.reply_to(message, text)

@bot.message_handler(content_types=['text'])
def convert(message: telebot.types.Message):
    try:
        quote, base, amount = message.text.split()
        url = f"https://openexchangerates.org/api/latest.json?app_id={'5682bca392624b73ad61786609f19e7b'}&symbols={quote},{base}"
        response = requests.get(url)
        data = response.json()
        rates = data['rates']
        if quote in rates and base in rates:
            quote_rate = rates[quote]
            base_rate = rates[base]
            converted_amount = float(amount) * (base_rate / quote_rate)
            result = f"{amount} {quote} = {converted_amount} {base}"
            bot.reply_to(message, result)
        else:
            bot.reply_to(message, 'Неверные символы валюты.')
    except ValueError:
        bot.reply_to(message, 'Неверный формат ввода.')
    except requests.exceptions.RequestException:
        bot.reply_to(message, 'Ошибка при получении курсов обмена.')


bot.polling()

