import tkinter as tk
from tkinter import ttk, messagebox
import json
import random
import string
import os
from datetime import datetime

class PasswordGeneratorApp:
    def __init__(self, master):
        self.master = master
        self.master.title("Random Password Generator")
        self.history = []

        self.load_history()

        self.create_widgets()
        self.update_length_label()

    def create_widgets(self):
        tk.Label(self.master, text="Длина пароля:").grid(row=0, column=0, sticky='w')
        self.length_var = tk.IntVar(value=12)
        self.length_scale = ttk.Scale(self.master, from_=4, to_=32, orient='horizontal',
                                      command=self.update_length_label)
        self.length_scale.set(12)
        self.length_scale.grid(row=0, column=1, sticky='we')
        self.length_label = tk.Label(self.master, text="12")
        self.length_label.grid(row=0, column=2, sticky='w')

        self.include_digits = tk.BooleanVar(value=True)
        self.include_letters = tk.BooleanVar(value=True)
        self.include_symbols = tk.BooleanVar(value=False)

        tk.Checkbutton(self.master, text="Цифры", variable=self.include_digits).grid(row=1, column=0, sticky='w')
        tk.Checkbutton(self.master, text="Буквы", variable=self.include_letters).grid(row=1, column=1, sticky='w')
        tk.Checkbutton(self.master, text="Спецсимволы", variable=self.include_symbols).grid(row=1, column=2, sticky='w')

        self.generate_button = tk.Button(self.master, text="Генерировать пароль", command=self.generate_password)
        self.generate_button.grid(row=2, column=0, columnspan=3, pady=10)

        tk.Label(self.master, text="Пароль:").grid(row=3, column=0, sticky='w')
        self.password_entry = tk.Entry(self.master, width=50)
        self.password_entry.grid(row=3, column=1, columnspan=2, sticky='we')

        columns = ('Пароль', 'Длина', 'Дата')
        self.tree = ttk.Treeview(self.master, columns=columns, show='headings')
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        self.tree.grid(row=4, column=0, columnspan=3, pady=10)

        self.refresh_history()

    def update_length_label(self, event=None):
        length = int(self.length_scale.get())
        self.length_label.config(text=str(length))
        self.length_var.set(length)

    def load_history(self):
        if os.path.exists('history.json'):
            try:
                with open('history.json', 'r', encoding='utf-8') as f:
                    self.history = json.load(f)
            except json.JSONDecodeError:
                self.history = []
        else:
            self.history = []

    def save_history(self):
        with open('history.json', 'w', encoding='utf-8') as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)

    def refresh_history(self):
        for item in self.tree.get_children():
            self.tree.delete(item)
        for entry in self.history:
            self.tree.insert('', 'end', values=(entry['password'], entry['length'], entry['date']))

    def generate_password(self):
        length = int(self.length_scale.get())
        charset = ''
        if self.include_digits.get():
            charset += string.digits
        if self.include_letters.get():
            charset += string.ascii_letters
        if self.include_symbols.get():
            charset += string.punctuation

        if not charset:
            messagebox.showerror("Ошибка", "Выберите хотя бы один тип символов!")
            return

        password = ''.join(random.choice(charset) for _ in range(length))
        self.password_entry.delete(0, tk.END)
        self.password_entry.insert(0, password)

        self.history.append({
            'password': password,
            'length': length,
            'date': datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        })
        self.save_history()
        self.refresh_history()

if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGeneratorApp(root)
    root.mainloop()
