# picoCTF Writeups

Writeups for seven picoCTF challenges I solved on my own, covering web exploitation, cryptography, forensics, and general skills. Each one follows the same format: goal, what I tried (including what failed), exact steps, and lessons learned.

picoCTF is a free capture the flag platform from Carnegie Mellon University. Flags and challenge instance links are left out on purpose, so these writeups do not spoil the challenges.

| Challenge | Event | Category | Main tool |
|-----------|-------|----------|-----------|
| logon | picoCTF 2019 | Web Exploitation | Browser DevTools |
| Bookmarklet | picoCTF 2024 | Web Exploitation | Browser console |
| Hashcrack | picoCTF 2025 | Cryptography | CrackStation |
| PcapPoisoning | picoCTF 2023 | Forensics | Wireshark |
| Inspect Me | picoCTF | Web Exploitation | Browser DevTools |
| Big Zip | picoCTF | General Skills | PowerShell |
| Crack the Gate 1 | picoCTF 2025 | Web Exploitation | curl |

---

## logon

**Goal:** Log in as a user and get to the hidden content on a web app.

**What I did:** I logged in with arbitrary credentials to start a session, then opened DevTools and checked the cookies. There were three: username, password, and admin. The admin cookie was set to False. I changed it to True and refreshed the page, which gave me elevated access.

**Steps**
1. Open the challenge page and log in with any credentials
2. Open DevTools, then Application, then Cookies
3. Change the admin cookie from False to True
4. Refresh the page

**Lessons learned**
* Never trust client side cookies for access control, because users can change them freely
* Check cookies after logging in, since they show how the app handles authorization
* A value like admin=False in a cookie is a red flag for insecure logic

---

## Bookmarklet

**Goal:** Get the flag from a page that hands you a bookmarklet.

**What I did:** The page displayed a JavaScript bookmarklet that held an encrypted flag and the code to decrypt it with a repeating key. I was not sure how to run a bookmarklet, so I pasted the code into the browser console instead, and it decrypted the flag.

**Steps**
1. Open the challenge page and copy the bookmarklet code
2. Open DevTools, then Console
3. Paste the code and press Enter

**Lessons learned**
* Client side code is fully readable, so any secret in it is not secret
* The console is a fast way to run a script without making a bookmark
* Reading the code first showed me what it would do before I ran it

---

## Hashcrack

**Goal:** Gain access to a secret on a server whose admin used weak password hashes.

**What I did:** I connected to the server with netcat from the picoCTF webshell and got a hash. It was 32 hex characters, so I identified it as MD5 and cracked it on CrackStation. Entering the answer gave me a second hash, which was SHA 1, and then a third, which was SHA 256. I cracked all three the same way.

**Steps**
1. Connect to the server with netcat
2. Identify each hash type from its length
3. Crack each one on CrackStation and submit the result
4. Repeat for the next hash

**Lessons learned**
* Hash length is a quick way to identify the algorithm
* Weak or common passwords fall to lookup tables in seconds
* MD5 and SHA 1 are not safe for storing passwords

---

## PcapPoisoning

**Goal:** Find a flag hidden in a packet capture.

**What I did:** I opened the capture in Wireshark and saw FTP, TCP, and ICMP traffic. My first search for the flag text in the packet list info column found nothing, which was a failed attempt. I then changed the search to look for a string inside the packet bytes, and it hit on packet 507, where the flag was visible.

**Steps**
1. Open the capture in Wireshark
2. Go to Edit, then Find Packet
3. Search for the flag text as a string in Packet bytes
4. Open the matching packet and read the data

**Lessons learned**
* Search scope matters: the info column only shows a summary, so use packet bytes to search the real data
* Failed searches are still useful because they narrow down where to look
* Plain text protocols like FTP show data in the clear

---

## Inspect Me

**Goal:** Find a flag in a site made of HTML, CSS, and JavaScript.

**What I did:** I looked through the page in DevTools. The flag was split into three parts: one in an HTML comment, one in a CSS comment, and one in the JavaScript file.

**Steps**
1. Open DevTools and read the HTML comments
2. Open the Style Editor and read the CSS comments
3. Open the Debugger, open the JavaScript file, and read it
4. Join the three parts

**Lessons learned**
* Check every file a page loads, not just the HTML
* Developers leave notes in comments that never get removed
* Different tabs in DevTools show different parts of the same site

---

## Big Zip

**Goal:** Find a flag inside a huge archive of folders and text files.

**What I did:** Extracting the archive took about ten minutes because of the number of files. Browsing it by hand was not realistic. I do not have grep on Windows, so I used PowerShell to search every file for the flag text, which found it right away.

**Steps**
1. Download and extract the archive
2. Open PowerShell in the extracted folder
3. Run a recursive search for the flag text:

   `ls -r | sls "picoCTF"`

**Lessons learned**
* Use a recursive search tool when a task has thousands of files
* PowerShell has an equivalent for grep: Select-String, or sls for short
* Know the tool for your operating system before you start

---

## Crack the Gate 1

**Goal:** Get into a restricted web portal when you know the login email but not the password.

**What I did:** I first tried common credentials and checked DevTools, and neither worked. Viewing the raw page source showed a scrambled comment at the top of the page. It was ROT13, and decoding it revealed a developer note about a bypass that used a custom HTTP header. I sent a login request with curl and that header, first to the wrong path, which returned an error, then to the login path, which succeeded.

**Steps**
1. View the page source with Ctrl+U
2. Decode the scrambled comment with a ROT13 tool
3. Send a login request with curl that adds the header from the note
4. Use the login path of the app, not the main page

**Lessons learned**
* Always read the raw page source, because DevTools does not show everything
* Encoded notes in source code can reveal authentication bypasses
* A custom header used as a backdoor should never ship in production code

---

All challenges were solved on picoCTF for learning purposes.
