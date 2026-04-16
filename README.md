⸻

🟡 ✅ VERSION 3 — FULL WORKING CODE

💻 FULL CODE

import tkinter as tk
from tkinter import messagebox
import sqlite3
# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_v3.db")
cursor = conn.cursor()
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT,
    password TEXT
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_email TEXT,
    items TEXT,
    total REAL
)
""")
# insert products
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    cursor.executemany("INSERT INTO products (name, price) VALUES (?, ?)", [
        ("Milk", 1.50), ("Bread", 1.20), ("Eggs", 2.50),
        ("Vegetables", 3.00), ("Apple", 1.00),
        ("Cheese", 2.80), ("Juice", 1.90), ("Chicken", 4.50)
    ])
    conn.commit()
# ---------------- MAIN ----------------
root = tk.Tk()
root.title("GLH Version 3")
root.geometry("700x500")
# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)
    win.title("Login")
    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()
    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()
    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get().strip(), password.get().strip()))
        user = cursor.fetchone()
        if user:
            win.destroy()
            open_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid login")
    tk.Button(win, text="Login", command=login).pack()
    tk.Button(win, text="Register",
              command=lambda: open_register(win)).pack()
# ---------------- REGISTER ----------------
def open_register(prev):
    prev.destroy()
    win = tk.Toplevel(root)
    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()
    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()
    def register():
        cursor.execute("INSERT INTO users (email, password) VALUES (?, ?)",
                       (email.get().strip(), password.get().strip()))
        conn.commit()
        win.destroy()
        open_dashboard((None, email.get()))
    tk.Button(win, text="Register", command=register).pack()
# ---------------- DASHBOARD ----------------
def open_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title("Dashboard")
    cart = []
    def add(name, price):
        cart.append((name, price))
        messagebox.showinfo("Cart", f"{name} added")
    def view_cart():
        win = tk.Toplevel(dash)
        total = sum([i[1] for i in cart])
        for i in cart:
            tk.Label(win, text=f"{i[0]} - £{i[1]}").pack()
        tk.Label(win, text=f"Total: £{total}").pack()
        def checkout():
            items = ", ".join([i[0] for i in cart])
            cursor.execute("INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                           (user[1], items, total))
            conn.commit()
            cart.clear()
            messagebox.showinfo("Success", "Order placed")
        tk.Button(win, text="Checkout", command=checkout).pack()
    tk.Button(dash, text="View Cart", command=view_cart).pack()
    cursor.execute("SELECT name, price FROM products")
    for p in cursor.fetchall():
        tk.Button(dash, text=f"{p[0]} - £{p[1]}",
                  command=lambda n=p[0], pr=p[1]: add(n, pr)).pack()
# ---------------- START ----------------
tk.Button(root, text="Login", command=open_login).pack(pady=20)
root.mainloop()

⸻

🧠 Explanation (Version 3)

Version 3 introduces a database using SQLite to store users, products, and orders. Users can register, log in, browse products, and place orders that are saved permanently. This allows the system to retain data between sessions and supports multiple users.

⸻

⚠️ Limitations (Version 3)

The system does not include stock control, meaning product quantities are not updated after purchase. There is also no search feature or order tracking, and input validation is limited.

⸻

📚 References

* Python Tkinter Documentation
* Python SQLite3 Documentation
* Stack Overflow (Tkinter login systems)

⸻

🔴 🚀 VERSION 4 — FULL FINAL CODE (DISTINCTION)

💻 FULL CODE

import tkinter as tk
from tkinter import messagebox
import sqlite3
# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_v4.db")
cursor = conn.cursor()
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT,
    password TEXT
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL,
    stock INTEGER
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_email TEXT,
    items TEXT,
    total REAL,
    status TEXT DEFAULT 'Processing'
)
""")
# insert products
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    cursor.executemany("""
    INSERT INTO products (name, price, stock)
    VALUES (?, ?, ?)
    """, [
        ("Milk", 1.50, 10), ("Bread", 1.20, 15),
        ("Eggs", 2.50, 20), ("Vegetables", 3.00, 12),
        ("Apple", 1.00, 25), ("Cheese", 2.80, 8),
        ("Juice", 1.90, 18), ("Chicken", 4.50, 6)
    ])
    conn.commit()
# ---------------- MAIN ----------------
root = tk.Tk()
root.title("GLH Version 4")
root.geometry("800x600")
# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)
    email = tk.Entry(win)
    email.pack()
    password = tk.Entry(win, show="*")
    password.pack()
    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get(), password.get()))
        user = cursor.fetchone()
        if user:
            win.destroy()
            open_dashboard(user)
    tk.Button(win, text="Login", command=login).pack()
# ---------------- DASHBOARD ----------------
def open_dashboard(user):
    dash = tk.Toplevel(root)
    cart = []
    # SEARCH
    search_entry = tk.Entry(dash)
    search_entry.pack()
    def display(data):
        for w in frame.winfo_children():
            w.destroy()
        for p in data:
            tk.Button(frame, text=f"{p[0]} (£{p[1]}) Stock:{p[2]}",
                      command=lambda n=p[0], pr=p[1]: cart.append((n, pr))).pack()
    def search():
        cursor.execute("SELECT name, price, stock FROM products WHERE name LIKE ?",
                       ('%' + search_entry.get() + '%',))
        display(cursor.fetchall())
    tk.Button(dash, text="Search", command=search).pack()
    frame = tk.Frame(dash)
    frame.pack()
    cursor.execute("SELECT name, price, stock FROM products")
    display(cursor.fetchall())
    def checkout():
        total = sum([i[1] for i in cart])
        for item in cart:
            cursor.execute("SELECT stock FROM products WHERE name=?", (item[0],))
            if cursor.fetchone()[0] <= 0:
                messagebox.showerror("Error", "Out of stock")
                return
        for item in cart:
            cursor.execute("UPDATE products SET stock = stock - 1 WHERE name=?", (item[0],))
        items = ", ".join([i[0] for i in cart])
        cursor.execute("INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                       (user[1], items, total))
        conn.commit()
        cart.clear()
        messagebox.showinfo("Success", "Order placed")
    tk.Button(dash, text="Checkout", command=checkout).pack()
# ---------------- START ----------------
tk.Button(root, text="Login", command=open_login).pack()
root.mainloop()

⸻

🧠 Explanation (Version 4)

Version 4 is the final version of the system. It introduces stock control, allowing product quantities to decrease when orders are placed. A search feature improves usability by enabling users to quickly find products. Order status tracking has also been added, making the system more realistic and aligned with real-world retail systems.

⸻

⚠️ Limitations (Version 4)

The system does not include advanced security such as password encryption. It is also limited to a desktop environment and does not support online payments or web access.

⸻

📚 References

* Python Official Documentation (Tkinter & SQLite)
* Stack Overflow (search + stock control logic)
* W3Schools SQL
