import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_final.db")
cursor = conn.cursor()

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
root.geometry("800x600")
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

# logo
try:
    logo = tk.PhotoImage(file="images/logo.png")
    tk.Label(card, image=logo, bg="white").pack(pady=5)
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
    for l in ["Username", "Email", "Password", "Address", "County"]:
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
    dash.geometry("1000x700")

    user_email = user[2]
    cart = []

    # load images
    imgs = {}
    for name in ["Milk","Bread","Eggs","Vegetables","Apple","Cheese","Juice","Chicken"]:
        try:
            imgs[name] = tk.PhotoImage(file=f"images/{name.lower()}.png").subsample(2,2)
        except:
            imgs[name] = None

    # header
    header = tk.Frame(dash, bg="#2e7d32")
    header.pack(fill="x")

    tk.Label(header, text="GLH Store", fg="white", bg="#2e7d32",
             font=("Arial", 16)).pack(side="left", padx=10)

    # search
    search_entry = tk.Entry(header)
    search_entry.pack(side="left", padx=10)

    main = tk.Frame(dash)
    main.pack(pady=20)

    def add_to_cart(n, p):
        cart.append((n, p))
        messagebox.showinfo("Cart", f"{n} added")

    def display_products(products):
        for widget in main.winfo_children():
            widget.destroy()

        r = c = 0
        for p in products:
            box = tk.Frame(main, bd=1, relief="solid", bg="white", padx=10, pady=10)
            box.grid(row=r, column=c, padx=15, pady=15)

            if imgs[p[0]]:
                lbl = tk.Label(box, image=imgs[p[0]], bg="white")
                lbl.image = imgs[p[0]]
                lbl.pack()
            else:
                tk.Label(box, text=p[0], bg="white").pack()

            tk.Label(box, text=p[0], font=("Arial", 10, "bold"), bg="white").pack()
            tk.Label(box, text=f"£{p[1]}", fg="green", bg="white").pack()

            tk.Button(box, text="Add",
                      command=lambda n=p[0], pr=p[1]: add_to_cart(n, pr)).pack(pady=5)

            c += 1
            if c == 3:
                c = 0
                r += 1

    def search_products():
        keyword = search_entry.get()
        cursor.execute("SELECT name, price FROM products WHERE name LIKE ?",
                       ('%' + keyword + '%',))
        display_products(cursor.fetchall())

    tk.Button(header, text="Search", command=search_products).pack(side="left")

    def view_cart():
        win = tk.Toplevel(dash)
        total = sum([i[1] for i in cart])

        for i in cart:
            tk.Label(win, text=f"{i[0]} - £{i[1]}").pack()

        tk.Label(win, text=f"Total £{total}").pack()

    tk.Button(header, text="Cart", command=view_cart).pack(side="right")

    cursor.execute("SELECT name, price FROM products")
    display_products(cursor.fetchall())

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Producer Dashboard")
    dash.geometry("900x600")
    dash.configure(bg="#f5f5f5")

    tk.Label(dash, text="Producer Dashboard",
             font=("Arial", 18, "bold"),
             bg="#f5f5f5").pack(pady=10)

    frame = tk.Frame(dash, bg="#f5f5f5")
    frame.pack(pady=20)

    def view_products():
        for w in frame.winfo_children():
            w.destroy()

        cursor.execute("SELECT name, price, stock FROM products")
        for p in cursor.fetchall():
            box = tk.Frame(frame, bg="white", bd=2, relief="groove", padx=10, pady=10)
            box.pack(pady=8, fill="x", padx=20)

            tk.Label(box, text=p[0], font=("Arial", 12, "bold"),
                     bg="white").pack(side="left")

            tk.Label(box, text=f"£{p[1]}", fg="green",
                     bg="white").pack(side="left", padx=20)

            color = "red" if p[2] < 5 else "green"
            tk.Label(box, text=f"Stock: {p[2]}",
                     fg=color, bg="white").pack(side="right")

    def update_stock():
        win = tk.Toplevel(dash)

        tk.Label(win, text="Product").pack()
        name = tk.Entry(win)
        name.pack()

        tk.Label(win, text="Stock").pack()
        stock = tk.Entry(win)
        stock.pack()

        def update():
            cursor.execute("UPDATE products SET stock=? WHERE name=?",
                           (stock.get(), name.get()))
            conn.commit()
            messagebox.showinfo("Done", "Updated")

        tk.Button(win, text="Update", command=update).pack()

    tk.Button(dash, text="View Products", bg="#4CAF50", fg="white",
              width=20, command=view_products).pack(pady=10)

    tk.Button(dash, text="Update Stock", bg="#1976D2", fg="white",
              width=20, command=update_stock).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(card, text="Customer", bg="#4CAF50", fg="white",
          width=20, command=open_login).pack(pady=10)

tk.Button(card, text="Producer", bg="#1976D2", fg="white",
          width=20, command=open_producer_dashboard).pack()

root.mainloop()
