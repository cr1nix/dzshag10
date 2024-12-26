import sqlite3
from datetime import datetime
import requests
from bs4 import BeautifulSoup


conn = sqlite3.connect('weather.db')
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE IF NOT EXISTS weather (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        datetime TEXT NOT NULL,
        temperature REAL NOT NULL
    )
''')
conn.commit()


url = 'https://www.accuweather.com/uk/ua/kyiv/324505/current-weather/324505'
headers = {'User-Agent': 'Mozilla/5.0'}
response = requests.get(url, headers=headers)
soup = BeautifulSoup(response.content, 'html.parser')


temp_element = soup.find('div', class_='temp')
if temp_element:
    temperature = temp_element.get_text(strip=True).replace('°', '')
    try:
        temperature = float(temperature)
    except ValueError:
        print("Не вдалося перетворити температуру на число.")
        temperature = None
else:
    print("Не вдалося знайти елемент з температурою.")
    temperature = None


if temperature is not None:
    current_datetime = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    cursor.execute('''
        INSERT INTO weather (datetime, temperature)
        VALUES (?, ?)
    ''', (current_datetime, temperature))
    conn.commit()
    print(f"Дані додано: {current_datetime}, {temperature}°C")
else:
    print("Дані не додано через помилку отримання температури.")


conn.close()
