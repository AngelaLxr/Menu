# Menu
import streamlit as st
import speech_recognition as sr
import difflib

# Initialize the recognizer
recognizer = sr.Recognizer()

# Updated menu items and sizes
menu_items = ["Hamburger", "Coca Cola", "French Fries"]
size_options = ["Small", "Medium", "Large"]

# Function to recognize speech from the microphone
def recognize_speech_from_mic(prompt="Speak now"):
    with sr.Microphone() as source:
        st.write(prompt)
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

        try:
            st.write("Recognizing speech...")
            text = recognizer.recognize_google(audio)
            return text
        except sr.UnknownValueError:
            return "Sorry, I could not understand the audio."
        except sr.RequestError as e:
            return f"Could not request results from Google Speech Recognition service; {e}"

# Function to find the closest matches for menu items
def get_closest_matches(voice_input, options):
    words = voice_input.split()
    matched_items = []

    for word in words:
        closest_matches = difflib.get_close_matches(word, options, n=1)
        if closest_matches and closest_matches[0] not in matched_items:
            matched_items.append(closest_matches[0])

    return matched_items

# Function to ask for size using speech
def ask_for_size(item_name):
    st.write(f"Please speak the size for {item_name} (small, medium, large)")
    size_input = recognize_speech_from_mic("Please say Small, Medium, or Large.")
    closest_size = get_closest_matches(size_input, size_options)
    
    if closest_size:
        return closest_size[0]
    else:
        return "Unknown Size"

# Create a button for triggering voice recognition
if st.button("Order by Voice"):
    # Recognize the initial order
    voice_input = recognize_speech_from_mic("Please say your order (e.g., Hamburger, Coca Cola, French Fries).")
    st.write(f"Recognized Speech: {voice_input}")

    # Find the closest matches for the menu items
    ordered_items = get_closest_matches(voice_input, menu_items)

    if ordered_items:
        final_order = {}

        # Loop through the ordered items and ask for sizes if necessary
        for item in ordered_items:
            if item == "Coca Cola" or item == "French Fries":
                size = ask_for_size(item)  # Ask for size via speech
                final_order[item] = size
            else:
                final_order[item] = None  # No size needed for other items

        # Display the final order with sizes
        st.write("Your order is:")
        for item, size in final_order.items():
            if size:
                st.write(f"{item} - Size: {size}")
            else:
                st.write(f"{item}")
    else:
        st.write("Sorry, we couldn't match any items to your order.")
else:
    st.write("You can order 'Hamburger', 'Coca Cola', or 'French Fries' using your voice.")
