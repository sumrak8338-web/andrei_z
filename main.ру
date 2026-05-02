import tkinter as tk
from tkinter import messagebox, ttk
import json
import os

class MovieLibrary(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Movie Library")
        self.geometry("600x400")

        # Инициализация списка фильмов
        self.movies = []
        self.load_movies()

        # Поля для ввода информации о фильме
        self.create_widgets()

    def create_widgets(self):
        # Параметры ввода
        labels = ["Название:", "Жанр:", "Год выпуска:", "Рейтинг (0-10):"]
        self.entries = []

        for label in labels:
            lbl = tk.Label(self, text=label)
            lbl.pack()
            entry = tk.Entry(self)
            entry.pack()
            self.entries.append(entry)

        # Кнопка добавления фильма
        add_button = tk.Button(self, text="Добавить фильм", command=self.add_movie)
        add_button.pack()

        # Поля фильтрации
        self.filter_genre_entry = tk.Entry(self, width=20)
        self.filter_genre_entry.pack()
        self.filter_genre_entry.insert(0, "Жанр для фильтрации")

        self.filter_year_entry = tk.Entry(self, width=20)
        self.filter_year_entry.pack()
        self.filter_year_entry.insert(0, "Год для фильтрации")

        filter_button = tk.Button(self, text="Применить фильтры", command=self.apply_filter)
        filter_button.pack()

        # Таблица для отображения фильмов
        self.tree = ttk.Treeview(self, columns=("Title", "Genre", "Year", "Rating"), show='headings')
        self.tree.heading("Title", text="Название")
        self.tree.heading("Genre", text="Жанр")
        self.tree.heading("Year", text="Год")
        self.tree.heading("Rating", text="Рейтинг")
        self.tree.pack(expand=True, fill=tk.BOTH)

        self.update_table()

    def add_movie(self):
        title = self.entries[0].get()
        genre = self.entries[1].get()
        year = self.entries[2].get()
        rating = self.entries[3].get()

        # Проверка корректности ввода
        if not title or not genre:
            messagebox.showerror("Ошибка", "Название и жанр обязательны для заполнения!")
            return

        if not year.isdigit() or int(year) < 1888:  # Первый фильм был выпущен в 1888
            messagebox.showerror("Ошибка", "Год должен быть числом и не может быть меньше 1888!")
            return

        try:
            rating = float(rating)
            if not (0 <= rating <= 10):
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Рейтинг должен быть числом от 0 до 10!")
            return

        movie = {
            "title": title,
            "genre": genre,
            "year": int(year),
            "rating": rating
        }
        self.movies.append(movie)
        self.save_movies()
        self.update_table()

        # Очистка полей ввода
        for entry in self.entries:
            entry.delete(0, tk.END)

    def apply_filter(self):
        genre_filter = self.filter_genre_entry.get()
        year_filter = self.filter_year_entry.get()

        filtered_movies = [movie for movie in self.movies if 
            (not genre_filter or genre_filter.lower() in movie["genre"].lower()) and
            (not year_filter or movie["year"] == int(year_filter))
        ]
        self.update_table(filtered_movies)

    def update_table(self, movies=None):
        for row in self.tree.get_children():
            self.tree.delete(row)

        if movies is None:
            movies = self.movies

        for movie in movies:
            self.tree.insert("", "end", values=(movie["title"], movie["genre"], movie["year"], movie["rating"]))

    def save_movies(self):
        with open("movies.json", "w") as f:
            json.dump(self.movies, f)

    def load_movies(self):
        if os.path.exists("movies.json"):
            with open
