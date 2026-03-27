import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh.db")
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
conn.commit()

# ---------------- MAIN WINDOW ----------------
root = tk.Tk()
root.title("Greenfield Local Hub")
root.geometry("600x500")
root.configure(bg="#e8f5e9")

# ---------------- HEADER ----------------
header = tk.Frame(root, bg="#2e7d32", height=80)
header.pack(fill="x")

tk.Label(header, text="Greenfield Local Hub",
         font=("Arial", 18, "bold"),
         fg="white", bg="#2e7d32").pack(pady=20)

# ---------------- CARD ----------------
card = tk.Frame(root, bg="white", bd=2, relief="raised")
card.place(relx=0.5, rely=0.5, anchor="center", width=350, height=320)

tk.Label(card, text="Welcome", font=("Arial", 14, "bold"), bg="white").pack(pady=10)
tk.Label(card, text="Select your role", bg="white").pack(pady=5)

# ---------------- FUNCTIONS ----------------

def open_customer():
    open_login()

def open_producer():
    messagebox.showinfo("Info", "Producer dashboard coming soon")

# -------- LOGIN --------
def open_login():
    win = tk.Toplevel(root)
    win.title("Login")
    win.geometry("300x250")

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get(), password.get()))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_dashboard()
        else:
            messagebox.showerror("Error", "Invalid login")

    tk.Button(win, text="Login", command=login).pack(pady=10)

    tk.Button(win, text="Don't have an account?",
              command=lambda: open_register(win)).pack()

# -------- REGISTER --------
def open_register(parent):
    parent.destroy()

    win = tk.Toplevel(root)
    win.title("Register")
    win.geometry("300x400")

    tk.Label(win, text="Username").pack()
    username = tk.Entry(win)
    username.pack()

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    tk.Label(win, text="Address").pack()
    address = tk.Entry(win)
    address.pack()

    tk.Label(win, text="County").pack()
    county = tk.Entry(win)
    county.pack()

    def register():
        cursor.execute("""
        INSERT INTO users (username, email, password, address, county)
        VALUES (?, ?, ?, ?, ?)
        """, (username.get(), email.get(), password.get(),
              address.get(), county.get()))
        conn.commit()

        win.destroy()
        open_dashboard()

    tk.Button(win, text="Register", command=register).pack(pady=10)

# -------- DASHBOARD --------
def open_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Shop")
    dash.geometry("600x500")
    dash.configure(bg="white")

    # SEARCH BAR
    search = tk.Entry(dash, width=40)
    search.pack(pady=10)

    # PRODUCT FRAME
    frame = tk.Frame(dash, bg="white")
    frame.pack()

    # LOAD IMAGES (NO PIL)
    milk = tk.PhotoImage(file="images/milk.png")
    bread = tk.PhotoImage(file="images/bread.png")
    eggs = tk.PhotoImage(file="images/eggs.png")
    veg = tk.PhotoImage(file="images/veg.png")

    products = [
        ("Milk", "£1.50", milk),
        ("Bread", "£1.20", bread),
        ("Eggs", "£2.50", eggs),
        ("Veg", "£3.00", veg),
    ]

    row = 0
    col = 0

    for name, price, img in products:
        box = tk.Frame(frame, bd=1, relief="solid")
        box.grid(row=row, column=col, padx=10, pady=10)

        lbl = tk.Label(box, image=img)
        lbl.image = img
        lbl.pack()

        tk.Label(box, text=name).pack()
        tk.Label(box, text=price, fg="green").pack()

        col += 1
        if col == 2:
            col = 0
            row += 1

# ---------------- BUTTONS ----------------
tk.Button(card, text="Customer", command=open_customer).pack(pady=10)
tk.Button(card, text="Producer", command=open_producer).pack()

root.mainloop()
