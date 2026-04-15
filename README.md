import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE SETUP ----------------
# using sqlite because its simple and works offline

conn = sqlite3.connect("glh.db")
cursor = conn.cursor()

# users table
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

# orders table
cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_email TEXT,
    items TEXT,
    total REAL
)
""")

# products table
cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL,
    stock INTEGER
)
""")

# add products first time only
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    # added more products later after testing
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
root.geometry("700x550")
root.configure(bg="#e8f5e9")

# header
top_bar = tk.Frame(root, bg="#2e7d32", height=80)
top_bar.pack(fill="x")

tk.Label(top_bar, text="Greenfield Local Hub",
         font=("Arial", 18, "bold"),
         fg="white", bg="#2e7d32").pack(pady=20)

# main card
main_card = tk.Frame(root, bg="white", bd=2, relief="raised")
main_card.place(relx=0.5, rely=0.5, anchor="center", width=400, height=350)

# logo (had issue before but fixed path)
try:
    logo_img = tk.PhotoImage(file="images/logo.png")
    logo_lbl = tk.Label(main_card, image=logo_img, bg="white")
    logo_lbl.image = logo_img
    logo_lbl.pack(pady=5)
except:
    tk.Label(main_card, text="[Logo missing]", bg="white").pack()

tk.Label(main_card, text="Welcome",
         font=("Arial", 14, "bold"),
         bg="white").pack()

tk.Label(main_card, text="Choose option",
         bg="white").pack(pady=5)

# ---------------- CUSTOMER LOGIN ----------------
def open_login():
    login_win = tk.Toplevel(root)
    login_win.title("Login")
    login_win.geometry("350x300")

    tk.Label(login_win, text="Login").pack(pady=10)

    tk.Label(login_win, text="Email").pack()
    email_entry = tk.Entry(login_win)
    email_entry.pack()

    tk.Label(login_win, text="Password").pack()
    pass_entry = tk.Entry(login_win, show="*")
    pass_entry.pack()

    def check_login():
        # login wasnt working before because of spaces
        email_input = email_entry.get().strip().lower()
        pass_input = pass_entry.get().strip()

        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email_input, pass_input))
        result = cursor.fetchone()

        if result:
            login_win.destroy()
            open_customer_dash(result)
        else:
            messagebox.showerror("Error", "Wrong login details")

    tk.Button(login_win, text="Login",
              bg="#4CAF50", fg="white",
              command=check_login).pack(pady=10)

    tk.Button(login_win, text="Register",
              command=lambda: open_register(login_win)).pack()

# ---------------- REGISTER ----------------
def open_register(prev):
    prev.destroy()

    reg_win = tk.Toplevel(root)
    reg_win.title("Register")
    reg_win.geometry("350x450")

    entries = {}

    labels = ["Username", "Email", "Password", "Address", "County"]

    for lab in labels:
        tk.Label(reg_win, text=lab).pack()
        ent = tk.Entry(reg_win, show="*" if lab=="Password" else None)
        ent.pack()
        entries[lab] = ent

    def save_user():
        data = {k: v.get().strip() for k, v in entries.items()}

        if "" in data.values():
            messagebox.showerror("Error", "Fill everything")
            return

        # didnt check duplicate at first, added later
        cursor.execute("SELECT * FROM users WHERE email=?", (data["Email"],))
        if cursor.fetchone():
            messagebox.showerror("Error", "Email exists")
            return

        cursor.execute("INSERT INTO users VALUES (NULL, ?, ?, ?, ?, ?)",
                       (data["Username"], data["Email"].lower(),
                        data["Password"], data["Address"], data["County"]))
        conn.commit()

        reg_win.destroy()
        open_customer_dash((None, data["Username"], data["Email"]))

    tk.Button(reg_win, text="Register",
              bg="#4CAF50", fg="white",
              command=save_user).pack(pady=15)

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dash(user):
    dash = tk.Toplevel(root)
    dash.title("Shop")
    dash.geometry("900x600")

    user_email = user[2]
    cartItems = []  # cart list

    # header
    bar = tk.Frame(dash, bg="#2e7d32")
    bar.pack(fill="x")

    tk.Label(bar, text="GLH Store", fg="white", bg="#2e7d32").pack(side="left")

    def show_cart():
        c_win = tk.Toplevel(dash)
        total = 0

        for item in cartItems:
            tk.Label(c_win, text=f"{item[0]} - £{item[1]}").pack()
            total += item[1]

        tk.Label(c_win, text=f"Total £{total}").pack()

        def checkout():
            if not cartItems:
                messagebox.showerror("Error", "Cart empty")
                return

            items = ", ".join([i[0] for i in cartItems])

            cursor.execute("INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                           (user_email, items, total))
            conn.commit()

            cartItems.clear()
            messagebox.showinfo("Done", "Order placed")

        tk.Button(c_win, text="Checkout", command=checkout).pack()

    tk.Button(bar, text="Cart", command=show_cart).pack(side="right")

    def show_orders():
        o_win = tk.Toplevel(dash)

        cursor.execute("SELECT items, total FROM orders WHERE user_email=?", (user_email,))
        for o in cursor.fetchall():
            tk.Label(o_win, text=f"{o[0]} | £{o[1]}").pack()

    tk.Button(bar, text="Orders", command=show_orders).pack(side="right")

    # products
    main = tk.Frame(dash)
    main.pack()

    cursor.execute("SELECT name, price FROM products")
    prod = cursor.fetchall()

    def add_cart(n, p):
        cartItems.append((n, p))
        messagebox.showinfo("Cart", f"{n} added")

    r = c = 0
    for p in prod:
        box = tk.Frame(main, bd=1, relief="solid")
        box.grid(row=r, column=c, padx=10, pady=10)

        tk.Label(box, text=p[0]).pack()
        tk.Label(box, text=f"£{p[1]}").pack()

        tk.Button(box, text="Add",
                  command=lambda n=p[0], pr=p[1]: add_cart(n, pr)).pack()

        c += 1
        if c == 3:
            c = 0
            r += 1

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Producer")
    dash.geometry("800x500")

    def view_products():
        win = tk.Toplevel(dash)
        cursor.execute("SELECT name, price, stock FROM products")
        for p in cursor.fetchall():
            tk.Label(win, text=f"{p[0]} | £{p[1]} | {p[2]}").pack()

    def update_stock():
        win = tk.Toplevel(dash)

        tk.Label(win, text="Name").pack()
        name = tk.Entry(win)
        name.pack()

        tk.Label(win, text="Stock").pack()
        stock = tk.Entry(win)
        stock.pack()

        def do_update():
            # simple update, could improve later
            cursor.execute("UPDATE products SET stock=? WHERE name=?",
                           (stock.get(), name.get()))
            conn.commit()
            messagebox.showinfo("Done", "Updated")

        tk.Button(win, text="Update", command=do_update).pack()

    def view_orders():
        win = tk.Toplevel(dash)
        cursor.execute("SELECT user_email, items, total FROM orders")
        for o in cursor.fetchall():
            tk.Label(win, text=f"{o[0]} | {o[1]} | £{o[2]}").pack()

    tk.Button(dash, text="View Products", command=view_products).pack(pady=10)
    tk.Button(dash, text="Update Stock", command=update_stock).pack(pady=10)
    tk.Button(dash, text="View Orders", command=view_orders).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(main_card, text="Customer",
          bg="#4CAF50", fg="white",
          command=open_login).pack(pady=10)

tk.Button(main_card, text="Producer",
          bg="#1976D2", fg="white",
          command=open_producer_dashboard).pack()

root.mainloop()
