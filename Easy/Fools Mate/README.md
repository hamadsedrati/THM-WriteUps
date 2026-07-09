# Fools Mate
Can you bypass the engine? - by DrGonz0

### Scenario
It's mate in one. You know it, the engine knows it, my grandma knows it. The board says checkmate is one click away. The engine says no. Settle the argument.
You can access the web app from your AttackBox's browser via: http://MACHINE-IP


### Summary
The Fools Mate room is a classic example of why you never trust client-side security. It's a web challenge with a chess board where all the game rules are checked right in your browser using JavaScript, but the backend server doesn't verify a thing. Because of that, you can just pop open your browser's DevTools or use Burp Suite to tweak the code, trick the game into letting you make an illegal checkmate move, and grab the flag. It's the ultimate lesson in why the server always needs to double-check user input.


## Recon
We'll start by going to the website as mentioned in the scenario, and we can see a chess board in the middle, and reset poisition button on the right.
first thing i did was 
![](https://github.com/hamadsedrati/THM-WriteUps/Easy/Fools%25Mate/img/interface.png)
