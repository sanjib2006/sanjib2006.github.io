---
title: CTF Writeups - Chronos CTF (IIT Mandi)
description: Solutions for selected challenges from Chronos CTF, Xpecto 2025, IIT Mandi.
date: 2025-08-16 00:00:00 +0530
categories: [CTF, Writeups]
tags: [ctf, forensics, rev, osint] 
pin: true
image:
    path: /assets/img/chronos-xpecto25/ctf-banner.png
    alt: Chronos CTF
---

# Chronos CTF

---

## Challenge: *Newlines*

**Category:** Forensics  

### Description
> *Description: Some secrets hide in plain sight, while others lurk between the lines. Can you find what’s hidden in this seemingly ordinary text file?*


**Given Files:**  
- `newlines.zip`

### Solution

Extracting `newlines.zip` revealed **70 text files** (`0.txt` to `69.txt`), each containing varying newlines. The newline count in each file corresponded to ASCII values. The following script extracted the hidden message:

```python
import os

def count_lines_to_ascii():
    line_counts = []
    for i in range(70):
        filename = f"{i}.txt"
        if os.path.exists(filename):
            with open(filename, "r", encoding="utf-8") as f:
                line_count = max(0, sum(1 for _ in f) - 1)
                line_counts.append(line_count)
        else:
            line_counts.append(0)
    
    ascii_string = ''.join(chr(count) for count in line_counts)
    print(ascii_string)

if __name__ == "__main__":
    count_lines_to_ascii()
```


### Flag

```
saic{s0_m4ny_n3wl1n35_b7w_7h15_15_4_v3ry_l0ng_fl4g_y0u_wr073_5cr1p7!!}
```

---

## Challenge: *Xpecto Decryptum*

**Category:** Reversing Engineering  


### Description

> *Someone thought it’d be funny to wrap a secret inside layers of Python magic. With just the right touch you might just unwrap this digital present and find the treasure inside. But be careful—messing with the wrong parts might leave you scratching your head instead! 🔐🐍*


**Given Code**  
```python
import base64
from cryptography.fernet import Fernet

payload = b'gAAAAABn2vH52liBfuFfazSkLcAVO5N3AaoIJslqczD_GbOi7Ygbq2eKZWF2YYqFSUv34O-eQeH_2DvL-cJZceUmk1Tlp_fOEhXn30uG4tjmgZm0Dfc2LhSDJtFrdYSriHuKOIfpjrs0V6xyo8_Vg2HsUePmXDXFF_mzicZqDX5wLw0NHBFn3fqjsdbavOnqZw0YIhPbEbyaupbzOULllF9VUGvn53D3RFlyHD8M3jY8xAPkj0YY6D2WmmKtDu16XKK4asdLEsYo-7uSGFG-UO6fxdf1ncWXD8pM0EAOYj1Y6tlSH3t9qXE='
key_str = 'supersecretkeyforthexpecto2025XD'
key_base64 = base64.b64encode(key_str.encode())
f = Fernet(key_base64)
plain = f.decrypt(payload)
exec(plain.decode())
```

### Issue
- Fernet requires a 32-byte key before Base64 encoding.
- The given `key_str` is 30 bytes long.
- Base64 encoding it directly doesn't work.

### Solution

- The original script attempts to use a key that is too short for Fernet encryption, leading to a decryption failure. To fix this:
- We ensure the key is exactly 32 bytes long by padding it if necessary.
- The adjusted key is then encoded correctly before being used for decryption.
- After fixing the key issue, the script successfully decrypts the payload, which contains a Python script.
- The exec() function executes the decrypted script, revealing the flag.

#### Fixed Code:
```python
import base64
from cryptography.fernet import Fernet

payload = b'gAAAAABn2vH52liBfuFfazSkLcAVO5N3AaoIJslqczD_GbOi7Ygbq2eKZWF2YYqFSUv34O-eQeH_2DvL-cJZceUmk1Tlp_fOEhXn30uG4tjmgZm0Dfc2LhSDJtFrdYSriHuKOIfpjrs0V6xyo8_Vg2HsUePmXDXFF_mzicZqDX5wLw0NHBFn3fqjsdbavOnqZw0YIhPbEbyaupbzOULllF9VUGvn53D3RFlyHD8M3jY8xAPkj0YY6D2WmmKtDu16XKK4asdLEsYo-7uSGFG-UO6fxdf1ncWXD8pM0EAOYj1Y6tlSH3t9qXE='

key_str = 'supersecretkeyforthexpecto2025XD'
key_32_bytes = key_str.ljust(32)[:32].encode()
key_base64 = base64.urlsafe_b64encode(key_32_bytes)

f = Fernet(key_base64)
decrypted_data = f.decrypt(payload)
print(decrypted_data.decode())
```

### Flag

```
saic{pr1n7ing_ju57_unp4ck3d_my_s3cr3t}
```
---

## Challenge: *Catch Me If You Can*

**Category:** Reverse Engineering

### Description:
> *This EXE lets you copy something… but before you can paste it, poof—it’s gone! Can you outsmart it?*

**Given Files:**
- `clip.exe`

### Solution
The given exe has a copy button and a timer of 0.01 secs, as soon as you copy the flag, the clipboard is cleared in 0.01 secs.
So if your clipboard history is turned on then it is easily available or you can track the the clipboard changes and print it using the following script. 
Run this script and then click on the copy button.


```python
import pyperclip
import time

def check_clipboard():
    original_clipboard = pyperclip.paste()
    
    while True:
        time.sleep(0.01)  
        current_clipboard = pyperclip.paste()

        if current_clipboard != original_clipboard:
            print(f"Clipboard changed: {current_clipboard}")
            break  

check_clipboard()
```

### Flag
```
saic{y0u_f!n@lly_c0p!3d_!t,right?}
```

---

## Challenge: *Crazy-LCS*
**Category:** MISC

### Description

> *The server plays hot-and-cold with your guesses—only telling you how much contiguous chunk matches the flag. No hints, no mercy, just pain. Can you piece it together, or will you substring yourself into madness? 😈🔍*

**Remote Source:** `nc iitmandi.co.in 8098`

### Solution

The server provides the length of the **Longest Common Substring (LCS)** between a guessed string and the hidden flag. To reconstruct the flag:

1. Start with "saic{"
2. Guess characters iteratively, checking the LCS length.
3. If LCS increases, retain the guessed character.
4. Continue until the flag is fully revealed.

<!-- image -->
![Desktop View](/assets/img/chronos-xpecto25/crazy-lcs.png){: width="972" height="589" }
_Crazy-LCS_


The following script automates this process:

```python
import string
import socket
import re

HOST = "iitmandi.co.in"
PORT = 8098
charset = string.ascii_letters + string.digits + "_"
flag = "saic{"

def send_guess(guess):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        s.recv(1024)
        s.sendall(guess.encode() + b"\n")
        response = s.recv(1024).decode().strip()
        match = re.search(r"LCS length: (\d+)", response)
        return int(match.group(1)) if match else 0

current_length = 5

while True:
    best_char = None
    for c in charset:
        test_flag = flag + c
        match_length = send_guess(test_flag)
        if match_length > current_length:
            best_char = c
            current_length = match_length
            break
    if best_char:
        flag += best_char
        print(f"[+] Found so far: {flag}")
        if best_char == "}":
            break
    else:
        print("[-] No progress, exiting...")
        break
flag += "}"
print(f"Flag: {flag} ")
```

### Flag

```
saic{3asy_dp_lc5_ch4ll3ng3}
```

---
## Challenge: *Size Doesn't Matter*

**Category:** MISC

### Description 

> *Maybe she lied...*

### Given files:
- `m100.png`

### Solution

The challenge title "Size Doesn't Matter" and the give description suggested that the image might have incorrect dimensions, making it unreadable or corrupted. PNG files store their width and height in the IHDR chunk, and if these values are incorrect, the image won’t render properly.

To fix this, I used the this [tool](https://github.com/cjharris18/png-dimensions-bruteforcer), which attempts to brute-force correct dimensions for a PNG file.

After running the script, the image was successfully restored:

<!-- image -->
![Desktop View](/assets/img/chronos-xpecto25/fixed.png){: width="972" height="589" }
_Fixed Image_

### Flag

```
saic{s1z3_d03s_ma773r_baby}
```
---
## Challenge: *Back In Time - 5*
**Category:** OSINT

### Description
> *The traitor just checked into a fancy hotel and posted a picture on Instagram—but no location tag! Your mission: analyze their posts, spot unique details, and track down the exact hotel they’re staying at. The clues are there... if you know where to look! 🌍🔍*
Flag Format: saic{hotel_name}
Example: If you find out the picture was taken at Hotel Prestige, the flag would be: saic{hotel_prestige}
Happy hunting, detective! 🕵️‍♂️

### Solution
From the github profile of `Pavitr Prabhakar` found out his username `SpideyPavitr`. Checked this username name on instagram and got the following image:

![Desktop View](/assets/img/chronos-xpecto25/image.webp){: width="972" height="589" }
_Given Image_

On google image search found out that this image was taken near the `Selamat Datang Monument`.

Explored the location on google earth and street view and found out that the hotel is `Grand Hyatt Jakarta`.

### Flag
```
saic{grand_hyatt_jakarta}
```
---