# Fools Mate
Can you bypass the engine? - by DrGonz0

### Scenario
It's mate in one. You know it, the engine knows it, my grandma knows it. The board says checkmate is one click away. The engine says no. Settle the argument.
You can access the web app from your AttackBox's browser via: http://MACHINE-IP


### Summary
The Fools Mate room is a classic example of why you never trust client-side security. It's a web challenge with a chess board where all the game rules are checked right in your browser using JavaScript, but the backend server doesn't verify a thing. Because of that, you can just pop open your browser's DevTools or use Burp Suite to tweak the code, trick the game into letting you make an illegal checkmate move, and grab the flag. It's the ultimate lesson in why the server always needs to double-check user input.


## Recon
We'll start by going to the website as mentioned in the scenario, and we can see a chess board in the middle, and reset poisition button on the right.
![](https://github.com/hamadsedrati/THM-WriteUps/blob/main/Easy/Fools%20Mate/img/interface.png)

First thing I did was run a simple nmap scan to expose open ports. (bad idea lol)
```bash
nmap -sV MACHINE-IP
```
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-09 14:29 -0400
Nmap scan report for 10.49.146.160
Host is up (0.048s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Node.js Express framework
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.27 seconds
```

now, I'm going to be honest and say that I went down an unnecessary rabbit hole which wasted my time and was completely unneeded :D

Basically, i tried to exploit the outdated OpenSSH 9.6p1 with CVE-2024-6387.... surprise surprise, I didn't get the flag :))
<img width="885" height="156" alt="image" src="https://github.com/user-attachments/assets/6789150d-9c64-4797-86b5-f70b203fda68" />


then, I shifted my focus to the chess website.
Firstly, i tried moving a piece and checking the network tab.
<img width="1915" height="321" alt="image" src="https://github.com/user-attachments/assets/d2c8cf8a-fc1b-4f17-8ecd-84fe7190c8cb" />

Then, I tried the moving the castle piece to A8 (checkmate), and my laptop got a death threat
<img width="380" height="157" alt="image" src="https://github.com/user-attachments/assets/af0ad03c-74f6-402b-b7ba-70448dd543ce" />
but i noticed that I didnt send a request, so i checked the app.js file and found the exact block that gets executed.
<img width="457" height="67" alt="image" src="https://github.com/user-attachments/assets/3b9768df-9384-4af1-bf89-7cbe15a639d7" />

i simply changed it to `return true`, then clicked reset position button.
<img width="1170" height="450" alt="image" src="https://github.com/user-attachments/assets/76cf5456-4ecc-4f05-ac91-c0a4e4327e63" />

