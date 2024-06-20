# Codsoft

import tkinter as tk
from tkinter import messagebox
import datetime
import time

class AlarmClockApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Alarm Clock App")
        self.root.geometry("300x250")

        # Create frames
        self.header_frame = tk.Frame(self.root, bg="#f0f0f0")
        self.header_frame.pack(fill="x")

        self.content_frame = tk.Frame(self.root, bg="#f0f0f0")
        self.content_frame.pack(fill="both", expand=True)

        # Create header widgets
        self.time_label = tk.Label(self.header_frame, text="", font=("Helvetica", 24), bg="#f0f0f0")
        self.time_label.pack(side="left", padx=10)

        self.alarm_button = tk.Button(self.header_frame, text="Set New Alarm", command=self.set_alarm)
        self.alarm_button.pack(side="right", padx=10)

        # Create content widgets
        self.alarm_listbox = tk.Listbox(self.content_frame, width=30, height=10)
        self.alarm_listbox.pack(fill="both", expand=True)

        self.snooze_button = tk.Button(self.content_frame, text="Snooze", command=self.snooze_alarm)
        self.snooze_button.pack(fill="x")

        self.dismiss_button = tk.Button(self.content_frame, text="Dismiss", command=self.dismiss_alarm)
        self.dismiss_button.pack(fill="x")

        # Initialize alarm clock
        self.alarm_clock = AlarmClock()
        self.update_time()

    def update_time(self):
        current_time = datetime.datetime.now().strftime("%H:%M:%S")
        self.time_label.config(text=current_time)
        self.root.after(1000, self.update_time)

    def set_alarm(self):
        alarm_time = self.get_alarm_time()
        if alarm_time:
            self.alarm_clock.set_alarm(alarm_time)
            self.alarm_listbox.insert(tk.END, alarm_time.strftime("%H:%M:%S"))

    def get_alarm_time(self):
        alarm_time_window = tk.Toplevel(self.root)
        alarm_time_window.title("Set Alarm Time")

        hour_label = tk.Label(alarm_time_window, text="Hour:")
        hour_label.pack()
        hour_entry = tk.Entry(alarm_time_window, width=5)
        hour_entry.pack()

        minute_label = tk.Label(alarm_time_window, text="Minute:")
        minute_label.pack()
        minute_entry = tk.Entry(alarm_time_window, width=5)
        minute_entry.pack()

        def set_alarm_time():
            hour = int(hour_entry.get())
            minute = int(minute_entry.get())
            alarm_time = datetime.datetime.now().replace(hour=hour, minute=minute, second=0)
            alarm_time_window.destroy()
            return alarm_time

        set_button = tk.Button(alarm_time_window, text="Set", command=set_alarm_time)
        set_button.pack()

        alarm_time_window.wait_window(alarm_time_window)

    def snooze_alarm(self):
        selected_alarm = self.alarm_listbox.curselection()
        if selected_alarm:
            alarm_time = self.alarm_clock.snooze_alarm(selected_alarm[0])
            self.alarm_listbox.delete(selected_alarm[0])
            self.alarm_listbox.insert(tk.END, alarm_time.strftime("%H:%M:%S"))

    def dismiss_alarm(self):
        selected_alarm = self.alarm_listbox.curselection()
        if selected_alarm:
            self.alarm_clock.dismiss_alarm(selected_alarm[0])
            self.alarm_listbox.delete(selected_alarm[0])

class AlarmClock:
    def __init__(self):
        self.alarms = []

    def set_alarm(self, alarm_time):
        self.alarms.append(alarm_time)

    def snooze_alarm(self, index):
        alarm_time = self.alarms[index]
        snooze_time = datetime.timedelta(minutes=5)
        alarm_time += snooze_time
        return alarm_time

    def dismiss_alarm(self, index):
        del self.alarms[index]

if __name__ == "__main__":
    root = tk.Tk()
    app = AlarmClockApp(root)
    root.mainloop()