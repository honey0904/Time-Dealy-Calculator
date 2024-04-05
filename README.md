import tkinter as tk
import pygame
import cv2
import time
from moviepy.editor import VideoFileClip, AudioFileClip
from PIL import Image, ImageTk
from threading import Thread
from tkinter import ttk
import pyttsx3
import tkinter as tk
import pygame
from PIL import Image, ImageTk
import cv2
import threading
import time
from tkinter import PhotoImage

def on_closing():
    pygame.mixer.music.stop()  # Stop the audio
    video_paused.set()  # Pause the video thread
    root.destroy()

root = tk.Tk()
root.title('Video and Sound Player')

def set_bg(window,image_path):
    bg_image=PhotoImage(file=image_path)
    bg_label=tk.label(window,image=bg_image)
    bg_label.place(relwidth=1,relheight=1)
    root=tk.Tk()
    root.geometry("800x600")
    
    bg_path=r"C:\Users\parip\Downloads\monday-word-made-from-metallic-letterpress-type-on-wooden-tray-FY5KFM.png"
    set_bg(root,bg_path)
    root.mainloop()
video_path = r"C:\Users\parip\Downloads\Untitled video - Made with Clipchamp (3) (1).mp4"
sound_path = r"C:\Users\parip\Downloads\Untitled-video-Made-with-Clipchamp-3-1.mp3"

# Initialize pygame once
pygame.init()

# Function to play video and sound
video_paused = threading.Event()

def play_video():
    cap = cv2.VideoCapture(video_path)
    while True:
        if not video_paused.is_set():
            ret, frame = cap.read()
            if ret:
                frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                img = Image.fromarray(frame)
                img_tk = ImageTk.PhotoImage(image=img)
                video_label.config(image=img_tk)
                video_label.image = img_tk
                root.update()
                time.sleep(0)  # Set the delay to slow down the video to about 10 FPS
            else:
                cap.release()
                break

def play_sound():
    pygame.mixer.music.load(sound_path)
    pygame.mixer.music.play()

def play_video_and_sound():
    video_thread = threading.Thread(target=play_video)
    sound_thread = threading.Thread(target=play_sound)

    video_thread.start()
    sound_thread.start()

def pause_video():
    video_paused.set()
    pygame.mixer.music.pause()

def resume_video():
    video_paused.clear()
    pygame.mixer.music.unpause()

def close_video():
    pygame.mixer.music.stop()  # Stop the audio
    video_paused.set()  # Pause the video thread
    root.destroy()

# Add the video label spanning the top of the window
video_label = tk.Label(root)
video_label.grid(row=0, column=0, columnspan=4, sticky="nsew")  # Fill the top part of the window

# Add the buttons frame at the bottom of the window
button_frame = tk.Frame(root)
button_frame.grid(row=1, column=0, columnspan=4, pady=10)

# Unicode symbols for buttons
play_symbol = u"\u25B6"  # Play symbol (►)
pause_symbol = u"\u23F8"  # Pause symbol (⏸)
resume_symbol = u"\u25B6\u25B6"  # Resume symbol (►►)
close_symbol = u"\u2716"  # Close symbol (✖)

play_button = tk.Button(button_frame, text=play_symbol, font=("Helvetica", 20), command=play_video_and_sound)
play_button.grid(row=0, column=0, padx=10)

pause_button = tk.Button(button_frame, text=pause_symbol, font=("Helvetica", 20), command=pause_video)
pause_button.grid(row=0, column=1, padx=10)

resume_button = tk.Button(button_frame, text=resume_symbol, font=("Helvetica", 20), command=resume_video)
resume_button.grid(row=0, column=2, padx=10)

close_button = tk.Button(button_frame, text=close_symbol, font=("Helvetica", 20), command=close_video)
close_button.grid(row=0, column=3, padx=10)

# Configure rows and columns to expand with the window size
root.columnconfigure(0, weight=1)
root.rowconfigure(0, weight=1)

# Bind the close button event to the on_closing function
root.protocol("WM_DELETE_WINDOW", on_closing)

root.mainloop()

class OleShopGUI:
    def init(self):
        self.root = tk.Tk()
        self.root.title("Welcome to Ole's Shop")
        self.root.geometry("800x500")  # Set the dimensions here
        self.root.config(bg="black")  # Set the background color of the main window

        # Add the welcome label at the top
        welcome_label = ttk.Label(self.root, text="Welcome to Ole's Shop", font=("Helvetica", 20, "bold"), foreground="blue")
        welcome_label.pack(pady=20)

        self.days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
        self.result_label = ttk.Label(self.root, text="", font=("Helvetica", 14, "bold"), foreground="blue")
        self.result_label.pack(pady=10)
        self.jobs_data = {}  # Store job information for each day
        self.create_day_buttons()

        # Initialize video player components
        self.video_label = tk.Label(self.root)
        self.video_label.pack(pady=10)  # Video label placed above the button

        self.play_button = tk.Button(self.root, text="Play Video", command=self.play_video_with_audio)
        self.play_button.pack(pady=10)  # Button placed below the video label

        self.root.mainloop()

    def time_to_minutes(self, time_str):
        hours, minutes = map(int, time_str.split(':'))
        if 8 <= hours < 13:
            return (hours - 8) * 60 + minutes
        else:
            return (hours + 4) * 60 + minutes - 30

    def calculate_average_delay_day(self, jobs):
        total_delay = 0
        previous_finish_time = 0

        for arrival_time_str, processing_time in jobs:
            arrival_time = self.time_to_minutes(arrival_time_str)
            start_time = max(arrival_time, previous_finish_time)
            finish_time = start_time + processing_time
            delay = max(0, start_time - arrival_time)
            total_delay += delay
            previous_finish_time = finish_time

        average_delay = total_delay / len(jobs) if len(jobs) > 0 else 0
        return average_delay

    def speak_text(self, text):
        engine = pyttsx3.init()
        engine.say(text)
        engine.runAndWait()

    def create_day_buttons(self):
        for day in self.days:
            button = ttk.Button(self.root, text=day, command=lambda day=day: self.on_day_selected(day))
            button.pack(pady=5, padx=20)

            # Apply a custom style for the buttons
            s = ttk.Style()
            s.configure(f"{day}.TButton", foreground="black", background="green", font=("Helvetica", 12, "bold"), padding=5)
            button.config(style=f"{day}.TButton")

    def on_day_selected(self, day):
        self.speak_text(f"You selected {day}. Please enter the number of jobs on {day}.")
        self.get_jobs_for_day(day)

    def on_jobs_entered(self, day, num_jobs):
        self.speak_text(f"You entered {num_jobs} jobs for {day}. Please enter the job details.")
        self.get_job_details(day, num_jobs)

    def on_calculate(self, day, jobs_data):
        average_delay = self.calculate_average_delay_day(jobs_data)
        self.result_label.config(text=f"Average delay for {day}: {average_delay:.2f} minutes")
        self.speak_text(f"Average delay for {day} is {average_delay:.2f} minutes.")

    def get_jobs_for_day(self, day):
        window = tk.Toplevel()
        window.title(day)
        window.config(bg="lightgray") # Set the background color of the day's window

        tk.Label(window, text=f"Enter the number of jobs on {day}:", font=("Helvetica", 12, "bold"), fg="blue").pack(pady=10)

        entry_num_jobs = ttk.Entry(window, font=("Helvetica", 12), foreground="black", background="white")
        entry_num_jobs.pack(pady=5)

        def save_num_jobs():
            num_jobs = int(entry_num_jobs.get())
            window.destroy()
            self.on_jobs_entered(day, num_jobs)

        tk.Button(window, text="Next", command=save_num_jobs, font=("Helvetica", 12, "bold"), fg="white", bg="blue").pack(pady=10)

    def get_job_details(self, day, num_jobs):
        window = tk.Toplevel()
        window.title(day)
        window.config(bg="lightgray") # Set the background color of the day's window

        entry_times = []
        entry_processing_times = []

        for i in range(num_jobs):
            tk.Label(window, text=f"Job {i + 1}", font=("Helvetica", 12, "bold"), fg="blue").pack(pady=5)
            entry_time = ttk.Entry(window, font=("Helvetica", 12), foreground="black", background="white")
            entry_time.pack(pady=5)
            entry_times.append(entry_time)

            tk.Label(window, text="Processing Time", font=("Helvetica", 12, "bold"), fg="blue").pack(pady=5)
            entry_processing_time = ttk.Entry(window, font=("Helvetica", 12), foreground="black", background="white")
            entry_processing_time.pack(pady=5)
            entry_processing_times.append(entry_processing_time)

        tk.Button(window, text="Calculate", command=lambda: self.on_calculate(day, self.get_jobs_data(entry_times, entry_processing_times)), font=("Helvetica", 12, "bold"), fg="white", bg="green").pack(pady=10)

    def get_jobs_data(self, entry_times, entry_processing_times):
        jobs_data = []
        for i in range(len(entry_times)):
            time_str = entry_times[i].get()
            processing_time = int(entry_processing_times[i].get())
            jobs_data.append((time_str, processing_time))
        jobs_data.sort(key=lambda job: (self.time_to_minutes(job[0]), job[1]))
        return jobs_data

    def play_video_with_audio(self):
        def video_thread():
            cap = cv2.VideoCapture(video_path)
            while cap.isOpened():
                ret, frame = cap.read()
                if ret:
                    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                    frame = cv2.resize(frame, (400, 400))  # Resize the frame to 400x300
                    img = Image.fromarray(frame)
                    img_tk = ImageTk.PhotoImage(image=img)
                    self.video_label.config(image=img_tk)
                    self.video_label.image = img_tk
                    self.root.update()
                    time.sleep(1.0 / video_clip.fps)  # Adjust frame rate to match video fps
                else:
                    cap.release()
                    pygame.mixer.music.stop()
                    cv2.destroyAllWindows()
                    break

        video_path = r"C:\Users\parip\Downloads\Untitled design.mp4"
        audio_path = r"C:\Users\parip\Downloads\Untitled design.mp3"

        # Load video and audio
        video_clip = VideoFileClip(video_path)
        audio_clip = AudioFileClip(audio_path)

        # Synchronize audio duration with video duration
        audio_duration = min(video_clip.duration, audio_clip.duration)
        audio_clip = audio_clip.subclip(0, audio_duration)

        # Set the audio of the video clip
        video_clip = video_clip.set_audio(audio_clip)

        # Create a video preview
        global preview_clip
        preview_clip = video_clip.subclip(0, min(20, video_clip.duration))  # Preview first 5 seconds

        pygame.mixer.init()
        pygame.mixer.music.load(audio_path)
        pygame.mixer.music.play()

        video_thread_obj = Thread(target=video_thread)
        video_thread_obj.start()

if _name_ == "_main_":
    app = OleShopGUI()
