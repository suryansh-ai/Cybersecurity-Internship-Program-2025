import tkinter as tk
from tkinter import ttk, filedialog, messagebox
import tldextract
from urllib.parse import urlparse
import whois
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph
from reportlab.lib.styles import getSampleStyleSheet
import requests

def homograph(url):
    # Parse the URL
    parsed_url = urlparse(url)

    # Extract the domain using tldextract
    domain_info = tldextract.extract(parsed_url.netloc)

    # Check if the domain contains non-ASCII characters (potential IDN)
    if any(ord(char) > 127 for char in domain_info.domain):
        return True
    return False

def check_url():
    url = entry.get()
    result_label.config(text="")

    # Handle different URL formats
    if not url.startswith("http://") and not url.startswith("https://"):
        url = "https://" + url

    if homograph(url):
        result_label.config(text=f"The URL '{url}' may be a potential IDN homograph attack.", foreground="red")
    else:
        result_label.config(text=f"The URL '{url}' appears to be genuine.", foreground="green")

# Create the main window
root = tk.Tk()
root.title("Homograph Detection Tool")

# Create and configure a frame
frame = ttk.Frame(root, padding=10)
frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

# Create and configure label and entry widgets for URL check
label = ttk.Label(frame, text="Enter a URL to check:")
label.grid(row=0, column=0, sticky=tk.W)

entry = tk.Entry(frame, width=50, bg="white", fg="black", insertbackground="black")
entry.grid(row=1, column=0, padx=5, pady=5)
entry.bind("<Return>", lambda event: check_url())  # Bind Enter key
entry.focus_set()  # Auto-focus on start

check_button = ttk.Button(frame, text="Check URL", command=check_url)
check_button.grid(row=1, column=1, padx=5, pady=5)

result_label = ttk.Label(frame, text="", font=("Arial", 12))
result_label.grid(row=2, column=0, columnspan=2, pady=10)

# Create and configure result text widget
result_text = tk.Text(frame, height=10, width=50, wrap=tk.WORD)
result_text.grid(row=5, column=0, columnspan=2, padx=5, pady=5)
result_text.tag_configure("green", foreground="green")
result_text.tag_configure("red", foreground="red")
result_text.config(state="disabled")

def toggle_dark_mode():
    global dark_mode
    dark_mode = not dark_mode

    bg = "#2e2e2e" if dark_mode else "#f0f0f0"
    fg = "#ffffff" if dark_mode else "#000000"
    entry_bg = "#3b3b3b" if dark_mode else "#ffffff"
    entry_fg = "#ffffff" if dark_mode else "#000000"

    root.configure(bg=bg)
    frame.configure(style="Dark.TFrame" if dark_mode else "TFrame")

    for widget in frame.winfo_children():
        if isinstance(widget, (ttk.Label, ttk.Button)):
            widget.configure(style="Dark.TLabel" if dark_mode else "TLabel")
        elif isinstance(widget, tk.Entry):
            widget.configure(bg=entry_bg, fg=entry_fg, insertbackground=fg)
        elif isinstance(widget, tk.Text):
            widget.configure(background=entry_bg, foreground=entry_fg, insertbackground=fg)

    result_label.configure(foreground=fg)

# Track dark mode state
dark_mode = False

# Create styles for dark mode
style = ttk.Style()
style.configure("Dark.TFrame", background="#2e2e2e")
style.configure("Dark.TLabel", background="#2e2e2e", foreground="#ffffff")
style.configure("Dark.TButton", background="#2e2e2e", foreground="#ffffff")

# Add the toggle button
dark_mode_button = ttk.Button(frame, text="Toggle Dark Mode 🌙", command=toggle_dark_mode)
dark_mode_button.grid(row=9, column=0, columnspan=2, pady=10)

# Start the GUI main loop
root.mainloop()
