# python-mini-projects
jaarvis 
import pyttsx3
import speech_recognition as sr
import datetime
import wikipedia
import webbrowser
import os

# ---------- TTS (pyttsx3) ----------
engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)  # 0=male, 1=female (varies by system)

def speak(text: str):
    engine.say(text)
    engine.runAndWait()

# ---------- Wish ----------
def wishMe():
    hour = int(datetime.datetime.now().hour)
    if 0 <= hour < 12:
        speak("Good morning! I am Jarvis. Please tell me, how may I help you?")
    elif 12 <= hour < 18:
        speak("Good afternoon! I am Jarvis. Please tell me, how may I help you?")
    else:
        speak("Good evening! I am Jarvis. Please tell me, how may I help you?")

# ---------- Speech to text ----------
def takeCommand(device_index=None):
    r = sr.Recognizer()
    r.pause_threshold = 0.8           # shorter pauses between words are allowed
    r.energy_threshold = 250          # baseline; will be recalibrated
    try:
        with sr.Microphone(device_index=device_index) as source:
            print("Listening...")
            # Calibrate to room noise for better VAD
            r.adjust_for_ambient_noise(source, duration=0.8)
            audio = r.listen(source, timeout=6, phrase_time_limit=8)

        print("Recognizing...")
        # IMPORTANT: it's 'language', not 'languages'
        query = r.recognize_google(audio, language='en-IN')
        print(f"User said: {query}\n")
        return query

    except sr.WaitTimeoutError:
        print("No speech detected (timeout).")
        return ""
    except sr.UnknownValueError:
        print("Sorry, I couldn't understand the audio.")
        return ""
    except sr.RequestError as e:
        # Typically internet/DNS issues for Google API
        print(f"Recognition request failed: {e}")
        return ""
    except Exception as e:
        print(f"Unexpected error: {e}")
        return ""

# ---------- Main ----------
if __name__ == "__main__":
    wishMe()
    while True:
        query = takeCommand().strip().lower()
        if not query:
            continue

        if 'wikipedia' in query:
            speak('Searching Wikipedia...')
            topic = query.replace('wikipedia', '').strip()
            try:
                results = wikipedia.summary(topic, sentences=2)
                speak("According to Wikipedia")
                print(results)
                speak(results)
            except Exception as e:
                print(f"Wikipedia error: {e}")
                speak("Sorry, I couldn't fetch that from Wikipedia.")

        elif 'open youtube' in query:
            webbrowser.open("https://youtube.com")

        elif 'open google' in query:
            webbrowser.open("https://google.com")

        elif 'open stackoverflow' in query:
            webbrowser.open("https://stackoverflow.com")

        elif 'play music' in query:
            try:
                music_dir = r'D:\songs\Favorite'  # change if needed
                songs = os.listdir(music_dir)
                if songs:
                    print(f"Playing: {songs[0]}")
                    os.startfile(os.path.join(music_dir, songs[0]))
                else:
                    speak("Your music folder is empty.")
            except Exception as e:
                print(f"Music error: {e}")
                speak("I couldn't play music. Please check your folder path.")

        elif 'the time' in query:
            # %S is seconds (uppercase). %s is NOT valid for strftime.
            strTime = datetime.datetime.now().strftime("%H:%M:%S")
            speak(f"Sir, the time is {strTime}")

        elif 'exit' in query or 'quit' in query or 'stop' in query:
            speak("Goodbye!")
            break
