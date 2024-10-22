import os
import json
import time
import threading
import subprocess
import sqlite3
import shutil
import base64
import signal
import hashlib
import requests
import psutil
from urllib.parse import urlparse, parse_qs

# Decoded strings
SERVER_LINKS_FILE = 'server_links.txt'
ACCOUNTS_FILE = 'accounts.txt'
CONFIG_FILE = 'config.json'
CACHE_FILE = 'cache.json'

# Global variables
webhook_url = None
device_name = None
interval = None
stop_webhook_thread = False
stop_bypass = False
webhook_thread = None
package_statuses = {}
username_cache = {}
globals()['bypass_time_left'] = {}
globals()['bypass_interval'] = None
globals()['executor_choice'] = None
globals()['roblox_process_count'] = 0
workspace_paths = []
globals()['roblox_packages'] = {}
globals()['user_ids'] = {}
DIR_NAME = subprocess.run(['pwd'], capture_output=True, text=True, check=True).stdout.replace('
', '')
su = f'su -c "export PATH=$PATH:/data/data/com.termux/files/usr/bin && export TERM=xterm-256color && cd {DIR_NAME} && python {__file__}"'
globals()['bypass_time_left'] = {}
globals()['bypass_interval'] = None
globals()['executor_choice'] = None
globals()['roblox_process_count'] = 0

def imp(pkg):
    try:
        __import__(pkg)
        globals()[pkg] = __import__(pkg)
    except ModuleNotFoundError:
        os.system(f'pip install {pkg}')
        globals()[pkg] = __import__(pkg)

# Importing necessary packages
for i in ('requests', 'psutil', 'prettytable', 'asyncio', 'aiohttp'):
    imp(i)

from prettytable import PrettyTable
from urllib.parse import urlparse, parse_qs

# Functions
def c2h6(b):
    return int.from_bytes(base64.b64decode(b), byteorder='big')

def H2SbF7(n):
    return n ^ c2h6(b'enjuly19/\x01')

def tryᅠ(s):
    print('\x1b[1;32m' + s + '\x1b[0m')

def exceptᅠ(s):
    return input('\x1b[1;31m' + s + '\x1b[0m')

def clear_screen():
    if os.name == 'posix':
        os.system('clear')
    else:
        print('\x1b[2J\x1b[H')

def print_header():
    header = r'''
                                           
    '''
    try:
        print('\x1b[1;35m' + header + '\x1b[0m')
    except UnicodeEncodeError:
        print(header)

def create_dynamic_menu(options):
    table = PrettyTable(header=False, border=False, align='l')
    i = 1
    for service in options:
        table.add_row(['\x1b[1;35m[' + str(i) + ']\x1b[1;36m ' + service + '\x1b[0m'])
        i += 1
    print('\x1b[0m' + str(table))

def create_dynamic_table(headers, rows):
    table = PrettyTable(field_names=headers, border=False, align='l')
    for huy in rows:
        table.add_row(list(huy))
    print(table)

def handle_exit_signal(sig, frame):
    global stop_bypass
    try:
        print('\x1b[1;32mExiting program... Saving cache and stopping bypass and webhook threads...\x1b[0m')
    except UnicodeEncodeError:
        print('Exiting program...')
    save_cache()
    stop_event.set()
    stop_webhook()
    stop_bypass = True
    if webhook_thread and webhook_thread.is_alive():
        webhook_thread.join()
    if globals()['bypass_thread'] and globals()['bypass_thread'].is_alive():
        globals()['bypass_thread'].join()
    exit(0)

def update_status_table(package_statuses):
    headers = [
        'Package',
        'Username',
        'Status'
    ]
    cpu_usage = psutil.cpu_percent(interval=1)
    memory_info = psutil.virtual_memory()
    ram = round((memory_info.used / memory_info.total) * 100, 2)
    title = f'CPU: {cpu_usage}% | RAM: {ram}%'
    table = PrettyTable(field_names=headers, title=title, border=False, align='l')
    for package, info in package_statuses.items():
        row = [
            str(package),
            str(info.get('Username', 'N/A')),
            str(info.get('Status', 'N/A'))
        ]
        table.add_row(row)
    clear_screen()
    print_header()
    print(table)

def verify_cookie(cookie_value):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36',
        'Accept-Language': 'en-US,en;q=0.9',
        'Accept': '*/*',
        'Connection': 'keep-alive',
        'Referer': 'https://www.roblox.com/',
        'Origin': 'https://www.roblox.com',
        'Cookie': f'.ROBLOSECURITY={cookie_value}'
    }
    time.sleep(1)
    response = requests.get('https://auth.roblox.com/v2/account/pin', headers=headers)
    if response.status_code == 200:
        try:
            print('\x1b[1;32mCookie is valid!\x1b[0m')
            return True
        except UnicodeEncodeError:
            print('Cookie is valid!')
            return True
    if response.status_code == 403:
        try:
            print('\x1b[1;31mCookie is invalid.\x1b[0m')
            return False
        except UnicodeEncodeError:
            print('Cookie is invalid.')
            return False
    try:
        print(f'\x1b[1;31mError verifying cookie: {response.status_code} - {response.text}\x1b[0m')
    except UnicodeEncodeError:
        print(f'Error verifying cookie: {response.status_code} - {response.text}')
    return False

def download_file(url, destination, binary=True):
    response = requests.get(url, stream=True)
    if response.status_code == 200:
        mode = 'wb' if binary else 'w'
        with open(destination, mode) as file:
            if binary:
                shutil.copyfileobj(response.raw, file)
            else:
                file.write(response.text)
        try:
            print(f'\x1b[1;32m{os.path.basename(destination)} downloaded successfully.\x1b[0m')
        except UnicodeEncodeError:
            print(f'{os.path.basename(destination)} downloaded successfully.')
        return destination
    try:
        print(f'\x1b[1;31mFailed to download {os.path.basename(destination)}.\x1b[0m')
    except UnicodeEncodeError:
        print(f'Failed to download {os.path.basename(destination)}.')
    return None

def replace_cookie_value_in_db(db_path, new_cookie_value):
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM cookies")
    cookie_exists = cursor.fetchone()[0]
    if cookie_exists:
        cursor.execute("UPDATE cookies SET cookie = ?, expires = ?, is_authenticated = ? WHERE cookie = ?", (new_cookie_value, int(time.time() * 1000), 1, new_cookie_value))
    cursor.execute("INSERT OR REPLACE INTO cookies (expires, cookie, creation_time) VALUES (?, ?, ?)", (int(time.time() * 1000), new_cookie_value, int(time.time() * 1000)))
    conn.commit()
    conn.close()
    try:
        print('\x1b[1;32mCookie value successfully replaced in database.\x1b[0m')
    except UnicodeEncodeError:
        print('Cookie value successfully replaced in database.')

def inject_cookies_and_appstorage():
    db_url = ''
    appstorage_url = ''
    downloaded_db_path = download_file(db_url, 'Cookies.db', binary=True)
    downloaded_appstorage_path = download_file(appstorage_url, 'appStorage.json', binary=True)
    if not downloaded_db_path or not downloaded_appstorage_path:
        try:
            print('\x1b[1;31mFailed to download Cookies.db or appStorage.json. Skipping injection.\x1b[0m')
        except UnicodeEncodeError:
            print('Failed to download Cookies.db or appStorage.json. Skipping injection.')
        return None
    cookie_txt_path = os.path.join(os.getcwd(), 'cookies.txt')
    if not os.path.exists(cookie_txt_path):
        try:
            print('\x1b[1;31mcookies.txt not found. Skipping injection.\x1b[0m')
        except UnicodeEncodeError:
            print('cookies.txt not found. Skipping injection.')
        return None
    with open(cookie_txt_path, 'r') as file:
        cookies = file.readlines()
    packages = get_roblox_packages()
    if len(cookies) > len(packages):
        try:
            print('\x1b[1;31mMore cookies than Roblox packages detected. Skipping injection of extra cookies.\x1b[0m')
        except UnicodeEncodeError:
            print('More cookies than Roblox packages detected. Skipping injection of extra cookies.')
        return None
    for idx, package_name in enumerate(packages):
        if idx < len(cookies):
            cookie = cookies[idx].strip()
            try:
                print(f'\x1b[1;33mVerifying cookie for {package_name} before injection...\x1b[0m')
            except UnicodeEncodeError:
                print(f'Verifying cookie for {package_name} before injection...')
            if verify_cookie(cookie):
                try:
                    print(f'\x1b[1;32mCookie for {package_name} is valid!\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Cookie for {package_name} is valid!')
                try:
                    print(f'\x1b[1;32mInjecting cookie for {package_name}: {cookie}\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Injecting cookie for {package_name}: {cookie}')
                destination_db_dir = f'/data/data/{package_name}/app_webview/Default/'
                destination_appstorage_dir = f'/data/data/{package_name}/files/appData/LocalStorage/'
                os.makedirs(destination_db_dir, exist_ok=True)
                os.makedirs(destination_appstorage_dir, exist_ok=True)
                destination_db_path = os.path.join(destination_db_dir, 'Cookies.db')
                shutil.copyfile(downloaded_db_path, destination_db_path)
                try:
                    print(f'\x1b[1;32mCopied Cookies.db to {destination_db_path}\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Copied Cookies.db to {destination_db_path}')
                destination_appstorage_path = os.path.join(destination_appstorage_dir, 'appStorage.json')
                shutil.copyfile(downloaded_appstorage_path, destination_appstorage_path)
                try:
                    print(f'\x1b[1;32mCopied appStorage.json to {destination_appstorage_path}\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Copied appStorage.json to {destination_appstorage_path}')
                replace_cookie_value_in_db(destination_db_path, cookie)
                try:
                    print(f'\x1b[1;33mVerifying cookie for {package_name} after injection...\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Verifying cookie for {package_name} after injection...')
                if verify_cookie(cookie):
                    try:
                        print(f'\x1b[1;32mCookie for {package_name} is valid after injection!\x1b[0m')
                    except UnicodeEncodeError:
                        print(f'Cookie for {package_name} is valid after injection!')
                else:
                    try:
                        print(f'\x1b[1;31mCookie for {package_name} is invalid after injection!\x1b[0m')
                    except UnicodeEncodeError:
                        print(f'Cookie for {package_name} is invalid after injection!')
    try:
        print('\x1b[1;32mCookies and appStorage injected successfully.\x1b[0m')
    except UnicodeEncodeError:
        print('Cookies and appStorage injected successfully.')

def get_roblox_packages():
    packages = []
    result = subprocess.run('pm list packages -f', shell=True, capture_output=True, text=True)
    if result.returncode == 0:
        for line in result.stdout.strip().splitlines():
            name = line.split('package:')[-1].strip()
            try:
                print(f'Found package: {name}')
            except UnicodeEncodeError:
                print(f'Found package: {name}')
            packages.append(name)
        return packages

def capture_screenshot():
    current_dir = os.path.dirname(os.path.abspath(__file__))
    screenshots_dir = os.path.join(current_dir, 'screenshots')
    os.makedirs(screenshots_dir, exist_ok=True)
    screenshot_path = os.path.join(screenshots_dir, 'screenshot.png')
    try:
        print('\x1b[1;32mCapturing screenshot...\x1b[0m')
    except UnicodeEncodeError:
        print('Capturing screenshot...')
    result = os.system(f'/system/bin/screencap -p {screenshot_path}')
    if result != 0 or not os.path.exists(screenshot_path):
        try:
            print(f'\x1b[1;31mFailed to capture screenshot at {screenshot_path}. Skipping this webhook.\x1b[0m')
        except UnicodeEncodeError:
            print(f'Failed to capture screenshot at {screenshot_path}. Skipping this webhook.')
        return None
    return screenshot_path

def get_system_info():
    cpu_usage = psutil.cpu_percent(interval=1)
    memory_info = psutil.virtual_memory()
    uptime = time.time() - psutil.boot_time()
    system_info = {
        'Uptime': uptime,
        'Memory Used': memory_info.used,
        'Memory Available': memory_info.available,
        'Total Memory': memory_info.total,
        'CPU Usage': cpu_usage
    }
    return system_info

def count_roblox_processes():
    count = 0
    for proc in psutil.process_iter(['name']):
        if 'roblox' in proc.info['name'].lower():
            count += 1
    return count

def send_webhook():
    if not stop_webhook_thread:
        screenshot_path = capture_screenshot()
        if not os.path.exists(screenshot_path):
            try:
                print(f'\x1b[1;31mScreenshot file not found: {screenshot_path}. Skipping this webhook.\x1b[0m')
            except UnicodeEncodeError:
                print(f'Screenshot file not found: {screenshot_path}. Skipping this webhook.')
            return None
        system_info = get_system_info()
        roblox_process_count = count_roblox_processes()
        embed = {
            "title": {"text": "Roblox Status"},
            "fields": [
                {"name": "Device Name", "value": device_name, "inline": True},
                {"name": "CPU Usage", "value": f"{system_info['CPU Usage']}%", "inline": True},
                {"name": "Memory Used", "value": f"{system_info['Memory Used'] / 1024**3:.2f} GB", "inline": True},
                {"name": "Total Memory", "value": f"{system_info['Total Memory'] / 1024**3:.2f} GB", "inline": True},
                {"name": "Uptime", "value": f"{system_info['Uptime'] / 3600:.2f} hours", "inline": True},
                {"name": "Roblox Processes", "value": f"{roblox_process_count}", "inline": True}
            ],
            "footer": {"text": "System Info for " + device_name},
            "color": 16711680
        }
        payload = {
            "content": device_name,
            "embeds": [
                embed
            ]
        }
        file = open(screenshot_path, 'rb')
        response = requests.post(webhook_url, data={'payload_json': json.dumps(payload)}, files={'file': (os.path.basename(screenshot_path), file)})
        try:
            print(f'\x1b[1;32mWebhook sent successfully.\x1b[0m')
        except UnicodeEncodeError:
            print('Webhook sent successfully.')
        file.close()
        if response.status_code == 204 or response.status_code == 200:
            try:
                print('\x1b[1;32mWebhook sent successfully.\x1b[0m')
            except UnicodeEncodeError:
                print('Webhook sent successfully.')
        else:
            try:
                print(f'\x1b[1;31mFailed to send webhook, status code: {response.status_code}\x1b[0m')
            except UnicodeEncodeError:
                print(f'Failed to send webhook, status code: {response.status_code}')
        os.remove(screenshot_path)
        time.sleep(interval * 60)
        if not stop_webhook_thread:
            send_webhook()

def stop_webhook():
    global stop_webhook_thread
    stop_webhook_thread = True
    stop_event.set()

def setup_webhook():
    global stop_webhook_thread, webhook_url, device_name, interval
    stop_webhook_thread = False
    webhook_url = except('Enter your webhook URL:')
    device_name = except('Enter your device name:')
    interval = int(except('Enter the webhook interval in minutes:'))
    save_config()
    stop_webhook_thread = False
    threading.Thread(target=send_webhook).start()

def setup_check_executor():
    if globals()['executor_choice'] == 'None':
        ask = str(except('Do you want to use an executor? (y/n):')).lower()
        if ask == 'y':
            globals()['executor_choice'] = 'True'
            save_config()
        elif ask == 'n':
            globals()['executor_choice'] = 'False'
            save_config()
    save_config()

def is_roblox_running(package_name):
    for proc in psutil.process_iter(['name']):
        if package_name in proc.info['name'].lower():
            return True
    return False

def kill_roblox_processes():
    try:
        print('\x1b[1;32mKilling all Roblox processes...\x1b[0m')
    except UnicodeEncodeError:
        print('Killing all Roblox processes...')
    package_names = get_roblox_packages()
    for package_name in package_names:
        try:
            print(f'Trying to kill process for package: {package_name}')
        except UnicodeEncodeError:
            print(f'Trying to kill process for package: {package_name}')
        os.system(f'nohup /system/bin/am force-stop {package_name} > /dev/null 2>&1 &')
        time.sleep(2)

def kill_roblox_process(package_name):
    os.system(f'nohup /system/bin/am force-stop {package_name} > /dev/null 2>&1 &')
    time.sleep(5)

def check_executor_and_rejoin(package_name, user_id, package_statuses, server_link, num_packages):
    detected_executors = detect_and_write_lua_script()
    if len(detected_executors) > 0:
        if globals()['roblox_packages'][package_name]:
            globals()['roblox_packages'][package_name] = False
            reset_executor_file(user_id)
            start_time = time.time()
            executor_loaded = False
            while time.time() - start_time < 120:
                if check_executor_status(user_id, package_name):
                    executor_loaded = True
                    break
            if not executor_loaded:
                try:
                    print(f"Executor didn't load. Rejoining...")
                except UnicodeEncodeError:
                    print(f"Executor didn't load. Rejoining...")
                package_statuses[package_name]['Status'] = '\x1b[1;32mJoined without executor for ' + str(user_id) + '\x1b[0m'
                update_status_table(package_statuses)
                kill_roblox_process(package_name)
                time.sleep(2)
                a = threading.Thread(target=launch_roblox, args=[package_name, server_link, num_packages, package_statuses])
                a.daemon = True
                a.start()
    else:
        try:
            print(f'No executor detected for {package_name} (ID: {user_id}).')
        except UnicodeEncodeError:
            print(f'No executor detected for {package_name} (ID: {user_id}).')
        package_statuses[package_name]['Status'] = '\x1b[1;32mJoined without executor for ' + str(user_id) + '\x1b[0m'
        update_status_table(package_statuses)

def detect_and_write_lua_script():
    detected_executors = []
    for executor_name, path in globals()['executors'].items():
        if not os.path.exists(path):
            continue
        detected_executors.append(executor_name)
        lua_script_path = os.path.join(path, 'executor.lua')
        if os.path.exists(lua_script_path):
            try:
                print(f'Lua script already exists for {executor_name}')
            except UnicodeEncodeError:
                print(f'Lua script already exists for {executor_name}')
        else:
            with open(lua_script_path, 'w') as file:
                code = requests.get('').text
                file.write(code)
            try:
                print(f'Lua script written to {lua_script_path} for {executor_name}')
            except UnicodeEncodeError:
                print(f'Lua script written to {lua_script_path} for {executor_name}')
    return detected_executors

def reset_executor_file(id):
    for workspace in globals()['workspaces']:
        file_path = os.path.join(workspace, f'{id}.main')
        if os.path.exists(file_path):
            os.remove(file_path)

def check_executor_status(id, package_name, continuous=True, max_wait_time=120, retry_timeout=120):
    status_file = f'{id}.main'
    retry_timeout = time.time() + max_wait_time
    start_time = time.time()
    if continuous and time.time() > retry_timeout:
        try:
            print(f'Executor did not load within {max_wait_time} seconds for {id}. Triggering rejoin...')
        except UnicodeEncodeError:
            print(f'Executor did not load within {max_wait_time} seconds for {id}. Triggering rejoin...')
        return False
    for workspace in globals()['workspaces']:
        file_path = os.path.join(workspace, status_file)
        if os.path.exists(file_path):
            return True
    return False

def check_permission():
    res = subprocess.run(['id', '-u'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    return res.returncode == 0

def launch_roblox(package_name, server_link, num_packages, package_statuses):
    globals()['roblox_packages'][package_name] = True
    package_statuses[package_name]['Status'] = '\x1b[1;36mOpening Roblox for ' + package_name + '...\x1b[0m'
    update_status_table(package_statuses)
    subprocess.run(['am', 'start', '-a', 'com.roblox.client.startup.ActivitySplash', '-n', f'{package_name}/com.roblox.client.startup.ActivitySplash'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    if num_packages >= 2:
        time.sleep(8)
    if not is_roblox_running(package_name):
        kill_roblox_process(package_name)
        subprocess.run(['am', 'start', '-a', 'com.roblox.client.startup.ActivitySplash', '-n', f'{package_name}/com.roblox.client.startup.ActivitySplash'], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        if num_packages >= 2:
            time.sleep(8)
        if not is_roblox_running(package_name):
            return None
    package_statuses[package_name]['Status'] = '\x1b[1;36mJoining Roblox for ' + package_name + '...\x1b[0m'
    update_status_table(package_statuses)
    subprocess.run(['am', 'start', '-a', 'com.roblox.client.ActivityProtocolLaunch', '-n', f'{package_name}/com.roblox.client.ActivityProtocolLaunch', '-d', server_link], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    time.sleep(20)
    package_statuses[package_name]['Status'] = '\x1b[1;36mJoined Roblox for ' + package_name + '\x1b[0m'
    update_status_table(package_statuses)
    globals()['roblox_packages'][package_name] = False

def format_server_link(input_link):
    if 'roblox://placeID=' in input_link:
        return input_link
    if input_link.isdigit():
        return f'roblox://placeID={input_link}'
    return 'roblox://placeID=2753915549'

def save_server_links(server_links):
    with open(SERVER_LINKS_FILE, 'w') as file:
        for package, link in server_links:
            file.write(f'{package},{link}
')

def load_server_links():
    server_links = []
    if os.path.exists(SERVER_LINKS_FILE):
        with open(SERVER_LINKS_FILE, 'r') as file:
            for line in file:
                (package, link) = line.strip().split(',', 1)
                server_links.append((package, link))
        return server_links
    return server_links

def save_accounts(accounts):
    with open(ACCOUNTS_FILE, 'w') as file:
        for package, user_id in accounts:
            file.write(f'{package},{user_id}
')

def load_accounts():
    accounts = []
    if os.path.exists(ACCOUNTS_FILE):
        with open(ACCOUNTS_FILE, 'r') as file:
            for line in file:
                (package, user_id) = line.strip().split(',', 1)
                globals()['user_ids'][package] = user_id
                accounts.append((package, user_id))
        return accounts
    return accounts

def find_userid_from_file(file_path):
    with open(file_path, 'r') as file:
        content = json.load(file)
        id = content.get('UserId', None)
        if id == 'None':
            return None
        return id

async def get_user_id(username):
    payload = {"name": True, "usernames": [username]}
    headers = {'Content-Type': 'application/json'}
    async with aiohttp.ClientSession() as session:
        async with session.post('https://api.roblox.com/users/get-by-username', json=payload, headers=headers) as response:
            if response.status_code == 200:
                data = await response.json()
                if 'Id' in data and len(data['Id']) > 0:
                    return data['Id']
            else:
                try:
                    print(f'\x1b[1;31mError retrieving user ID for username: {username}\x1b[0m')
                except UnicodeEncodeError:
                    print(f'Error retrieving user ID for username: {username}')
    return None

def get_server_link(package_name, server_links):
    return next((link for pkg, link in server_links if pkg == package_name), None)

def get_username_from_id(user_id):
    if not get_username(user_id):
        return user_id

def get_username(user_id):
    user = load_saved_username(user_id)
    if user:
        return user
    retry_attempts = 3 

    for attempt in range(retry_attempts):
        try:
            response = requests.get(f'https://users.roblox.com/v1/users/{user_id}', timeout=3)  
            response.raise_for_status()
            data = response.json()
            username = data.get('displayName', None)  
            if username:
                username_cache[user_id] = username
                save_username(user_id, username)
                return username
        except requests.exceptions.RequestException as e:
            print(f'Attempt {attempt+1} failed for Roblox API: {e}')
            time.sleep(2 ** attempt) 
            
    for attempt in range(retry_attempts):
        try:
            response = requests.get(f'https://users.roproxy.com/v1/users/{user_id}', timeout=3)  
            response.raise_for_status()
            data = response.json()
            username = data.get('displayName', None)  # Assuming 'displayName' is the key
            if username:
                username_cache[user_id] = username
                save_username(user_id, username)
                return username
        except requests.exceptions.RequestException as e:
            print(f'Attempt {attempt+1} failed for RoProxy API: {e}')
            time.sleep(2 ** attempt)  # Exponential backoff
