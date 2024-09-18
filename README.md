from pyrogram import Client
import pandas as pd
import matplotlib.pyplot as plt

# Ваши api_id и api_hash
api_id = '####'
api_hash = '######'

# Настройте ваш Telegram клиент
app = Client("telegram_analytics", api_id=api_id, api_hash=api_hash)

# Список юзернеймов каналов (без префикса @)
channel_usernames = [
    'crypthustle',
    'Temsikoof',
    'its_cryptotrade',
    'drivecrypto',
    'cryptonspaceen',
    'libertycryptoq',
    'cryptotoponchaindata',
    'crypto_vestor',
    'BN_invest',
    'richydad_tg',
    'cryptotoponchaindataru',
    'mnstmoney',
    'tcryptoday',
    'cryyptonight',
    'UKRIPTA'
]

def analyze_channel(channel_username):
    with app:
        # Получаем основную информацию о канале
        channel = app.get_chat(channel_username)
        print(f"Название канала: {channel.title}")
        print(f"Количество подписчиков: {channel.members_count}")
        
        # Получаем последние 100 сообщений для анализа
        messages = app.get_chat_history(channel_username, limit=1000)
        data = []

        for message in messages:
            if message.views is not None:  # Проверяем, есть ли информация о просмотрах
                # Получаем количество реакций, если они есть
                reaction_count = 0
                if message.reactions:
                    # Проверяем, содержит ли message.reactions атрибут `reactions`
                    if hasattr(message.reactions, 'reactions'):
                        reaction_count = sum(reaction.count for reaction in message.reactions.reactions)

                data.append({
                    'message_id': message.id,
                    'views': message.views,
                    'reactions': reaction_count,
                    'date': message.date
                })

        # Создаем DataFrame для обработки данных
        df = pd.DataFrame(data)
        if df.empty:
            print("Недостаточно данных для анализа.")
            return
        
        # Расчет средней вовлеченности
        average_views = df['views'].mean()
        median_views = df['views'].median()
        average_reactions = df['reactions'].mean()

        # Дополнительные параметры анализа
        views_to_subscribers_ratio = (average_views / channel.members_count) * 100
        reaction_to_view_ratio = (average_reactions / average_views) * 100 if average_views > 0 else 0
        publication_frequency = len(df) / ((df['date'].max() - df['date'].min()).days + 1)

        print(f"Среднее количество просмотров: {average_views:.2f}")
        print(f"Медианное количество просмотров: {median_views:.2f}")
        print(f"Среднее количество реакций: {average_reactions:.2f}")
        print(f"Процент вовлеченности (просмотры/подписчики): {views_to_subscribers_ratio:.2f}%")
        print(f"Соотношение реакций к просмотрам: {reaction_to_view_ratio:.2f}%")
        print(f"Частота публикаций (постов в день): {publication_frequency:.2f}")

        # Проверка на накрутку
        if views_to_subscribers_ratio < 10:
            print("Возможна накрутка: низкое соотношение просмотров к подписчикам.")
        if reaction_to_view_ratio < 1:
            print("Возможна накрутка: низкое соотношение реакций к просмотрам.")
        if publication_frequency > 50:  # Например, если больше 50 постов в день
            print("Возможна накрутка: слишком высокая частота публикаций.")
        if df['views'].max() > average_views * 5:
            print("Возможна накрутка: аномальные пики просмотров.")
        else:
            print("Вовлеченность в норме.")

        # Визуализация распределения просмотров
        plt.figure(figsize=(10, 5))
        plt.plot(df['date'], df['views'], marker='o')
        plt.title(f'Динамика просмотров сообщений в канале {channel.title}')
        plt.xlabel('Дата')
        plt.ylabel('Количество просмотров')
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()

# Запускаем анализ для каждого канала
for username in channel_usernames:
    print(f"\nАнализ канала: {username}")
    try:
        analyze_channel(username)
    except Exception as e:
        print(f"Ошибка при анализе канала {username}: {e}")
