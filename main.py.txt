import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker - Трекер прочитанных книг")
        self.root.geometry("900x600")
       
        self.books_file = "books.json"
        self.books = self.load_books()
       
        self.create_widgets()
        self.display_books()
   
    def create_widgets(self):
        # Рамка для ввода данных
        input_frame = ttk.LabelFrame(self.root, text="Добавление книги", padding="10")
        input_frame.pack(fill=tk.X, padx=10, pady=5)
       
        # Поля ввода
        ttk.Label(input_frame, text="Название книги:").grid(row=0, column=0, sticky=tk.W, padx=5, pady=5)
        self.title_entry = ttk.Entry(input_frame, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)
       
        ttk.Label(input_frame, text="Автор:").grid(row=0, column=2, sticky=tk.W, padx=5, pady=5)
        self.author_entry = ttk.Entry(input_frame, width=25)
        self.author_entry.grid(row=0, column=3, padx=5, pady=5)
       
        ttk.Label(input_frame, text="Жанр:").grid(row=1, column=0, sticky=tk.W, padx=5, pady=5)
        self.genre_entry = ttk.Entry(input_frame, width=30)
        self.genre_entry.grid(row=1, column=1, padx=5, pady=5)
       
        ttk.Label(input_frame, text="Кол-во страниц:").grid(row=1, column=2, sticky=tk.W, padx=5, pady=5)
        self.pages_entry = ttk.Entry(input_frame, width=25)
        self.pages_entry.grid(row=1, column=3, padx=5, pady=5)
       
        # Кнопка добавления
        self.add_button = ttk.Button(input_frame, text="Добавить книгу", command=self.add_book)
        self.add_button.grid(row=2, column=0, columnspan=4, pady=10)
       
        # Рамка для фильтрации
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация", padding="10")
        filter_frame.pack(fill=tk.X, padx=10, pady=5)
       
        ttk.Label(filter_frame, text="Фильтр по жанру:").pack(side=tk.LEFT, padx=5)
        self.filter_genre_entry = ttk.Entry(filter_frame, width=20)
        self.filter_genre_entry.pack(side=tk.LEFT, padx=5)
       
        self.filter_genre_button = ttk.Button(filter_frame, text="Применить фильтр", command=self.filter_by_genre)
        self.filter_genre_button.pack(side=tk.LEFT, padx=5)
       
        ttk.Button(filter_frame, text="Сбросить фильтр", command=self.display_books).pack(side=tk.LEFT, padx=20)
       
        ttk.Label(filter_frame, text="Страниц >").pack(side=tk.LEFT, padx=5)
        self.filter_pages_entry = ttk.Entry(filter_frame, width=10)
        self.filter_pages_entry.pack(side=tk.LEFT, padx=5)
       
        self.filter_pages_button = ttk.Button(filter_frame, text="Применить", command=self.filter_by_pages)
        self.filter_pages_button.pack(side=tk.LEFT, padx=5)
       
        ttk.Button(filter_frame, text="Сбросить всё", command=self.reset_filters).pack(side=tk.LEFT, padx=20)
       
        # Таблица с книгами
        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
       
        columns = ("title", "author", "genre", "pages")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings")
       
        self.tree.heading("title", text="Название")
        self.tree.heading("author", text="Автор")
        self.tree.heading("genre", text="Жанр")
        self.tree.heading("pages", text="Страниц")
       
        self.tree.column("title", width=250)
        self.tree.column("author", width=200)
        self.tree.column("genre", width=150)
        self.tree.column("pages", width=100)
       
        scrollbar = ttk.Scrollbar(table_frame, orient=tk.VERTICAL, command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
       
        self.tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
       
        # Кнопка удаления
        delete_frame = ttk.Frame(self.root)
        delete_frame.pack(fill=tk.X, padx=10, pady=5)
       
        self.delete_button = ttk.Button(delete_frame, text="Удалить выбранную книгу", command=self.delete_book)
        self.delete_button.pack(side=tk.LEFT, padx=5)
       
        self.clear_all_button = ttk.Button(delete_frame, text="Очистить всё избранное", command=self.clear_all)
        self.clear_all_button.pack(side=tk.LEFT, padx=5)
       
        # Счётчик книг
        self.count_label = ttk.Label(self.root, text="Всего книг: 0")
        self.count_label.pack(side=tk.BOTTOM, pady=5)
       
        self.update_count()
   
    def add_book(self):
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_entry.get().strip()
        pages = self.pages_entry.get().strip()
       
        # Проверка на пустые поля
        if not title or not author or not genre or not pages:
            messagebox.showwarning("Предупреждение", "Все поля должны быть заполнены!")
            return
       
        # Проверка, что страницы — число
        try:
            pages = int(pages)
            if pages <= 0:
                raise ValueError
        except ValueError:
            messagebox.showwarning("Предупреждение", "Количество страниц должно быть положительным числом!")
            return
       
        # Добавление книги
        book = {
            "title": title,
            "author": author,
            "genre": genre,
            "pages": pages
        }
        self.books.append(book)
        self.save_books()
        self.display_books()
       
        # Очистка полей
        self.title_entry.delete(0, tk.END)
        self.author_entry.delete(0, tk.END)
        self.genre_entry.delete(0, tk.END)
        self.pages_entry.delete(0, tk.END)
       
        messagebox.showinfo("Успех", f"Книга '{title}' добавлена!")
   
    def display_books(self, books_to_show=None):
        for item in self.tree.get_children():
            self.tree.delete(item)
       
        if books_to_show is None:
            books_to_show = self.books
       
        for book in books_to_show:
            self.tree.insert("", tk.END, values=(
                book["title"],
                book["author"],
                book["genre"],
                book["pages"]
            ))
       
        self.update_count()
   
    def filter_by_genre(self):
        genre = self.filter_genre_entry.get().strip().lower()
        if not genre:
            messagebox.showwarning("Предупреждение", "Введите жанр для фильтрации!")
            return
       
        filtered = [book for book in self.books if genre in book["genre"].lower()]
        self.display_books(filtered)
        messagebox.showinfo("Результат", f"Найдено книг: {len(filtered)}")
   
    def filter_by_pages(self):
        pages_str = self.filter_pages_entry.get().strip()
        if not pages_str:
            messagebox.showwarning("Предупреждение", "Введите количество страниц!")
            return
       
        try:
            pages = int(pages_str)
        except ValueError:
            messagebox.showwarning("Предупреждение", "Введите число!")
            return
       
        filtered = [book for book in self.books if book["pages"] > pages]
        self.display_books(filtered)
        messagebox.showinfo("Результат", f"Найдено книг: {len(filtered)}")
   
    def reset_filters(self):
        self.filter_genre_entry.delete(0, tk.END)
        self.filter_pages_entry.delete(0, tk.END)
        self.display_books()
   
    def delete_book(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Выберите книгу для удаления!")
            return
       
        item = self.tree.item(selected[0])
        values = item["values"]
        title = values[0]
       
        if messagebox.askyesno("Подтверждение", f"Удалить книгу '{title}'?"):
            for i, book in enumerate(self.books):
                if book["title"] == title and book["author"] == values[1]:
                    del self.books[i]
                    break
            self.save_books()
            self.display_books()
   
    def clear_all(self):
        if messagebox.askyesno("Подтверждение", "Удалить все книги?"):
            self.books = []
            self.save_books()
            self.display_books()
   
    def update_count(self):
        self.count_label.config(text=f"Всего книг: {len(self.books)}")
   
    def load_books(self):
        if os.path.exists(self.books_file):
            try:
                with open(self.books_file, "r", encoding="utf-8") as f:
                    return json.load(f)
            except (json.JSONDecodeError, FileNotFoundError):
                return []
        return []
   
    def save_books(self):
        with open(self.books_file, "w", encoding="utf-8") as f:
            json.dump(self.books, f, ensure_ascii=False, indent=2)

if __name__ == "__main__":
    root = tk.Tk()
    app = BookTracker(root)
    root.mainloop()