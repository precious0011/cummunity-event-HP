import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3

# ---------------- 1. DATABASE SETUP ----------------
conn = sqlite3.connect("glh_final.db")
cursor = conn.cursor()

cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT, email TEXT, password TEXT, address TEXT, county TEXT)")
cursor.execute("CREATE TABLE IF NOT EXISTS orders (id INTEGER PRIMARY KEY AUTOINCREMENT, user_email TEXT, items TEXT, total REAL)")
# Added img_file column for Version 4
cursor.execute("CREATE TABLE IF NOT EXISTS products (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, price REAL, stock INTEGER, img_file TEXT)")

# Insert default products if table is empty
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    items = [
        ("Milk", 1.50, 10, "milk.png"), ("Bread", 1.20, 15, "bread.png"),
        ("Eggs", 2.50, 20, "eggs.png"), ("Vegetables", 3.00, 12, "veg.png"),
        ("Apple", 1.00, 25, "apple.png"), ("Cheese", 2.80, 8, "cheese.png")
    ]
    cursor.executemany("INSERT INTO products (name, price, stock, img_file) VALUES (?, ?, ?, ?)", items)
    conn.commit()

# ---------------- 2. HELPER FUNCTIONS ----------------
# Update this path to your actual images folder
IMAGE_PATH = r"C:\Users\TLevel-Digital-OS-11\Desktop\Version control\images\\"

def get_img(filename):
    try:
        return tk.PhotoImage(file=IMAGE_PATH + filename)
    except:
        return None

# ---------------- 3. DASHBOARD FUNCTIONS ----------------

def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title(f"GLH Store - {user[1]}")
    dash.geometry("900x700")
    
    user_email = user[2]
    cart = []

    # Header & Search
    nav = tk.Frame(dash, bg="#2e7d32", pady=10)
    nav.pack(fill="x")
    
    tk.Label(nav, text="SEARCH:", fg="white", bg="#2e7d32").pack(side="left", padx=10)
    search_var = tk.StringVar()
    search_entry = tk.Entry(nav, textvariable=search_var)
    search_entry.pack(side="left", padx=5)

    # Product Container
    content_frame = tk.Frame(dash)
    content_frame.pack(fill="both", expand=True, padx=20, pady=20)

    def refresh_view():
        for widget in content_frame.winfo_children():
            widget.destroy()
        
        term = f"%{search_var.get()}%"
        cursor.execute("SELECT name, price, img_file FROM products WHERE name LIKE ?", (term,))
        
        r, c = 0, 0
        for p_name, p_price, p_img in cursor.fetchall():
            box = tk.Frame(content_frame, bd=1, relief="solid", padx=10, pady=10)
            box.grid(row=r, column=c, padx=10, pady=10)

            img = get_img(p_img)
            if img:
                lbl = tk.Label(box, image=img)
                lbl.image = img
                lbl.pack()
            else:
                tk.Label(box, text="[No Image]", bg="#ccc", width=10, height=4).pack()

            tk.Label(box, text=p_name, font=("Arial", 10, "bold")).pack()
            tk.Label(box, text=f"£{p_price:.2f}", fg="green").pack()
            tk.Button(box, text="Add", command=lambda n=p_name, p=p_price: cart.append((n,p))).pack()

            c += 1
            if c == 3: c=0; r+=1

    tk.Button(nav, text="Go", command=refresh_view).pack(side="left")
    tk.Button(nav, text="View Cart", command=lambda: messagebox.showinfo("Cart", f"Items: {len(cart)}")).pack(side="right", padx=10)
    
    refresh_view()

def open_producer_dashboard():
    p_win = tk.Toplevel(root)
    p_win.title("Producer Console")
    p_win.geometry("800x500")

    sidebar = tk.Frame(p_win, bg="#1a237e", width=150)
    sidebar.pack(side="left", fill="y")
    
    main_area = tk.Frame(p_win, bg="white")
    main_area.pack(side="right", fill="both", expand=True)

    def show_stock():
        for w in main_area.winfo_children(): w.destroy()
        tree = ttk.Treeview(main_area, columns=("Name", "Stock"), show="headings")
        tree.heading("Name", text="Product")
        tree.heading("Stock", text="In Stock")
        tree.pack(fill="both", pady=20)
        
        cursor.execute("SELECT name, stock FROM products")
        for row in cursor.fetchall(): tree.insert("", "end", values=row)

    tk.Button(sidebar, text="Inventory", command=show_stock, bg="#283593", fg="white").pack(fill="x", pady=5)
    show_stock()

# ---------------- 4. MAIN UI LAYOUT ----------------
root = tk.Tk()
root.title("Greenfield Local Hub")
root.geometry("700x550")
root.configure(bg="#e8f5e9")

# Header
header = tk.Frame(root, bg="#2e7d32", height=80)
header.pack(fill="x")
tk.Label(header, text="TOKA FITNESS / GLH", font=("Arial", 18, "bold"), fg="white", bg="#2e7d32").pack(pady=20)

# Create the 'card' FIRST
card = tk.Frame(root, bg="white", bd=2, relief="raised")
card.place(relx=0.5, rely=0.5, anchor="center", width=420, height=350)

# Add elements to the card
tk.Label(card, text="Welcome", font=("Arial", 14, "bold"), bg="white").pack(pady=10)

# Login logic
def open_login():
    win = tk.Toplevel(root)
    win.geometry("300x200")
    tk.Label(win, text="Email").pack()
    e = tk.Entry(win); e.pack()
    
    def do_login():
        cursor.execute("SELECT * FROM users WHERE email=?", (e.get(),))
        user = cursor.fetchone()
        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "User not found")

    tk.Button(win, text="Login", command=do_login).pack(pady=10)

# Finally, the Buttons (defined AFTER 'card' exists)
tk.Button(card, text="Customer Portal", bg="#4CAF50", fg="white", width=25, command=open_login).pack(pady=10)
tk.Button(card, text="Producer Portal", bg="#1976D2", fg="white", width=25, command=open_producer_dashboard).pack()

root.mainloop()
