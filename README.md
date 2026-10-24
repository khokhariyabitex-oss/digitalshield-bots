import os
import sys
import threading
import time
import zipfile
from http.server import HTTPServer, BaseHTTPRequestHandler

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
ZIP_NAME = os.path.join(BASE_DIR, "DigitalShield_Bots_Render.zip")

# Auto-extract zip file if present in repository
if os.path.exists(ZIP_NAME):
    print(">> Found DigitalShield_Bots_Render.zip, extracting files...")
    try:
        with zipfile.ZipFile(ZIP_NAME, "r") as z:
            z.extractall(BASE_DIR)
        print(">> Extraction completed successfully!")
    except Exception as e:
        print(f">> Extract notice: {e}")

if BASE_DIR not in sys.path:
    sys.path.insert(0, BASE_DIR)

# ----------------- RENDER HEALTH CHECK WEB SERVER -----------------
class HealthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/plain; charset=utf-8")
        self.end_headers()
        status_lines = [
            "OK - Digital Shield Telegram Bots are LIVE on Render!",
            "- Dailytrx Post Bot (@Dailytrxmatebot): Running",
            "- Proof Manager Bot (@Mangerprofbot): Running",
            "Server Time: " + time.strftime("%Y-%m-%d %H:%M:%S UTC", time.gmtime())
        ]
        status_msg = "\n".join(status_lines) + "\n"
        self.wfile.write(status_msg.encode("utf-8"))

    def log_message(self, format, *args):
        pass

def run_health_server():
    port = int(os.environ.get("PORT", 8080))
    server = HTTPServer(("0.0.0.0", port), HealthHandler)
    print(f">> [Render Web Service] Health server listening on 0.0.0.0:{port}")
    server.serve_forever()

def run_trx_bot():
    while True:
        try:
            import friend_trx_bot
            print(">> [Bot 1] Dailytrx Post Bot (@Dailytrxmatebot) is RUNNING!")
            friend_trx_bot.bot.infinity_polling(none_stop=True, timeout=60, long_polling_timeout=30)
        except Exception as e:
            print(f">> [Bot 1] Dailytrx Error: {e}. Reconnecting in 5s...")
            time.sleep(5)

def run_manager_bot():
    while True:
        try:
            import friend_manager_bot
            print(">> [Bot 2] Proof Manager Bot (@Mangerprofbot) is RUNNING!")
            friend_manager_bot.bot.infinity_polling(none_stop=True, timeout=60, long_polling_timeout=30)
        except Exception as e:
            print(f">> [Bot 2] Manager Error: {e}. Reconnecting in 5s...")
            time.sleep(5)

if __name__ == "__main__":
    print("=================================================================")
    print("   DIGITAL SHIELD TELEGRAM BOTS - 24/7 RUNNER (RENDER READY)")
    print("=================================================================")

    # Health check server thread for Render port detection
    web_thread = threading.Thread(target=run_health_server, daemon=True)
    web_thread.start()

    print(">> Starting Bots in parallel threads...")
    t1 = threading.Thread(target=run_trx_bot, daemon=True)
    t2 = threading.Thread(target=run_manager_bot, daemon=True)

    t1.start()
    t2.start()

    print("=================================================================")
    print(">> SUCCESS: BOTH BOTS ARE ACTIVE & LIVE 24/7!")
    print(">> Bot 1: https://t.me/Dailytrxmatebot")
    print(">> Bot 2: https://t.me/Mangerprofbot")
    print("=================================================================")

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\nShutting down bots...")
