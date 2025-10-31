import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3

# --- Database setup ---
conn = sqlite3.connect("expenses.db")
cursor = conn.cursor()
cursor.execute("""
    CREATE TABLE IF NOT EXISTS expenses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        date TEXT,
        category TEXT,
        amount REAL,
        description TEXT
    )
""")
conn.commit()

# --- Add expense ---
def add_expense():
    date = date_entry.get()
    category = category_entry.get()
    amount = amount_entry.get()
    description = description_entry.get()
    
    if not (date and category and amount):
        messagebox.showwarning("Input Error", "Please fill in all required fields!")
        return
    
    cursor.execute("INSERT INTO expenses (date, category, amount, description) VALUES (?, ?, ?, ?)",
                   (date, category, amount, description))
    conn.commit()
    messagebox.showinfo("Success", "Expense added successfully!")
    clear_fields()
    view_expenses()

# --- View expenses ---
def view_expenses():
    for row in tree.get_children():
        tree.delete(row)
    cursor.execute("SELECT * FROM expenses")
    for row in cursor.fetchall():
        tree.insert("", tk.END, values=row)

# --- Delete selected expense ---
def delete_expense():
    selected_item = tree.selection()
    if not selected_item:
        messagebox.showwarning("Select item", "Please select an expense to delete.")
        return
    expense_id = tree.item(selected_item)["values"][0]
    cursor.execute("DELETE FROM expenses WHERE id=?", (expense_id,))
    conn.commit()
    view_expenses()

# --- Clear input fields ---
def clear_fields():
    date_entry.delete(0, tk.END)
    category_entry.delete(0, tk.END)
    amount_entry.delete(0, tk.END)
    description_entry.delete(0, tk.END)

# --- GUI setup ---
root = tk.Tk()
root.title("Expense Tracker")
root.geometry("700x500")

# Labels and Entries
tk.Label(root, text="Date (YYYY-MM-DD):").pack()
date_entry = tk.Entry(root)
date_entry.pack()

tk.Label(root, text="Category:").pack()
category_entry = tk.Entry(root)
category_entry.pack()

tk.Label(root, text="Amount:").pack()
amount_entry = tk.Entry(root)
amount_entry.pack()

tk.Label(root, text="Description:").pack()
description_entry = tk.Entry(root)
description_entry.pack()

# Buttons
tk.Button(root, text="Add Expense", command=add_expense, bg="#4CAF50", fg="white").pack(pady=5)
tk.Button(root, text="View Expenses", command=view_expenses, bg="#2196F3", fg="white").pack(pady=5)
tk.Button(root, text="Delete Selected", command=delete_expense, bg="#f44336", fg="white").pack(pady=5)

# Expense table
columns = ("ID", "Date", "Category", "Amount", "Description")
tree = ttk.Treeview(root, columns=columns, show="headings", height=10)
for col in columns:
    tree.heading(col, text=col)
tree.pack(pady=10)

view_expenses()

root.mainloop()
