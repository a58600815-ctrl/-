# -import requests
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.utils import get_color_from_hex

class RobloxRadarApp(App):
    def build(self):
        self.layout = BoxLayout(orientation='vertical', padding=20, spacing=15)
        
        # Название внутри приложения
        self.title_label = Label(
            text="Радар Роблокс 🎯", 
            font_size='24sp', 
            bold=True,
            size_hint_y=None,
            height=50,
            color=get_color_from_hex('#FFFFFF')
        )
        self.layout.add_widget(self.title_label)
        
        self.user_input = TextInput(
            text='', 
            hint_text='Введите Никнейм или User ID...', 
            multiline=False,
            size_hint_y=None,
            height=50,
            font_size='18sp'
        )
        self.layout.add_widget(self.user_input)
        
        self.check_button = Button(
            text="Сканировать статус", 
            size_hint_y=None,
            height=55,
            background_color=get_color_from_hex('#00A2FF'),
            font_size='18sp',
            bold=True
        )
        self.check_button.bind(on_press=self.start_search)
        self.layout.add_widget(self.check_button)
        
        self.result_label = Label(
            text="Радар готов к работе", 
            font_size='18sp',
            halign='center',
            valign='middle'
        )
        self.result_label.bind(size=self.result_label.setter('text_size'))
        self.layout.add_widget(self.result_label)
        
        return self.layout

    def start_search(self, instance):
        query = self.user_input.text.strip()
        if not query:
            self.result_label.text = "⚠️ Введите ник или ID!"
            return
        self.result_label.text = "📡 Сканирую сеть..."
        if query.isdigit():
            self.get_roblox_status(int(query), f"ID: {query}")
        else:
            self.convert_username_to_id(query)

    def convert_username_to_id(self, username):
        url = "https://roblox.com"
        payload = {"usernames": [username], "excludeBannedUsers": False}
        try:
            response = requests.post(url, json=payload, timeout=5)
            if response.status_code == 200:
                data = response.json().get("data", [])
                if data:
                    user_id = data.get("id")
                    official_name = data.get("name")
                    self.get_roblox_status(user_id, f"Ник: {official_name}")
                else:
                    self.result_label.text = f"❌ Игрок '{username}' не найден."
            else:
                self.result_label.text = f"❌ Ошибка ответа: {response.status_code}"
        except Exception as e:
            self.result_label.text = f"🚨 Ошибка сети:\n{str(e)}"

    def get_roblox_status(self, user_id, user_info_str):
        url = "https://roblox.com"
        payload = {"userIds": [user_id]}
        try:
            response = requests.post(url, json=payload, timeout=5)
            if response.status_code == 200:
                presences = response.json().get("userPresences", [])
                if presences:
                    user_data = presences
                    status_code = user_data.get("userPresenceType", 0)
                    last_location = user_data.get("lastLocation", "Неизвестно")
                    place_id = user_data.get("placeId", None)
                    
                    if status_code == 0:
                        status_text = "🔴 Оффлайн"
                    elif status_code == 1:
                        status_text = "🟢 Онлайн (В меню)"
                    elif status_code == 2:
                        status_text = f"🎮 В игре!\n📌 Плейс: {last_location}\n🆔 Place ID: {place_id}"
                    elif status_code == 3:
                        status_text = "🛠 В Roblox Studio"
                    else:
                        status_text = "❓ Неизвестно"
                        
                    self.result_label.text = f"👤 {user_info_str}\n\nСтатус: {status_text}"
                else:
                    self.result_label.text = "⚠️ Не удалось получить статус."
            else:
                self.result_label.text = f"❌ Ошибка API: {response.status_code}"
        except Exception as e:
            self.result_label.text = f"🚨 Ошибка сети:\n{str(e)}"

if __name__ == '__main__':
    RobloxRadarApp().run()
