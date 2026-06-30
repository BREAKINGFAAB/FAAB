import os
import sys
import json
import gzip
import shutil
import time
import subprocess
import urllib.request
import urllib.parse
import base64

SUB_URL = "https://raw.githubusercontent.com/MahanKenway/Freedom-V2Ray/main/configs/mix.txt"
BATCH_SIZE = 100
BINARY_URL = "https://github.com/xxf098/LiteSpeedTest/releases/download/v0.15.0/lite-linux-amd64-v0.15.0.gz"
ARCHIVE_NAME = "lite-speed-test.gz"
BINARY_NAME = "lite-speed-test"
CONFIG_NAME = "config.json"

def download_and_prepare_binary():
    if not os.path.exists(BINARY_NAME):
        try:
            req = urllib.request.Request(BINARY_URL, headers={'User-Agent': 'Mozilla/5.0'})
            with urllib.request.urlopen(req) as response, open(ARCHIVE_NAME, 'wb') as out_file:
                shutil.copyfileobj(response, out_file)
            with gzip.open(ARCHIVE_NAME, 'rb') as f_in, open(BINARY_NAME, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)
            os.chmod(BINARY_NAME, 0o755)
            os.remove(ARCHIVE_NAME)
        except Exception as e:
            sys.exit(1)

def create_test_config():
    config_data = {
        "group": "Github_Worker", "sub": "", "testMode": "all", "speedtestMode": "all",
        "thread": 100, "pingMethod": "tcping", "speedDuration": 3, "speedSize": 10, "concurrency": 1
    }
    with open(CONFIG_NAME, "w", encoding="utf-8") as f:
        json.dump(config_data, f)

def fetch_and_decode_sub(url):
    try:
        req = urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})
        with urllib.request.urlopen(req) as response:
            raw_text = response.read().decode('utf-8').strip()
        valid_protocols = ('vmess://', 'vless://', 'trojan://', 'ss://', 'ssr://')
        if not any(raw_text.startswith(p) for p in valid_protocols):
            padded = raw_text + "=" * ((4 - len(raw_text) % 4) % 4)
            raw_text = base64.b64decode(padded).decode('utf-8', errors='ignore')
        return raw_text
    except:
        return ""

def get_remark_from_link(link):
    try:
        if link.startswith("vmess://"):
            b64_str = link[8:]
            b64_str += "=" * ((4 - len(b64_str) % 4) % 4)
            data = json.loads(base64.b64decode(b64_str).decode('utf-8', errors='ignore'))
            return data.get("ps", "").strip()
        elif "#" in link:
            return urllib.parse.unquote(link.split("#")[-1]).strip()
    except: pass
    return None

def run_batch(batch_configs):
    with open("batch.txt", "w", encoding="utf-8") as f:
        f.write("\n".join(batch_configs))
    try:
        subprocess.run([f"./{BINARY_NAME}", "--config", CONFIG_NAME, "--test", "batch.txt"],
                       stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, timeout=180)
    except: pass
    
    result_files = [f for f in os.listdir('.') if f.startswith('result_') and f.endswith('.json')]
    batch_results = []
    for rf in result_files:
        try:
            with open(rf, "r", encoding="utf-8") as f:
                nodes = json.load(f).get("nodes", [])
                batch_results.extend([n for n in nodes if n.get("speed", 0) > 0])
        except: pass
        try: os.remove(rf)
        except: pass
    if os.path.exists("batch.txt"): os.remove("batch.txt")
    return batch_results

if __name__ == "__main__":
    download_and_prepare_binary()
    create_test_config()
    raw_text = fetch_and_decode_sub(SUB_URL)
    valid_protocols = ('vmess://', 'vless://', 'trojan://', 'ss://', 'ssr://')
    configs = [line.strip() for line in raw_text.splitlines() if line.strip().startswith(valid_protocols)]
    
    if not configs:
        sys.exit(0)
        
    config_map = {get_remark_from_link(c): c for c in configs if get_remark_from_link(c)}
    total_batches = (len(configs) + BATCH_SIZE - 1) // BATCH_SIZE
    master_results = []
    
    for b_idx in range(1, total_batches + 1):
        batch_configs = configs[(b_idx-1)*BATCH_SIZE : b_idx*BATCH_SIZE]
        master_results.extend(run_batch(batch_configs))
        
    top_20 = sorted(master_results, key=lambda x: x.get("speed", 0), reverse=True)[:20]
    
    # ذخیره کانفیگ‌های گلچین شده در یک فایل متنی
    with open("best_configs.txt", "w", encoding="utf-8") as f:
        for node in top_20:
            raw_link = config_map.get(node.get("remarks", ""), None)
            if raw_link:
                f.write(raw_link + "\n")
