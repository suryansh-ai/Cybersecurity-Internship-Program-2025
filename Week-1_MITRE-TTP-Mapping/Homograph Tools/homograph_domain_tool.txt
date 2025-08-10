import tkinter as tk
from tkinter import messagebox
import random, string, threading, urllib.parse, webbrowser, pyperclip
from flask import Flask, redirect

# Flask app
app = Flask(__name__)
links = {}

@app.route("/<token>")
def go(token):
    return redirect(links.get(token, "/"), 302)

def run_flask():
    app.run(port=5000, debug=False, use_reloader=False)

# Homograph map (preview only)
hm = {
    'a': ['а', 'ɑ'], 'c': ['с'], 'e': ['е'], 'i': ['і', 'ɩ'],
    'o': ['о', 'ο'], 'p': ['р'], 's': ['ѕ'], 'y': ['у'],
    'l': ['ⅼ', 'ӏ'], 'n': ['ո']
}

def homograph_preview(domain):
    return "".join(
        random.choice(hm[ch.lower()]) if ch.lower() in hm and random.choice([True, False]) else ch
        for ch in domain
    )

def short_token(n=6):
    return ''.join(random.choice(string.ascii_letters+string.digits) for _ in range(n))

# Create short link
def create_short():
    url = entry.get().strip()
    if not url:
        messagebox.showerror("Error", "Enter a URL"); return
    if "://" not in url:
        url = "http://" + url

    chosen_domain = custom_domain.get().strip() or domain_var.get()
    fake_domain = homograph_preview(chosen_domain)

    token = short_token()
    links[token] = url

    short_link = f"http://{fake_domain}/{token}"
    preview_var.set(short_link)
    local_short.set(f"http://127.0.0.1:5000/{token}")

def copy_link():
    if preview_var.get():
        pyperclip.copy(preview_var.get())
        messagebox.showinfo("Copied", "Preview link copied to clipboard")

def open_link(e):
    if local_short.get():
        webbrowser.open(local_short.get())

# Flask in background
threading.Thread(target=run_flask, daemon=True).start()

# Tkinter UI
root = tk.Tk()
root.title("Homograph Domain Tool")
root.geometry("500x350")

tk.Label(root, text="Enter Long URL:").pack()
entry = tk.Entry(root, width=50); entry.pack(pady=5)

tk.Label(root, text="Choose Domain or Type Custom:").pack()
domain_var = tk.StringVar(value="google.com")
tk.OptionMenu(root, domain_var, "google.com", "office.com", "facebook.com", "yahoo.com").pack(pady=2)
custom_domain = tk.Entry(root, width=30)
custom_domain.insert(0, "")
custom_domain.pack(pady=2)

tk.Button(root, text="Generate Short Link", command=create_short).pack(pady=5)

tk.Label(root, text="Homograph Preview (click to open):").pack()
preview_var = tk.StringVar()
preview_lbl = tk.Label(root, textvariable=preview_var, fg="blue", cursor="hand2")
preview_lbl.pack()
preview_lbl.bind("<Button-1>", open_link)

tk.Button(root, text="Copy Preview Link", command=copy_link).pack(pady=5)

local_short = tk.StringVar()

root.mainloop()
