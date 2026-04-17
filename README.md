version 4
import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_final.db")
cursor = conn.cursor()

# tables
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

# insert products
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

# ---------------- MAIN ----------------
root = tk.Tk()
root.title("Greenfield Local Hub - Version 4")
root.geometry("800x600")
root.configure(bg="#e8f5e9")

# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get().strip().lower(), password.get().strip()))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid login")

    tk.Button(win, text="Login", command=login).pack()

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title("Customer Dashboard")
    dash.geometry("1000x700")

    user_email = user[2]
    cart = []

    # HEADER
    header = tk.Frame(dash, bg="#2e7d32")
    header.pack(fill="x")

    tk.Label(header, text="GLH Store", fg="white", bg="#2e7d32",
             font=("Arial", 16)).pack(side="left", padx=10)

    # SEARCH BAR
    search_entry = tk.Entry(header)
    search_entry.pack(side="left", padx=10)

    def search_products():
        keyword = search_entry.get()
        cursor.execute("SELECT name, price FROM products WHERE name LIKE ?",
                       ('%' + keyword + '%',))
        display_products(cursor.fetchall())

    tk.Button(header, text="Search", command=search_products).pack(side="left")

    # CART
    def view_cart():
        win = tk.Toplevel(dash)
        total = sum([i[1] for i in cart])

        for i in cart:
            tk.Label(win, text=f"{i[0]} - £{i[1]}").pack()

        tk.Label(win, text=f"Total £{total}").pack()

    tk.Button(header, text="Cart", command=view_cart).pack(side="right")

    # PRODUCTS AREA
    main = tk.Frame(dash)
    main.pack(pady=20)

    # LOAD IMAGES (make sure files exist)
    images = {}
    for name in ["Milk","Bread","Eggs","Vegetables","Apple","Cheese","Juice","Chicken"]:
        try:
            img = tk.PhotoImage(file=f"images/{name.lower()}.png")
        except:
            img = None
        images[name] = img

    def add_to_cart(n, p):
        cart.append((n, p))
        messagebox.showinfo("Cart", f"{n} added")

    def display_products(products):
        for widget in main.winfo_children():
            widget.destroy()

        r = c = 0
        for p in products:
            box = tk.Frame(main, bd=1, relief="solid")
            box.grid(row=r, column=c, padx=15, pady=15)

            # IMAGE instead of text
            if images[p[0]]:
                lbl = tk.Label(box, image=images[p[0]])
                lbl.image = images[p[0]]
                lbl.pack()
            else:
                tk.Label(box, text=p[0]).pack()

            tk.Label(box, text=f"£{p[1]}", fg="green").pack()

            tk.Button(box, text="Add",
                      command=lambda n=p[0], pr=p[1]: add_to_cart(n, pr)).pack()

            c += 1
            if c == 3:
                c = 0
                r += 1

    cursor.execute("SELECT name, price FROM products")
    display_products(cursor.fetchall())

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Producer Dashboard")
    dash.geometry("900x600")

    title = tk.Label(dash, text="Producer Dashboard",
                     font=("Arial", 18, "bold"))
    title.pack(pady=10)

    frame = tk.Frame(dash)
    frame.pack(pady=20)

    def view_products():
        for w in frame.winfo_children():
            w.destroy()

        cursor.execute("SELECT name, price, stock FROM products")
        for p in cursor.fetchall():
            box = tk.Frame(frame, bd=1, relief="solid", padx=10, pady=5)
            box.pack(pady=5, fill="x")

            tk.Label(box, text=f"{p[0]}").pack(side="left")
            tk.Label(box, text=f"£{p[1]}").pack(side="left", padx=20)
            tk.Label(box, text=f"Stock: {p[2]}").pack(side="right")

    def update_stock():
        win = tk.Toplevel(dash)

        tk.Label(win, text="Product").pack()
        name = tk.Entry(win)
        name.pack()

        tk.Label(win, text="Stock").pack()
        stock = tk.Entry(win)
        stock.pack()

        def update():
            try:
                value = int(stock.get())
                cursor.execute("UPDATE products SET stock=? WHERE name=?",
                               (value, name.get()))
                conn.commit()
                messagebox.showinfo("Done", "Updated")
            except:
                messagebox.showerror("Error", "Enter valid number")

        tk.Button(win, text="Update", command=update).pack()

    tk.Button(dash, text="View Products", command=view_products).pack(pady=10)
    tk.Button(dash, text="Update Stock", command=update_stock).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(root, text="Customer", command=open_login).pack(pady=20)
tk.Button(root, text="Producer", command=open_producer_dashboard).pack()

root.mainloop()


version 5 
import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_v5.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT,
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
root.title("GLH Version 5")
root.geometry("900x650")

# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get().strip().lower(), password.get().strip()))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid login")

    tk.Button(win, text="Login", command=login).pack()

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title("Customer Dashboard")

    user_email = user[2]
    cart = []

    # SEARCH
    search = tk.Entry(dash)
    search.pack()

    frame = tk.Frame(dash)
    frame.pack()

    def display(data):
        for w in frame.winfo_children():
            w.destroy()

        for p in data:
            tk.Button(frame, text=f"{p[0]} (£{p[1]}) Stock:{p[2]}",
                      command=lambda n=p[0], pr=p[1]: cart.append((n, pr))).pack()

    def search_products():
        cursor.execute("SELECT name, price, stock FROM products WHERE name LIKE ?",
                       ('%' + search.get() + '%',))
        display(cursor.fetchall())

    tk.Button(dash, text="Search", command=search_products).pack()

    cursor.execute("SELECT name, price, stock FROM products")
    display(cursor.fetchall())

    # CART
    def checkout():
        if not cart:
            messagebox.showerror("Error", "Cart empty")
            return

        total = sum([i[1] for i in cart])

        # STOCK CHECK
        for item in cart:
            cursor.execute("SELECT stock FROM products WHERE name=?", (item[0],))
            if cursor.fetchone()[0] <= 0:
                messagebox.showerror("Error", f"{item[0]} out of stock")
                return

        # REDUCE STOCK
        for item in cart:
            cursor.execute("UPDATE products SET stock = stock - 1 WHERE name=?", (item[0],))

        items = ", ".join([i[0] for i in cart])

        cursor.execute("INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                       (user_email, items, total))
        conn.commit()

        cart.clear()
        messagebox.showinfo("Success", "Order placed")

    tk.Button(dash, text="Checkout", command=checkout).pack(pady=10)

    # VIEW ORDERS WITH STATUS
    def view_orders():
        win = tk.Toplevel(dash)

        cursor.execute("SELECT items, total, status FROM orders WHERE user_email=?", (user_email,))
        for o in cursor.fetchall():
            tk.Label(win, text=f"{o[0]} | £{o[1]} | {o[2]}").pack()

    tk.Button(dash, text="View Orders", command=view_orders).pack()

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Producer Dashboard")

    # VIEW ORDERS
    def view_orders():
        win = tk.Toplevel(dash)

        cursor.execute("SELECT id, user_email, items, total, status FROM orders")
        for o in cursor.fetchall():
            frame = tk.Frame(win)
            frame.pack()

            tk.Label(frame, text=f"{o[1]} | {o[2]} | £{o[3]} | {o[4]}").pack(side="left")

            def update_status(order_id):
                cursor.execute("UPDATE orders SET status='Completed' WHERE id=?", (order_id,))
                conn.commit()
                messagebox.showinfo("Updated", "Status changed")

            tk.Button(frame, text="Mark Completed",
                      command=lambda oid=o[0]: update_status(oid)).pack(side="right")

    tk.Button(dash, text="View Orders", command=view_orders).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(root, text="Customer", command=open_login).pack(pady=20)
tk.Button(root, text="Producer", command=open_producer_dashboard).pack()

root.mainloop()
