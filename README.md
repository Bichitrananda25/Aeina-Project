import pyttsx3
import random
import os
import webbrowser
import tkinter as tk
from tkinter import Label, Entry, Button, Text, Scrollbar, Frame
from PIL import Image, ImageTk, ImageSequence
import speech_recognition as sr
import threading
import time
import pygame  # For background music

# Initialize Text-to-Speech Engine
engine = pyttsx3.init()

voices = engine.getProperty("voices")
if len(voices) > 1:
    engine.setProperty("voice", voices[1].id)  # Female voice
engine.setProperty("rate", 150)

# Initialize Speech Recognition
recognizer = sr.Recognizer()

# Initialize Background Music (Optional)
def play_music():
    pygame.mixer.init()
    music_path = "background_music.mp3"  # Change this to an actual file path
    if os.path.exists(music_path):
        pygame.mixer.music.load(music_path)
        pygame.mixer.music.play(-1)

# Predefined Outfit Recommendations with URLs
outfits = {
    "male": {
        "party": [("🕴 Black suit with a stylish shirt", "https://www.amazon.com/s?k=mens+party+wear")],
        "office": [("👔 Formal shirt with trousers", "https://www.amazon.com/s?k=formal+men+shirts")],
        "interview": [("📑 Full suit with polished shoes", "https://www.amazon.com/s?k=men+interview+outfit")],
        "dating": [("💖 Casual shirt with jeans", "https://www.amazon.com/s?k=men+casual+dating+outfit")]
    },
    "female": {
        "party": [("👗 Trendy short dress with heels", "https://www.amazon.com/s?k=women+party+dress")],
        "office": [("💼 Pencil skirt with a blouse", "https://www.amazon.com/s?k=women+office+outfit")],
        "interview": [("📖 Formal dress with minimal accessories", "https://www.amazon.com/s?k=women+interview+outfit")],
        "dating": [("❤ Red dress with heels", "https://www.amazon.com/s?k=women+dating+outfit")]
    }
}

# Global variables to track state
current_step = "start"
user_gender = ""

# Function to Speak Text
def speak(text):
    print(f"Lyra: {text}")  # Debugging
    engine.say(text)
    engine.runAndWait()

# GUI Chatbot Interface
def fashion_assistant():
    global current_step, user_gender

    # GUI Setup
    root = tk.Tk()
    root.title("Lyra - Your AI Assistant")
    root.geometry("500x600")
    root.config(bg="#1E1E1E")

    # Load and Animate GIF Avatar
    avatar_path = r"C:\Users\Asus\OneDrive\Desktop\ShirtTryOn\lyra_avatar.gif"  # Update with your actual GIF path
    avatar_label = Label(root, bg="#1E1E1E")
    avatar_label.pack()

    if os.path.exists(avatar_path):
        lyra_gif = Image.open(avatar_path)
        frames = [ImageTk.PhotoImage(frame) for frame in ImageSequence.Iterator(lyra_gif)]

        def animate_gif(frame=0):
            avatar_label.config(image=frames[frame])
            root.after(100, animate_gif, (frame + 1) % len(frames))  # Loop animation

        animate_gif()
    else:
        print("Warning: Avatar image not found.")
        avatar_label.config(text="🖼️ Avatar Missing", font=("Arial", 12))

    # Chatbox Frame
    chat_frame = Frame(root, bg="#2C2F33")
    chat_frame.pack(padx=10, pady=10, expand=True, fill="both")

    # Chatbox (Display Messages)
    chatbox = Text(chat_frame, wrap="word", state="disabled", font=("Arial", 12), bg="#36393F", fg="white")
    chatbox.pack(padx=10, pady=10, expand=True, fill="both")

    # Scrollbar
    scrollbar = Scrollbar(chatbox)
    scrollbar.pack(side="right", fill="y")
    chatbox.config(yscrollcommand=scrollbar.set)

    # Input Box
    entry = Entry(root, font=("Arial", 14), bg="#40444B", fg="white", insertbackground="white")
    entry.pack(padx=10, pady=10, fill="x")

    # Function to Update Chatbox
    def update_chatbox(speaker, message, tag="bot"):
        chatbox.config(state="normal")
        chatbox.insert(tk.END, f"{speaker}: {message}\n", tag)
        chatbox.config(state="disabled")
        chatbox.update_idletasks()

    # Welcome Message Function
    def welcome_message():
        speak("Welcome to AutoTrend! Hi, I am Lyra. Are you a Male or Female ?")
        update_chatbox("Lyra", "Welcome to AutoTrend! Hi, I am Lyra. Are you male or female?", "bot")
        global current_step
        current_step = "awaiting_gender"

    # Function to Handle User Input
    def process_input():
        global current_step, user_gender

        user_message = entry.get().strip().lower()
        entry.delete(0, tk.END)  # Clear input field

        if not user_message:
            return  # Ignore empty input

        update_chatbox("You", user_message, "user")

        # Step 1: Asking for Gender
        if current_step == "awaiting_gender":
            if user_message in ["male", "female"]:
                user_gender = user_message  # Store gender
                speak("For which occasion? Party, Office, Interview, or Dating?")
                update_chatbox("Lyra", "Choose an occasion: Party, Office, Interview, Dating", "bot")
                current_step = "awaiting_occasion"
            else:
                speak("Please enter Male or Female.")
                update_chatbox("Lyra", "Invalid input. Please enter Male or Female.", "bot")
            return
        
        # Step 2: Waiting for Occasion Input
        if current_step == "awaiting_occasion":
            if user_message in outfits[user_gender]:
                suggestion, url = random.choice(outfits[user_gender][user_message])
                speak(f"For {user_message}, I suggest: {suggestion}")
                update_chatbox("Lyra", suggestion, "bot")
                webbrowser.open(url)  # Open link
                update_chatbox("Lyra", f"Click here for more options: {url}", "bot")
                current_step = "awaiting_gender"  # Reset chatbot
            else:
                speak("Sorry, I can only suggest outfits for Party, Office, Interview, or Dating.") 
                update_chatbox("Lyra", "Invalid occasion. Please choose from Party, Office, Interview, or Dating.", "bot")
            return

    # Buttons
    send_button = Button(root, text="Send", font=("Arial", 14), command=process_input)
    send_button.pack()
    voice_button = Button(root, text="🎤 Speak", font=("Arial", 14), command=process_input)
    voice_button.pack()

    # Run welcome message after GUI loads
    root.after(1000, welcome_message)  # Wait 1 sec then show welcome message

    # Run GUI
    root.mainloop()

# Run the chatbot
if __name__ == "__main__":
    threading.Thread(target=play_music).start()
    fashion_assistant()
# Aeina-Project
Student project based on Fashion Designing using AI
