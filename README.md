# TextToSpeechSpeechToText_pp
This is a python project which is gui based. It converts text to speech and also speech to text using different python libraries. 

from gtts import gTTS, lang
import os
import platform
import tkinter as tk
from tkinter import messagebox, ttk
import speech_recognition as sr

# --- Load Supported Languages ---
languages = lang.tts_langs() 
language_names = list(languages.values())

# --- Functions ---
def text_to_speech():
    text = text_entry.get("1.0", "end-1c")
    selected_name = selected_accent.get()

    if len(text) <= 1:
        messagebox.showerror(title="⚠️ Missing Text", message="Please enter some text first!")
        return

    try:
        language_code = [k for k, v in languages.items() if v == selected_name][0]
        speech = gTTS(text=text, lang=language_code, slow=False)
        speech.save("text.mp3")

        if platform.system() == "Windows":
            os.startfile("text.mp3")
        else:
            os.system("mpg123 text.mp3")

        messagebox.showinfo(title="✅ Success", message="Speech has been generated successfully!")
    except Exception as e:
        messagebox.showerror(title="❌ Error", message=f"Error: {e}")

def list_accents():
    messagebox.showinfo(title="🌎 Supported Accents",
                        message="\n".join([f"{k}: {v}" for k, v in languages.items()]))

def speech_to_text():
    recorder = sr.Recognizer()
    try:
        duration = int(duration_entry.get())
    except:
        messagebox.showerror(title="⚠️ Missing Duration", message="Enter a valid duration in seconds")
        return

    messagebox.showinfo(message=f"🎤 Recording for {duration} seconds.\nSpeak clearly after clicking OK.")

    with sr.Microphone() as mic:
        try:
            recorder.adjust_for_ambient_noise(mic, duration=1)
            audio_input = recorder.listen(mic, phrase_time_limit=duration)

            selected_name = selected_accent.get()
            language_code = [k for k, v in languages.items() if v == selected_name][0]
            lang_for_recognition = language_code + "-IN" if language_code in ["mr", "hi", "en"] else language_code

            text_output = recorder.recognize_google(audio_input, language=lang_for_recognition)
            messagebox.showinfo(title="📝 Recognized Speech", message=f"You said:\n\n{text_output}")

        except sr.UnknownValueError:
            messagebox.showerror(title="😕 Oops", message="Could not understand your speech. Try again.")
        except sr.RequestError as e:
            messagebox.showerror(title="🌐 Network Error", message=f"Google API error: {e}")
        except Exception as e:
            messagebox.showerror(title="⚠️ Error", message=f"Unexpected error: {e}")

# --- GUI Setup ---
window = tk.Tk()
window.geometry("700x500")
window.title("🎤 Speech ↔ Text Converter")
window.resizable(False, False)

# Gradient Background
canvas = tk.Canvas(window, width=700, height=500, highlightthickness=0)
canvas.pack(fill="both", expand=True)

for i in range(0, 500):
    r = 240 - int(i * 0.1)
    g = 248 - int(i * 0.2)
    b = 255 - int(i * 0.1)
    color = f"#{r:02x}{g:02x}{b:02x}"
    canvas.create_line(0, i, 700, i, fill=color)

# Frame with New Soft Color
frame = tk.Frame(window, bg="#FFF5F5", bd=3, relief="ridge")  # soft peach/pink background
frame.place(x=50, y=70, width=600, height=300)

# Header
header = tk.Label(window, text="✨ Speech ↔ Text Converter ✨",
                  font=("Helvetica", 20, "bold"), bg="#E6F7FF", fg="#1A202C")
header.place(relx=0.5, y=30, anchor="center")

# Text input
tk.Label(frame, text="📝 Text:", bg="#FFF5F5", font=("Arial", 12, "bold")).place(x=10, y=10)
text_entry = tk.Text(frame, width=50, height=5, wrap="word", font=("Arial", 11))
text_entry.place(x=120, y=10)  # moved right for better spacing

# Accent dropdown
tk.Label(frame, text="🌐 Accent:", bg="#FFF5F5", font=("Arial", 12, "bold")).place(x=10, y=120)
selected_accent = tk.StringVar()
selected_accent.set("Marathi")
accent_dropdown = ttk.Combobox(frame, textvariable=selected_accent, values=language_names, width=35)
accent_dropdown.place(x=130, y=120)  # moved right for spacing

# Duration entry
tk.Label(frame, text="⏱ Duration (sec):", bg="#FFF5F5", font=("Arial", 12, "bold")).place(x=10, y=160)
duration_entry = tk.Entry(frame, width=37)
duration_entry.insert(0, "5")
duration_entry.place(x=170, y=160)  # moved right for spacing

# Button Styling
def style_button(btn, color):
    btn.configure(font=("Arial", 12, "bold"), bg=color, fg="white", relief="flat", cursor="hand2", activebackground="#2C7A7B")

# Buttons
btn1 = tk.Button(window, text="🌎 List Accents", command=list_accents)
btn1.place(x=80, y=420, width=150, height=35)
style_button(btn1, "#63B3ED")

btn2 = tk.Button(window, text="🔊 Text ➝ Speech", command=text_to_speech)
btn2.place(x=280, y=420, width=150, height=35)
style_button(btn2, "#48BB78")

btn3 = tk.Button(window, text="🎤 Speech ➝ Text", command=speech_to_text)
btn3.place(x=480, y=420, width=150, height=35)
style_button(btn3, "#F6AD55")

window.mainloop()