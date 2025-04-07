import speech_recognition as sr
import pyttsx3
import openai
import pywhatkit
import os
import requests
import datetime
import webbrowser
from googlesearch import search

# Set API Keys (Replace with actual keys)
OPENAI_API_KEY = " "
WEATHER_API_KEY = "4f54fcb19bb9a8f5cb8ad5b074915b26"

openai.api_key = OPENAI_API_KEY

# Initialize Text-to-Speech
engine = pyttsx3.init()
engine.setProperty('rate', 160)  # Speed of voice
engine.setProperty('volume', 1.0)  # Volume level

def speak(text):
    """Convert text to speech."""
    engine.say(text)
    engine.runAndWait()

# Initialize Speech Recognition
recognizer = sr.Recognizer()
recognizer.energy_threshold = 300  # Filter background noise
recognizer.dynamic_energy_threshold = True  # Auto-adjust threshold

def listen():
    """Capture voice command from the microphone."""
    with sr.Microphone() as source:
        print("Listening...")
        recognizer.adjust_for_ambient_noise(source, duration=1)
        try:
            audio = recognizer.listen(source, timeout=5)
            command = recognizer.recognize_google(audio).lower()
            print("You said:", command)
            return command
        except sr.UnknownValueError:
            return ""
        except sr.RequestError:
            speak("Sorry, I am having trouble connecting to the internet.")
            return ""
        except sr.WaitTimeoutError:
            return ""

def get_weather(city):
    """Fetch weather details for a given city."""
    url = f"http://api.weatherstack.com/current?access_key={WEATHER_API_KEY}&query={city}"
    response = requests.get(url).json()

    if "current" in response:
        temp = response["current"]["temperature"]
        weather_desc = response["current"]["weather_descriptions"][0]
        return f"The temperature in {city} is {temp}°C with {weather_desc}."
    else:
        return "Sorry, I couldn't find the weather details. Please check the city name."

def get_time():
    """Get the current time."""
    now = datetime.datetime.now()
    return now.strftime("%I:%M %p")

def search_google(query):
    """Perform a Google search and return the top result."""
    results = list(search(query, num=1, stop=1))
    return results[0] if results else "No results found."

def open_maps(destination):
    """Open Google Maps with directions."""
    url = f"https://www.google.com/maps/dir//{destination.replace(' ', '+')}"
    webbrowser.open(url)
    return f"Opening Google Maps for directions to {destination}."

def execute_command(command):
    """Process user commands."""
    if "hello" in command:
        speak("Hello! How can I assist you?")
    elif "time" in command:
        current_time = get_time()
        speak(f"The current time is {current_time}.")
    elif "weather" in command:
        speak("Which city's weather do you want to check?")
        city = listen().strip()
        if city:
            weather_info = get_weather(city)
            speak(weather_info)
        else:
            speak("I couldn't hear the city name. Please try again.")
    elif "play" in command:
        song = command.replace("play", "").strip()
        speak(f"Playing {song} on YouTube")
        pywhatkit.playonyt(song)
    elif "open youtube" in command:
        speak("Opening YouTube")
        webbrowser.open("https://www.youtube.com")
    elif "search" in command:
        query = command.replace("search", "").strip()
        speak(f"Searching Google for {query}")
        result = search_google(query)
        speak(f"Here is the top result: {result}")
        webbrowser.open(result)
    elif "calculate" in command:
        try:
            expression = command.replace("calculate", "").strip()
            result = eval(expression)
            speak(f"The result is {result}")
        except:
            speak("Sorry, I couldn't perform the calculation.")
    elif "navigate" in command or "directions to" in command:
        speak("Which location do you want directions to?")
        destination = listen().strip()
        if destination:
            response = open_maps(destination)
            speak(response)
        else:
            speak("I couldn't understand the destination. Please try again.")
    elif "shutdown" in command:
        speak("Shutting down your computer.")
        os.system("shutdown /s /t 5")
    elif "restart" in command:
        speak("Restarting your computer.")
        os.system("shutdown /r /t 5")
    elif "exit" in command or "stop" in command:
        speak("Goodbye! Have a great day!")
        exit()
    else:
        speak("I didn't understand that command.")

if __name__ == "__main__":
    speak("Hello, I am Mahi. How can I assist you?")
    while True:
        user_command = listen()
        if user_command:
            execute_command(user_command)
