# AI_Phone_Agent
An AI-powered phone agent built with Python, Tkinter, and NLP for speech interaction and user engagement.
import os
import speech_recognition as sr
import pyttsx3
import spacy
import tkinter as tk
from tkinter import font
from threading import Thread
import datetime
import requests
import random
import logging
import queue

# Set up logging
logging.basicConfig(level=logging.INFO)

# Initialize text-to-speech engine
engine = pyttsx3.init()

# Load spaCy model for NLP
nlp = spacy.load("en_core_web_sm")

class AIAgent:
    def __init__(self):
        self.root = tk.Tk()
        self.setup_ui()
        self.response_queue = queue.Queue()  # Queue for thread communication
        self.is_listening = False
        self.user_name = None  # To store user's name
        self.ai_name = "Ehonexus"  # Name of the AI

    def setup_ui(self):
        self.root.title("AI Phone Agent")
        self.root.geometry("500x500")
        self.root.config(bg="#f4f4f9")

        custom_font = font.nametofont("TkDefaultFont").copy()
        custom_font.config(size=12, family="Helvetica")

        title_label = tk.Label(self.root, text="AI Phone Agent", font=("Helvetica", 16, "bold"), fg="#4A90E2", bg="#f4f4f9")
        title_label.pack(pady=20)

        frame = tk.Frame(self.root, bg="#f4f4f9")
        frame.pack(padx=10, pady=10)

        self.chat_box = tk.Text(frame, height=15, width=50, wrap=tk.WORD, font=custom_font, bg="#e0f7fa", fg="#004d40", bd=1, relief="solid", state=tk.DISABLED)
        self.chat_box.grid(row=0, column=0)

        scrollbar = tk.Scrollbar(frame, orient=tk.VERTICAL)
        scrollbar.config(command=self.chat_box.yview)
        scrollbar.grid(row=0, column=1, sticky="ns")

        start_button = tk.Button(self.root, text="Start Listening", command=self.start_interaction, width=20, height=2, bg="#4A90E2", fg="white", font=("Helvetica", 12, "bold"), relief="flat", activebackground="#007aff")
        start_button.pack(pady=20)

        footer_label = tk.Label(self.root, text="AI Phone Agent - Powered by Tkinter", font=("Helvetica", 8), fg="#888", bg="#f4f4f9")
        footer_label.pack(side="bottom", pady=10)

    def speak(self, text):
        engine.say(text)
        engine.runAndWait()

    def listen_to_user(self):
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            logging.info("Adjusting for ambient noise...")
            recognizer.adjust_for_ambient_noise(source, duration=1)
            logging.info("Listening for a command...")
            audio = recognizer.listen(source)

        try:
            text = recognizer.recognize_google(audio)
            return text
        except sr.UnknownValueError:
            return "Sorry, I could not understand that."
        except sr.RequestError:
            return "Sorry, the speech service is down."

    def process_input(self, user_input):
        doc = nlp(user_input)
        response = ""

        # Check for user name
        if self.user_name is None and ("my name is" in user_input.lower() or "call me" in user_input.lower()):
            self.user_name = user_input.split()[-1]  # Assume the last word is the name
            response = f"Nice to meet you, {self.user_name}!"
        elif "hello" in user_input.lower():
            response = f"Hello, {self.user_name if self.user_name else 'how can I assist you today'}? I am Ehonexus."
        elif "help" in user_input.lower():
            response = "I can help you with information, making a call, or managing your data."
        elif "weather" in user_input.lower():
            response = self.get_weather(user_input)
        elif "joke" in user_input.lower():
            response = self.get_joke()
        elif "quote" in user_input.lower():
            response = self.get_quote()
        elif "time" in user_input.lower() or "date" in user_input.lower():
            response = self.get_time_and_date()
        elif "shutdown" in user_input.lower() or "exit" in user_input.lower():
            response = "Shutting down. Goodbye!"
            self.root.quit()
        else:
            response = "Sorry, I didn't quite catch that. Can you please rephrase your request?"

        self.speak(response)
        self.update_chat_box(f"Ehonexus: {response}")

    def get_weather(self, user_input):
        # Placeholder for weather fetching logic
        return "I can fetch the weather for you. Please specify the city."

    def get_joke(self):
        # Placeholder for joke fetching logic
        return "Why did the scarecrow win an award? Because he was outstanding in his field!"

    def get_quote(self):
        # Placeholder for quote fetching logic
        return "The only way to do great work is to love what you do. - Steve Jobs"

    def get_time_and_date(self):
        now = datetime.datetime.now()
        return f"The current date and time is {now.strftime('%Y-%m-%d %H:%M:%S')}."

    def update_chat_box(self, message):
        self.chat_box.config(state=tk.NORMAL)
        self.chat_box.insert(tk.END, message + "\n")
        self.chat_box.config(state=tk.DISABLED)
        self.chat_box.yview(tk.END)

    def start_interaction(self):
        self.is_listening = True
        self.update_chat_box("Ehonexus: Listening for your command...")
        while self.is_listening:
            user_input = self.listen_to_user()
            self.process_input(user_input)

if __name__ == "__main__":
    agent = AIAgent()
    agent.root.mainloop()
