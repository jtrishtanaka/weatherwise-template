---
title: "Before/After Example: Weather Data Retrieval Function"
author: "Tanaka Jera"
format: 
  pdf:
    toc: false
    colorlinks: true
  docx:
    toc: false
    highlight-style: github
  html:
    toc: true
    toc-expand: 2
    embed-resources: true
---


## Initial Prompt

```
Write a function to retrieve weather data for a user specified location .
```

## Initial AI-Generated Code (Before)

```python
import requests

API_KEY = "YOUR_API_KEY_HERE"  # 🔐 Replace with your real OpenWeatherMap API key

def get_weather(city):
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
    try:
        res = requests.get(url, timeout=5).json()
        if res.get("cod") != 200:
            print("❌ City not found.")
            return
        print(f"\n🌤️ {res['name']}, {res['sys']['country']}")
        print(f"Temp: {res['main']['temp']}°C | {res['weather'][0]['description'].capitalize()}")
    except:
        print("⚠️ Error: Couldn't fetch weather.")

city = input("Enter a city: ")
get_weather(city)

```

## My Analysis of the Initial Code

The initial code has several issues:

1. No error handling - will crash if the API is down or returns an error
2. No input validation for the location parameter
3. Returns the raw API response which is complex and includes unnecessary data
4. Hardcoded API KEY 
5. No fallback or suggestions thus no smart UX .

## My Follow-up Prompt

```
Thanks for the starting point. I'd like to improve this function with:

1. Error handling for API failures and bad responses
2. Location input validation
3. Generated API key 
4. A way to process the response to return only essential weather information
5. Proper documentation with parameters and return values explained

Also, can you make it handle the forecast_days parameter to allow getting forecast for multiple days?
```

## Improved AI-Generated Code (After)

```python
import requests
import os
import sys
from urllib.parse import quote
from collections import defaultdict, Counter
from datetime import datetime

def get_api_key():
    key = os.getenv("OPENWEATHER_API_KEY")
    if not key:
        print("❌ API key not found. Set OPENWEATHER_API_KEY in your environment.")
        sys.exit(1)
    return key

def validate_city(city):
    return bool(city and city.replace(" ", "").isalpha())

def fetch_current_weather(city, api_key):
    encoded_city = quote(city)
    url = f"https://api.openweathermap.org/data/2.5/weather?q={encoded_city}&appid={api_key}&units=metric"
    try:
        res = requests.get(url, timeout=5)
        res.raise_for_status()
        data = res.json()
        if data.get("cod") != 200:
            print(f"❌ Error: {data.get('message', 'City not found.')}")
            return None
        return data
    except requests.exceptions.RequestException as e:
        print(f"⚠️ Network or API error: {e}")
        return None

def fetch_forecast(city, api_key):
    encoded_city = quote(city)
    url = f"https://api.openweathermap.org/data/2.5/forecast?q={encoded_city}&appid={api_key}&units=metric"
    try:
        res = requests.get(url, timeout=5)
        res.raise_for_status()
        data = res.json()
        if data.get("cod") != "200":
            print(f"❌ Forecast error: {data.get('message', 'Not available')}")
            return None
        return data
    except requests.exceptions.RequestException as e:
        print(f"⚠️ Forecast fetch error: {e}")
        return None

def display_current_weather(data):
    name = data["name"]
    country = data["sys"]["country"]
    temp = data["main"]["temp"]
    feels_like = data["main"]["feels_like"]
    desc = data["weather"][0]["description"].capitalize()
    humidity = data["main"]["humidity"]
    wind = data["wind"]["speed"]
    icon = data["weather"][0]["icon"]
    emoji = weather_icon_emoji(icon)

    print(f"\n {name}, {country} (Now)")
    print(f"{emoji} {desc}")
    print(f"🌡️ Temp: {temp}°C (Feels like {feels_like}°C)")
    print(f"💧 Humidity: {humidity}%")
    print(f"🌬️ Wind Speed: {wind} m/s")

def display_forecast(data):
    print("\n📅 5-Day Forecast:")
    days = defaultdict(list)

    for entry in data["list"]:
        date = entry["dt_txt"].split(" ")[0]
        temp = entry["main"]["temp"]
        desc = entry["weather"][0]["description"]
        icon = entry["weather"][0]["icon"]
        days[date].append((temp, desc, icon))

    for date, entries in list(days.items())[:5]:
        temps = [t[0] for t in entries]
        descs = [d[1] for d in entries]
        icons = [i[2] for i in entries]
        avg_temp = sum(temps) / len(temps)
        common_desc = Counter(descs).most_common(1)[0][0].capitalize()
        emoji = weather_icon_emoji(icons[0])

        date_fmt = datetime.strptime(date, "%Y-%m-%d").strftime("%A, %b %d")
        print(f"\n📆 {date_fmt}")
        print(f"{emoji} {common_desc}")
        print(f"🌡️ Avg Temp: {avg_temp:.1f}°C")

def weather_icon_emoji(icon_code):
    mapping = {
        "01": "☀️", "02": "🌤️", "03": "⛅", "04": "☁️",
        "09": "🌧️", "10": "🌦️", "11": "⛈️", "13": "❄️", "50": "🌫️"
    }
    return mapping.get(icon_code[:2], "🌈")

def main():
    print("🌍 Weather Pro 3000 – Now with 5-Day Forecast!")
    city = input("Enter a city: ").strip()

    if not validate_city(city):
        print("⚠️ Invalid input. Use letters only.")
        return

    api_key = get_api_key()
    current = fetch_current_weather(city, api_key)
    forecast = fetch_forecast(city, api_key)

    if current:
        display_current_weather(current)
    if forecast:
        display_forecast(forecast)

if __name__ == "__main__":
    main()

    
    

    
```

## Why My Prompting Strategy Was Effective

My follow-up prompt was effective because:

1. **I identified specific problems** in the initial code rather than just asking for "better code"
2. **I requested specific improvements** with clear objectives (error handling, validation, etc.)
3. **I suggested a new feature** (forecast_days parameter) that would make the function more useful
4. **I asked for proper documentation** which resulted in comprehensive docstrings

This approach guided the AI to generate a much more robust function that:

- Is properly documented
- Validates inputs
- Has comprehensive error handling
- Returns structured, processed data instead of raw API response
- Includes the new forecast functionality I requested

The key to effective prompting was being specific about what needed improvement and why, rather than making vague requests for "better" code.
