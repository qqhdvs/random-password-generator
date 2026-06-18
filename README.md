# random-password-generator
import tkinter as tk
from tkinter import ttk, messagebox
import random
import string
import json
import os

HISTORY_FILE = "password_history.json"

class PasswordGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Password Generator")
        self.root.geometry("550x450")
        self.root.resizable(False, False)
        
        # Переменные для настроек
        self.use_digits = tk.BooleanVar(value=True)
        self.use_letters = tk.BooleanVar(value=True)
        self.use_specials = tk.BooleanVar(value=False)
        self.length_var = tk.IntVar(value=12)
        
        self.create_widgets()
        self.load_history()

    def create_widgets(self):
        # 1. Элементы интерфейса: Настройки
        settings_frame = tk.LabelFrame(self.root, text=" Настройки пароля ", padx=10, pady=10)
        settings_frame.pack(fill="x", padx=15, pady=10)
        
        # Ползунок длины пароля
        tk.Label(settings_frame, text="Длина пароля:").grid(row=0, column=0, sticky="w")
        self.slider = tk.Scale(settings_frame, from_=4, to=32, orient="horizontal", variable=self.length_var, length=200)
        self.slider.grid(row=0, column=1, padx=10, columnspan=2)
        
        # Чекбоксы для выбора символов
        tk.Checkbutton(settings_frame, text="Цифры (0-9)", variable=self.use_digits).grid(row=1, column=0, sticky="w", pady=5)
        tk.Checkbutton(settings_frame, text="Буквы (a-z)", variable=self.use_letters).grid(row=1, column=1, sticky="w", pady=5)
        tk.Checkbutton(settings_frame, text="Спецсимволы", variable=self.use_specials).grid(row=1, column=2, sticky="w", pady=5)
        
        # Кнопка генерации
        self.gen_btn = tk.Button(self.root, text="Сгенерировать пароль", bg="#28a745", fg="white", font=("Arial", 11, "bold"), command=self.generate_password)
        self.gen_btn.pack(fill="x", padx=15, pady=5)
        
        # Поле вывода
        self.result_entry = tk.Entry(self.root, font=("Arial", 14), justify="center")
        self.result_entry.pack(fill="x", padx=15, pady=10)
        
        # Таблица истории
        history_frame = tk.LabelFrame(self.root, text=" История генераций ", padx=10, pady=10)
        history_frame.pack(fill="both", expand=True, padx=15, pady=10)
        
        columns = ("Индекс", "Пароль", "Длина")
        self.tree = ttk.Treeview(history_frame, columns=columns, show="headings", height=6)
        self.tree.heading("Индекс", text="№")
        self.tree.heading("Пароль", text="Пароль")
        self.tree.heading("Длина", text="Длина")
        
        self.tree.column("Индекс", width=40, anchor="center")
        self.tree.column("Пароль", width=350, anchor="w")
        self.tree.column("Длина", width=60, anchor="center")
        
        scrollbar = ttk.Scrollbar(history_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

    def generate_password(self):
        length = self.length_var.get()
        
        # 4. Проверка корректности ввода (минимальная/максимальная длина)
        if length < 4 or length > 32:
            messagebox.showerror("Ошибка", "Длина должна быть от 4 до 32 символов!")
            return
            
        char_pool = ""
        if self.use_digits.get(): char_pool += string.digits
        if self.use_letters.get(): char_pool += string.ascii_letters
        if self.use_specials.get(): char_pool += string.punctuation
            
        if not char_pool:
            messagebox.showwarning("Внимание", "Выберите хотя бы один тип символов!")
            return
            
        # 2. Использование библиотеки random для генерации
        password = "".join(random.choice(char_pool) for _ in range(length))
        
        self.result_entry.delete(0, tk.END)
        self.result_entry.insert(0, password)
        self.save_to_history(password)

    # 3. Сохранение истории в JSON и загрузка обратно
    def save_to_history(self, password):
        history = []
        if os.path.exists(HISTORY_FILE):
            try:
                with open(HISTORY_FILE, "r", encoding="utf-8") as f:
                    history = json.load(f)
            except:
                history = []
        history.append({"password": password, "length": len(password)})
        with open(HISTORY_FILE, "w", encoding="utf-8") as f:
            json.dump(history, f, indent=4, ensure_ascii=False)
        self.update_treeview(history)

    def load_history(self):
        if os.path.exists(HISTORY_FILE):
            try:
                with open(HISTORY_FILE, "r", encoding="utf-8") as f:
                    self.update_treeview(json.load(f))
            except:
                pass

    def update_treeview(self, history):
        for item in self.tree.get_children():
            self.tree.delete(item)
        for idx, entry in enumerate(history, start=1):
            self.tree.insert("", "end", values=(idx, entry["password"], entry["length"]))

if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGeneratorApp(root)
    root.mainloop()
