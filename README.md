```python
import json
import uuid
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
from datetime import datetime

class WeatherDiaryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.root.geometry("800x600")
        
        # Данные: список всех записей
        self.entries = []          # каждый элемент: dict с ключами id, date, temperature, description, precipitation
        self.max_id = 0            # для генерации новых ID
        
        # Файл по умолчанию для автосохранения/автозагрузки
        self.default_file = "weather_diary.json"
        
        # Создание интерфейса
        self.create_widgets()
        
        # Загрузка данных из файла по умолчанию при старте
        self.load_from_file(self.default_file)
        self.refresh_display()
    
    def create_widgets(self):
        # --- Панель ввода новой записи ---
        input_frame = ttk.LabelFrame(self.root, text="Добавить запись", padding=5)
        input_frame.pack(fill="x", padx=5, pady=5)
        
        # Дата
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="e", padx=5, pady=2)
        self.date_var = tk.StringVar()
        self.date_entry = ttk.Entry(input_frame, textvariable=self.date_var, width=15)
        self.date_entry.grid(row=0, column=1, padx=5, pady=2)
        
        # Температура
        ttk.Label(input_frame, text="Температура (°C):").grid(row=0, column=2, sticky="e", padx=5, pady=2)
        self.temp_var = tk.StringVar()
        self.temp_entry = ttk.Entry(input_frame, textvariable=self.temp_var, width=10)
        self.temp_entry.grid(row=0, column=3, padx=5, pady=2)
        
        # Описание
        ttk.Label(input_frame, text="Описание:").grid(row=0, column=4, sticky="e", padx=5, pady=2)
        self.desc_var = tk.StringVar()
        self.desc_entry = ttk.Entry(input_frame, textvariable=self.desc_var, width=20)
        self.desc_entry.grid(row=0, column=5, padx=5, pady=2)
        
        # Осадки
        self.precip_var = tk.BooleanVar()
        self.precip_check = ttk.Checkbutton(input_frame, text="Осадки", variable=self.precip_var)
        self.precip_check.grid(row=0, column=6, padx=5, pady=2)
        
        # Кнопка добавления
        self.add_btn = ttk.Button(input_frame, text="Добавить запись", command=self.add_entry)
        self.add_btn.grid(row=0, column=7, padx=10, pady=2)
        
        # --- Панель фильтров ---
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация", padding=5)
        filter_frame.pack(fill="x", padx=5, pady=5)
        
        ttk.Label(filter_frame, text="Фильтр по дате (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="e", padx=5, pady=2)
        self.filter_date_var = tk.StringVar()
        self.filter_date_entry = ttk.Entry(filter_frame, textvariable=self.filter_date_var, width=15)
        self.filter_date_entry.grid(row=0, column=1, padx=5, pady=2)
        self.filter_date_btn = ttk.Button(filter_frame, text="Применить", command=self.apply_filters)
        self.filter_date_btn.grid(row=0, column=2, padx=2, pady=2)
        
        ttk.Label(filter_frame, text="Температура выше (°C):").grid(row=0, column=3, sticky="e", padx=5, pady=2)
        self.filter_temp_var = tk.StringVar()
        self.filter_temp_entry = ttk.Entry(filter_frame, textvariable=self.filter_temp_var, width=10)
        self.filter_temp_entry.grid(row=0, column=4, padx=5, pady=2)
        self.filter_temp_btn = ttk.Button(filter_frame, text="Применить", command=self.apply_filters)
        self.filter_temp_btn.grid(row=0, column=5, padx=2, pady=2)
        
        self.reset_filters_btn = ttk.Button(filter_frame, text="Сбросить фильтры", command=self.reset_filters)
        self.reset_filters_btn.grid(row=0, column=6, padx=10, pady=2)
        
        # --- Таблица для отображения записей ---
        tree_frame = ttk.Frame(self.root)
        tree_frame.pack(fill="both", expand=True, padx=5, pady=5)
        
        # Scrollbar
        scrollbar = ttk.Scrollbar(tree_frame)
        scrollbar.pack(side="right", fill="y")
        
        # Treeview с колонками (id скрыта)
        columns = ("id", "date", "temperature", "description", "precipitation")
        self.tree = ttk.Treeview(tree_frame, columns=columns, show="headings", yscrollcommand=scrollbar.set)
        scrollbar.config(command=self.tree.yview)
        
        # Определение заголовков
        self.tree.heading("id", text="ID")
        self.tree.heading("date", text="Дата")
        self.tree.heading("temperature", text="Температура (°C)")
        self.tree.heading("description", text="Описание")
        self.tree.heading("precipitation", text="Осадки")
        
        # Настройка ширины колонок (id скрываем)
        self.tree.column("id", width=0, stretch=False)
        self.tree.column("date", width=100)
        self.tree.column("temperature", width=100)
        self.tree.column("description", width=200)
        self.tree.column("precipitation", width=80)
        
        self.tree.pack(fill="both", expand=True)
        
        # --- Нижняя панель с кнопками управления файлами и удалением ---
        control_frame = ttk.Frame(self.root)
        control_frame.pack(fill="x", padx=5, pady=5)
        
        self.save_btn = ttk.Button(control_frame, text="Сохранить в файл...", command=self.save_to_file_dialog)
        self.save_btn.pack(side="left", padx=5)
        
        self.load_btn = ttk.Button(control_frame, text="Загрузить из файла...", command=self.load_from_file_dialog)
        self.load_btn.pack(side="left", padx=5)
        
        self.delete_btn = ttk.Button(control_frame, text="Удалить выбранную запись", command=self.delete_entry)
        self.delete_btn.pack(side="right", padx=5)
        
    # ---------- Работа с записями ----------
    def add_entry(self):
        """Добавление новой записи после проверки корректности"""
        date_str = self.date_var.get().strip()
        temp_str = self.temp_var.get().strip()
        description = self.desc_var.get().strip()
        precipitation = self.precip_var.get()
        
        # Проверка даты
        if not self.is_valid_date(date_str):
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте ГГГГ-ММ-ДД")
            return
        
        # Проверка температуры
        try:
            temperature = float(temp_str)
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом")
            return
        
        # Генерация уникального ID
        self.max_id += 1
        new_id = self.max_id
        
        # Создание записи
        entry = {
            "id": new_id,
            "date": date_str,
            "temperature": temperature,
            "description": description,
            "precipitation": precipitation
        }
        self.entries.append(entry)
        
        # Автосохранение в файл по умолчанию
        self.save_to_file(self.default_file)
        
        # Очистка полей ввода
        self.date_var.set("")
        self.temp_var.set("")
        self.desc_var.set("")
        self.precip_var.set(False)
        
        # Обновление отображения
        self.refresh_display()
        messagebox.showinfo("Успех", "Запись добавлена")
    
    def delete_entry(self):
        """Удаление выбранной записи из таблицы"""
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Выберите запись для удаления")
            return
        
        # Получаем ID записи из скрытой колонки
        item = self.tree.item(selected[0])
        record_id = item["values"][0]   # первая колонка id
        
        # Находим и удаляем запись
        for i, entry in enumerate(self.entries):
            if entry["id"] == record_id:
                del self.entries[i]
                break
        
        # Сохраняем изменения и обновляем отображение
        self.save_to_file(self.default_file)
        self.refresh_display()
        messagebox.showinfo("Успех", "Запись удалена")
    
    # ---------- Фильтрация и отображение ----------
    def apply_filters(self):
        """Применяет текущие фильтры и обновляет таблицу"""
        date_filter = self.filter_date_var.get().strip()
        temp_filter_str = self.filter_temp_var.get().strip()
        
        # Проверка даты фильтра (если введена)
        if date_filter and not self.is_valid_date(date_filter):
            messagebox.showerror("Ошибка", "Неверный формат даты в фильтре. Используйте ГГГГ-ММ-ДД")
            return
        
        # Проверка температурного фильтра
        temp_threshold = None
        if temp_filter_str:
            try:
                temp_threshold = float(temp_filter_str)
            except ValueError:
                messagebox.showerror("Ошибка", "Температурный фильтр должен быть числом")
                return
        
        # Очистка таблицы
        for row in self.tree.get_children():
            self.tree.delete(row)
        
        # Фильтрация и вставка
        for entry in self.entries:
            # Фильтр по дате
            if date_filter and entry["date"] != date_filter:
                continue
            # Фильтр по температуре (выше порога)
            if temp_threshold is not None and entry["temperature"] <= temp_threshold:
                continue
            
            # Вставка строки
            precip_text = "Да" if entry["precipitation"] else "Нет"
            self.tree.insert("", "end", values=(
                entry["id"],
                entry["date"],
                entry["temperature"],
                entry["description"],
                precip_text
            ))
    
    def reset_filters(self):
        """Сбрасывает фильтры и показывает все записи"""
        self.filter_date_var.set("")
        self.filter_temp_var.set("")
        self.apply_filters()
    
    def refresh_display(self):
        """Обновляет отображение (применяет текущие фильтры)"""
        self.apply_filters()
    
    # ---------- Работа с JSON файлами ----------
    def save_to_file(self, filename):
        """Сохраняет текущие записи в указанный JSON файл"""
        try:
            with open(filename, "w", encoding="utf-8") as f:
                json.dump(self.entries, f, ensure_ascii=False, indent=2)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить файл:\n{e}")
    
    def load_from_file(self, filename):
        """Загружает записи из JSON файла и обновляет ID"""
        try:
            with open(filename, "r", encoding="utf-8") as f:
                data = json.load(f)
            if isinstance(data, list):
                self.entries = data
                # Восстанавливаем max_id на основе максимального ID в записях
                if self.entries:
                    max_id = max(entry.get("id", 0) for entry in self.entries)
                    self.max_id = max_id
                else:
                    self.max_id = 0
                self.refresh_display()
            else:
                messagebox.showwarning("Предупреждение", "Файл не содержит список записей")
        except FileNotFoundError:
            # Файл не существует — ничего не загружаем
            pass
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось загрузить файл:\n{e}")
    
    def save_to_file_dialog(self):
        """Диалог для сохранения в выбранный файл"""
        filename = filedialog.asksaveasfilename(
            defaultextension=".json",
            filetypes=[("JSON files", "*.json"), ("All files", "*.*")]
        )
        if filename:
            self.save_to_file(filename)
            messagebox.showinfo("Успех", "Данные сохранены")
    
    def load_from_file_dialog(self):
        """Диалог для загрузки из выбранного файла"""
        filename = filedialog.askopenfilename(
            filetypes=[("JSON files", "*.json"), ("All files", "*.*")]
        )
        if filename:
            self.load_from_file(filename)
            # После загрузки из внешнего файла автоматически сохраняем в файл по умолчанию
            self.save_to_file(self.default_file)
            messagebox.showinfo("Успех", "Данные загружены")
    
    # ---------- Вспомогательные функции ----------
    @staticmethod
    def is_valid_date(date_str):
        """Проверяет строку на соответствие формату ГГГГ-ММ-ДД"""
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
            return True
        except ValueError:
            return False

if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiaryApp(root)
    root.mainloop()
```
