import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
# using sqlite because its simple for this project
conn = sqlite3.connect("glh_final.db")
cursor = conn.cursor()

# create tables
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT,
    email TEXT,
    password TEXT,
    address TEXT,
    county TEXT
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

cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL,
    stock INTEGER
)
""")

# insert products first time
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    cursor.executemany("""
    INSERT INTO products (name, price, stock)
    VALUES (?, ?, ?)
    """, [
        ("Milk", 1.50, 10),
        ("Bread", 1.20, 15),
        ("Eggs", 2.50, 20),
        ("Vegetables", 3.00, 12),
        ("Apple", 1.00, 25),
        ("Cheese", 2.80, 8),
        ("Juice", 1.90, 18),
        ("Chicken", 4.50, 6)
    ])
    conn.commit()

# ---------------- MAIN WINDOW ----------------
root = tk.Tk()
root.title("Greenfield Local Hub")
root.geometry("750x550")
root.configure(bg="#e8f5e9")

# header
header = tk.Frame(root, bg="#2e7d32", height=80)
header.pack(fill="x")

tk.Label(header, text="Greenfield Local Hub",
         fg="white", bg="#2e7d32",
         font=("Arial", 18, "bold")).pack(pady=20)

# main card
card = tk.Frame(root, bg="white", bd=2, relief="raised")
card.place(relx=0.5, rely=0.5, anchor="center", width=420, height=350)

# logo (make sure file exists)
try:
    logo = tk.PhotoImage(file=r"C:\Users\TLevel-Digital-OS-11\Desktop\Version control\images\logo.png")
    lbl = tk.Label(card, image=logo, bg="white")
    lbl.image = logo
    lbl.pack(pady=5)
except:
    tk.Label(card, text="[Logo here]", bg="white").pack()

tk.Label(card, text="Welcome", font=("Arial", 14, "bold"), bg="white").pack()
tk.Label(card, text="Choose your role", bg="white").pack(pady=5)

# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)
    win.title("Login")
    win.geometry("350x300")

    tk.Label(win, text="Login", font=("Arial", 14)).pack(pady=10)

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        # had issue before so cleaned inputs
        e = email.get().strip().lower()
        p = password.get().strip()

        cursor.execute("SELECT * FROM users WHERE email=? AND password=?", (e, p))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid details")

    tk.Button(win, text="Login", bg="#4CAF50", fg="white",
              command=login).pack(pady=10)

    tk.Button(win, text="Register",
              command=lambda: open_register(win)).pack()

# ---------------- REGISTER ----------------
def open_register(prev):
    prev.destroy()

    win = tk.Toplevel(root)
    win.title("Register")
    win.geometry("350x450")

    fields = {}
    labels = ["Username", "Email", "Password", "Address", "County"]

    for l in labels:
        tk.Label(win, text=l).pack()
        ent = tk.Entry(win, show="*" if l=="Password" else None)
        ent.pack()
        fields[l] = ent

    def register():
        data = {k: v.get().strip() for k, v in fields.items()}

        if "" in data.values():
            messagebox.showerror("Error", "Fill all fields")
            return

        cursor.execute("SELECT * FROM users WHERE email=?", (data["Email"],))
        if cursor.fetchone():
            messagebox.showerror("Error", "Email exists")
            return

        cursor.execute("""
        INSERT INTO users (username, email, password, address, county)
        VALUES (?, ?, ?, ?, ?)
        """, (data["Username"], data["Email"].lower(),
              data["Password"], data["Address"], data["County"]))

        conn.commit()

        win.destroy()
        open_customer_dashboard((None, data["Username"], data["Email"]))

    tk.Button(win, text="Register", bg="#4CAF50", fg="white",
              command=register).pack(pady=15)

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title("Customer Dashboard")
    dash.geometry("950x650")

    user_email = user[2]
    cart = []

    # header
    top = tk.Frame(dash, bg="#2e7d32")
    top.pack(fill="x")

    tk.Label(top, text="GLH Store", fg="white", bg="#2e7d32").pack(side="left")

    # cart window
    def view_cart():
        win = tk.Toplevel(dash)
        total = sum([i[1] for i in cart])

        for i in cart:
            tk.Label(win, text=f"{i[0]} - £{i[1]}").pack()

        tk.Label(win, text=f"Total £{total}").pack()

        def checkout():
            if not cart:
                messagebox.showerror("Error", "Cart empty")
                return

            items = ", ".join([i[0] for i in cart])

            cursor.execute("INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                           (user_email, items, total))
            conn.commit()

            cart.clear()
            messagebox.showinfo("Success", "Order placed")

        tk.Button(win, text="Checkout", command=checkout).pack()

    tk.Button(top, text="Cart", command=view_cart).pack(side="right")

    # orders
    def view_orders():
        win = tk.Toplevel(dash)

        cursor.execute("SELECT items, total FROM orders WHERE user_email=?", (user_email,))
        data = cursor.fetchall()

        if not data:
            tk.Label(win, text="No orders yet").pack()
        else:
            for d in data:
                tk.Label(win, text=f"{d[0]} | £{d[1]}").pack()

    tk.Button(top, text="Orders", command=view_orders).pack(side="right")

    # product display
    main = tk.Frame(dash)
    main.pack(pady=20)

    cursor.execute("SELECT name, price FROM products")
    products = cursor.fetchall()

    def add_to_cart(n, p):
        cart.append((n, p))
        messagebox.showinfo("Cart", f"{n} added")

    r = c = 0
    for p in products:
        box = tk.Frame(main, bd=1, relief="solid")
        box.grid(row=r, column=c, padx=15, pady=15)

        tk.Label(box, text=p[0]).pack()
        tk.Label(box, text=f"£{p[1]}", fg="green").pack()

        tk.Button(box, text="Add",
                  command=lambda n=p[0], pr=p[1]: add_to_cart(n, pr)).pack()

        c += 1
        if c == 3:
            c = 0
            r += 1

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Producer Dashboard")
    dash.geometry("800x500")

    def view_products():
        win = tk.Toplevel(dash)
        cursor.execute("SELECT name, price, stock FROM products")
        for p in cursor.fetchall():
            tk.Label(win, text=f"{p[0]} | £{p[1]} | Stock: {p[2]}").pack()

    def update_stock():
        win = tk.Toplevel(dash)

        tk.Label(win, text="Product name").pack()
        name = tk.Entry(win)
        name.pack()

        tk.Label(win, text="New stock").pack()
        stock = tk.Entry(win)
        stock.pack()

        def update():
            cursor.execute("UPDATE products SET stock=? WHERE name=?",
                           (stock.get(), name.get()))
            conn.commit()
            messagebox.showinfo("Done", "Stock updated")

        tk.Button(win, text="Update", command=update).pack()

    def view_orders():
        win = tk.Toplevel(dash)
        cursor.execute("SELECT user_email, items, total FROM orders")
        for o in cursor.fetchall():
            tk.Label(win, text=f"{o[0]} | {o[1]} | £{o[2]}").pack()

    tk.Button(dash, text="View Products", command=view_products).pack(pady=10)
    tk.Button(dash, text="Update Stock", command=update_stock).pack(pady=10)
    tk.Button(dash, text="View Orders", command=view_orders).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(card, text="Customer", bg="#4CAF50", fg="white",
          width=20, command=open_login).pack(pady=10)

tk.Button(card, text="Producer", bg="#1976D2", fg="white",
          width=20, command=open_producer_dashboard).pack()

root.mainloop()
