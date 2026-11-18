"""
==================================================
FAST CHAT AI - Ultimate Error-Free Mobile App
Developer: Kolli Satwik (Satwik Star)
Age: 15 Years | Aspiring Software Engineer
Amalapuram, Ambedkar Konaseema District
==================================================
"""

import os
import json
import threading

try:
    import requests
except ImportError:
    requests = None

from kivy.app import App
from kivy.clock import Clock
from kivy.metrics import dp
from kivy.core.window import Window
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.scrollview import ScrollView
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.widget import Widget
from kivy.graphics import Color, RoundedRectangle

APP_NAME = "FAST CHAT AI"
MODEL_NAME = "gpt-4o-mini"
API_URL = "https://api.openai.com/v1/chat/completions"

Window.clearcolor = (0.035, 0.055, 0.09, 1)
Window.softinput_mode = "pan"

class FastChatAI(App):
    def build(self):  
        self.title = APP_NAME  
        self.history = []  
        self.busy = False  
        try:
            self.history_file = os.path.join(self.user_data_dir, "fast_chat_history.json")
        except Exception:
            self.history_file = "fast_chat_history.json"
        self.load_history()  
        self.main = BoxLayout(orientation="vertical", spacing=dp(8), padding=dp(12))  
        self.show_splash()  
        Clock.schedule_once(self.show_chat, 1.5)  
        return self.main  

    def show_splash(self, *args):  
        self.main.clear_widgets()  
        self.main.add_widget(Widget())  
        logo = Label(text="[b]⚡ FAST CHAT AI[/b]", markup=True, font_size="30sp", color=(0.25, 0.75, 1, 1), size_hint_y=None, height=dp(65))  
        subtitle = Label(text="Powered by OpenAI | Created by Satwik Star", font_size="15sp", color=(0.8, 0.85, 0.95, 1), size_hint_y=None, height=dp(40))  
        self.main.add_widget(logo)  
        self.main.add_widget(subtitle)  
        self.main.add_widget(Widget())  

    def show_chat(self, *args):  
        self.main.clear_widgets()  
        header = BoxLayout(orientation="vertical", size_hint_y=None, height=dp(85), spacing=dp(4))  
        title = Label(text="[b]⚡ FAST CHAT AI[/b]", markup=True, font_size="23sp", color=(0.25, 0.75, 1, 1))  
        powered = Label(text="Developer: Kolli Satwik (15 Yrs)", font_size="12sp", color=(0.65, 0.75, 0.9, 1))  
        header.add_widget(title)  
        header.add_widget(powered)  
        self.main.add_widget(header)  

        self.api_key = TextInput(hint_text="Enter your OpenAI API key", password=True, multiline=False, size_hint_y=None, height=dp(46), font_size="14sp", padding=dp(12), background_color=(0.12, 0.16, 0.23, 1), foreground_color=(1, 1, 1, 1))  
        self.main.add_widget(self.api_key)  

        buttons = BoxLayout(size_hint_y=None, height=dp(42), spacing=dp(8))  
        new_button = Button(text="＋ New Chat", background_normal="", background_color=(0.1, 0.35, 0.55, 1))  
        new_button.bind(on_release=self.new_chat)  
        clear_button = Button(text="Clear Screen", background_normal="", background_color=(0.35, 0.22, 0.3, 1))  
        clear_button.bind(on_release=self.clear_screen)  
        buttons.add_widget(new_button)  
        buttons.add_widget(clear_button)  
        self.main.add_widget(buttons)  

        self.scroll = ScrollView(do_scroll_x=False, bar_width=dp(4))  
        self.chat_layout = GridLayout(cols=1, spacing=dp(12), size_hint_y=None, padding=(dp(5), dp(10)))  
        self.chat_layout.bind(minimum_height=self.chat_layout.setter("height"))  
        self.scroll.add_widget(self.chat_layout)  
        self.main.add_widget(self.scroll)  

        bottom = BoxLayout(size_hint_y=None, height=dp(58), spacing=dp(7))  
        self.message_input = TextInput(hint_text="Message FAST CHAT AI...", multiline=False, font_size="16sp", padding=dp(12), background_color=(0.12, 0.16, 0.23, 1), foreground_color=(1, 1, 1, 1))  
        self.message_input.bind(on_text_validate=self.send_message)  
        send_button = Button(text="SEND ➤", size_hint_x=None, width=dp(90), background_normal="", background_color=(0.05, 0.48, 0.8, 1))  
        send_button.bind(on_release=self.send_message)  
        bottom.add_widget(self.message_input)  
        bottom.add_widget(send_button)  
        self.main.add_widget(bottom)  

        self.status = Label(text="Ready to chat", size_hint_y=None, height=dp(23), font_size="11sp", color=(0.5, 0.75, 0.85, 1))  
        self.main.add_widget(self.status)  
        self.refresh_chat()  

    def add_message(self, speaker, message):  
        colors = {"YOU": (0.3, 0.8, 1, 1), "FAST AI": (0.65, 1, 0.7, 1), "ERROR": (1, 0.4, 0.4, 1)}  
        card = BoxLayout(orientation="vertical", size_hint_y=None, padding=dp(12), spacing=dp(6))  
        with card.canvas.before:  
            Color(0.10, 0.15, 0.22, 1)  
            rectangle = RoundedRectangle(pos=card.pos, size=card.size, radius=[dp(12)])  
        card.bind(pos=lambda instance, value: setattr(rectangle, "pos", value), size=lambda instance, value: setattr(rectangle, "size", value))  
        heading = Label(text=speaker, color=colors.get(speaker, (1, 1, 1, 1)), bold=True, font_size="14sp", size_hint_y=None, height=dp(25), halign="left")  
        body = Label(text=str(message), color=(0.95, 0.96, 1, 1), font_size="15sp", size_hint_y=None, halign="left", valign="top")  
        body.text_size = (Window.width - dp(65), None)  
        body.bind(texture_size=lambda instance, value: setattr(instance, "height", value[1] + dp(8)))  
        card.add_widget(heading)  
        card.add_widget(body)  
        card.bind(minimum_height=card.setter("height"))  
        self.chat_layout.add_widget(card)  
        Clock.schedule_once(lambda dt: setattr(self.scroll, "scroll_y", 0), 0.1)  

    def refresh_chat(self):  
        self.chat_layout.clear_widgets()  
        if not self.history:  
            self.add_message("FAST AI", "Hello! Welcome to FAST CHAT AI.\n\nAsk me anything. I am ready to help you!")  
            return  
        for item in self.history:  
            speaker = "YOU" if item["role"] == "user" else "FAST AI"  
            self.add_message(speaker, item["content"])  

    def send_message(self, *args):  
        if self.busy:  
            self.status.text = "Please wait for the current answer."  
            return  
        question = self.message_input.text.strip()  
        key = self.api_key.text.strip()  
        if not question:  
            self.status.text = "Please enter a message."  
            return  
        if not key:  
            self.status.text = "API key is required."  
            return  
        if requests is None:
            self.status.text = "Error: 'requests' module not installed."
            return  
        self.message_input.text = ""  
        self.add_message("YOU", question)  
        self.status.text = "FAST AI is thinking..."  
        self.busy = True  
        self.message_input.disabled = True  
        threading.Thread(target=self.ask_ai, args=(question, key), daemon=True).start()  

    def ask_ai(self, question, key):  
        try:  
            instructions = "You are FAST CHAT AI, a helpful and careful AI assistant. Give accurate, useful and clear answers, support Telugu and English."
            messages = [{"role": "system", "content": instructions}]
            for item in self.history[-16:]:
                messages.append({"role": item["role"], "content": item["content"]})
            messages.append({"role": "user", "content": question})
            payload = {"model": MODEL_NAME, "messages": messages, "max_tokens": 1200}  
            headers = {"Authorization": "Bearer " + key, "Content-Type": "application/json"}  
            response = requests.post(API_URL, headers=headers, json=payload, timeout=90)  
            if response.status_code != 200:  
                raise Exception(f"API Error {response.status_code}")  
            data = response.json()  
            answer = data.get("choices", [{}])[0].get("message", {}).get("content", "").strip()
            if not answer:  
                raise Exception("Empty response.")  
            self.history.append({"role": "user", "content": question})  
            self.history.append({"role": "assistant", "content": answer})  
            self.save_history()  
            Clock.schedule_once(lambda dt: self.finish_answer(answer), 0)  
        except Exception as error:  
            Clock.schedule_once(lambda dt: self.show_error(str(error)), 0)  

    def finish_answer(self, answer):  
        self.add_message("FAST AI", answer)  
        self.status.text = "Answer received."  
        self.busy = False  
        self.message_input.disabled = False  
        self.message_input.focus = True  

    def show_error(self, error):  
        self.add_message("ERROR", error)  
        self.status.text = "Error occurred."  
        self.busy = False  
        self.message_input.disabled = False  

    def save_history(self):  
        try:  
            os.makedirs(os.path.dirname(self.history_file), exist_ok=True)  
            with open(self.history_file, "w", encoding="utf-8") as file:  
                json.dump(self.history, file, ensure_ascii=False, indent=2)  
        except Exception:  
            pass  

    def load_history(self):  
        try:  
            if os.path.exists(self.history_file):  
                with open(self.history_file, "r", encoding="utf-8") as file:  
                    data = json.load(file)  
                if isinstance(data, list):  
                    self.history = data  
        except Exception:  
            self.history.append = []  

    def new_chat(self, *args):  
        self.history = []  
        self.save_history()  
        self.refresh_chat()  

    def clear_screen(self, *args):  
        self.chat_layout.clear_widgets()  
        self.add_message("FAST AI", "Screen cleared.")  

if __name__ == "__main__":  
    FastChatAI().run()
 About the Developer: Kolli Satwik (Satwik Star)
Profile & Background:
Kolli Satwik  is a remarkably talented 15-year-old aspiring software engineer and student residing in Amalapuram , Ambedkar Konaseema district, Andhra Pradesh.
An Extraordinary Achievement:
At just 15 years of age, without relying on high-end computers or expensive setups, Satwik has utilized his mobile phone as his primary powerhouse to learn Python programming, Kivy framework, and API integration. Building "FAST CHAT AI" entirely on a mobile device showcases his rare determination, passion, and natural technical brilliance. His relentless drive to overcome every technical hurdle proves that age is just a number when passion and hard work meet. Satwik Star is undoubtedly on a brilliant path toward becoming an exceptional software engineering leader! 🚀 and his faith only jesus christ 