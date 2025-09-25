# Cracking the Vault: A Nostalgic Hack at Zero Nights 2017 🚪💻

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lpglhiktyyjm5c7owcvw.png)

## 0. Introduction: Where Hackers Family Reunites

Picture this: **Moscow, November 2017**. The air is crisp, the coffee is strong, and the room buzzes with the energy of hundreds of security enthusiasts wearing everything from **corporate polos to hacker hoodies**. This was **Zero Nights** - not just another security conference, but something closer to a **family reunion for the Russian infosec community**. 

Back in my university days (circa 2009-2010), I had cut my teeth on **CTF competitions** with my team **Cr@zY Geek$** . Those late nights solving challenges created bonds that lasted years. Now, as a established security specialist and journalist for **"Hacker" magazine**, I was returning to this playground with both my notebook and my curiosity ready for action. Little did I know I'd soon be part of a team that would **crack an analog safe** using nothing but wits and patience - no brute force allowed!

## 1. Zero Nights 2017: More Than Just Talks

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p7yne23moa5wt2csw3sc.png)


**[Zero Nights](https://xakep.ru/2017/11/30/zn2017-results/)** wasn't your typical stuffy security conference. It had established itself as Russia's answer to **DEF CON**  with its unique blend of cutting-edge research, hands-on challenges, and unforgettable parties. The 2017 edition continued this tradition with:

- **Mind-bending talks** on everything from critical infrastructure vulnerabilities to novel web exploitation techniques
- **Live hacking villages** where attendees could try their skills on various systems
- **Hardware hacking zone** filled with gadgets begging to be disassembled
- **The infamous "safe cracking challenge"** that would become my obsession for the next 24 hours

What made Zero Nights special was its **emphasis on community** rather than commercialization. Unlike corporate events, this felt like a **gathering of friends** who happened to be obsessed with breaking things to make them stronger - the true hacker ethos in action!

## 2. The Safe-Cracking Challenge: No Brute Force Allowed! 🎯🔒

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o8p2bijec1br2cjnkida.png)

The challenge appeared simple on the surface: **"Open the analog safe without force"**. The vintage **Soviet-era safe** stood in the corner almost mockingly, its dial seeming to dare hackers to try their luck. The rules were straightforward:

1. **No physical damage** to the safe whatsoever
2. **No power tools** or destructive methods
3. **Teams of up to 5 people** could participate
4. **First to open it** would win glory and prizes

For a crowd used to **binary exploitation** and **web app attacks**, this mechanical beast presented a completely different kind of challenge. We couldn't rely on our usual arsenal of **scripts, debuggers, or exploit code** . This was back-to-basics security research at its finest - understanding a system through analysis rather than aggression.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7ufe0ed5w8l47nqhwk3u.png)

## 3. Assembling the A-Team: Hackers vs. Hardware 🤝🔧

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e4mdfgkyyn3t34lv43gy.png)

As a journalist, I had come to cover the event, but the challenge was too tempting to resist. 

We were the **perfect ragtag team** of digital specialists facing an analog problem. Our first approaches were predictably "digital-minded":

*   "Maybe there's a **side-channel attack** on the dial?"
*   "Could we **acoustically analyze** the clicks?"
*   "What about **thermal imaging** to see the mechanism?"

After several hilarious failed attempts (including trying to use a **stethoscope app** on someone's smartphone), we stepped back to actually understand how mechanical safes work.

## 4. The Eureka Moment: Listening to the Mechanics 👂🔊

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3ervh4cddtk73n0likpg.png)

Mechanical safes typically use a **wheel pack** mechanism where each wheel corresponds to a number in the combination. When the dial is turned, there are subtle **auditory and tactile feedback points** that indicate when a wheel is correctly aligned. Our breakthrough came when we realized:

- The **third number** had the **least resistance** when aligned correctly
- There was a barely perceptible **"click"** that could be felt more than heard
- The **dial had slight variations** in tension at certain points

We divided responsibilities: Dmitry with his sensitive hands focused on the tactile feedback, Anna with her perfect pitch listened for auditory clues, Max kept precise track of positions, Irina researched similar safe models on her phone, and I coordinated our efforts and documented the process.

After **three hours** of meticulous testing (and several coffee breaks), we had mapped out likely numbers for each part of the combination. The real magic happened when we realized the mechanism required **specific sequencing** rather than just raw numbers - something that reminded me of **timing attacks** in cryptography .

## 5. Victory! The Sweet Sound of Success 🏆🎉

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2ltdzpxzzf9npwzvpy64.png)

The moment of truth came at **4:37 PM** on the second day of the conference. After countless iterations and adjustments based on our observations, Dmitry slowly turned the dial through what we believed was the correct combination: **17-33-49**.

There was a **satisfying clunk** that was music to our ears. The handle turned smoothly, and the heavy door swung open to reveal... well, honestly just some conference swag inside, but that wasn't the point! We had **outsmarted**, not overpowered, the mechanical beast.

The crowd that had gathered around cheered - at hacker conferences, everyone celebrates others' successes because we all understand the thrill of the solve. In that moment, we weren't competitors but fellow explorers who had collectively cracked a puzzle.

## 6. Lessons from the Safe: Why Hardware Hacking Matters 🔍📠

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yre21v9cgzx6k8u9o37e.png)

This exercise taught us valuable lessons that applied directly to our digital security work:

1.  **Systems have physical elements** that can be exploited - something often forgotten in cloud-centric security
2.  **Patience and observation** often beat brute force in both analog and digital domains
3.  **Interdisciplinary teams** bring diverse perspectives that crack tough problems
4.  **Understanding how something is built** is the first step to understanding how to break it

The safe challenge embodied the true **hacker spirit**: we didn't want to damage the safe, we wanted to understand it. Just like in **CTF competitions** , the goal wasn't destruction but mastery through comprehension - breaking to build better.

## 7. Conclusion: Keeping the Hacker Spirit Alive 🧡🔓

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r9wvmguqzlyr50121t1g.png)

Looking back at **Zero Nights 2017** through the lens of time, I'm filled with **nostalgia** for that incredible community atmosphere. The safe-cracking challenge wasn't just a fun diversion - it was a **living example of hacker culture** at its best:

- **Curiosity-driven exploration** without malicious intent
- **Collaborative problem-solving** that transcended competitive instincts
- **Respect for systems** even while figuring out how to bypass them
- **Pure joy of learning** through hands-on experience

In our increasingly digital world, there's something special about **physical challenges** that bring people together in shared space and time. The **emotions and highs** from that safe-cracking victory stayed with me far longer than any corporate penetration test finding.

So here's to the **hackers, the breakers, the curious minds** who take things apart to see how they work - may our culture of creative exploration never fade! Whether it's **exploiting a binary** , **finding a web vulnerability** , or **cracking an analog safe**, the spirit remains the same: **"It's there, let's see how it works!"**

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0xarb6voyof7x2k0ia4v.png)

*// D3One, nostalgic hacker and safe cracker extraordinaire*

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n3ewxt56i2c2vx6zm2rz.png)

## **Understanding the Adversary: How a Combination Lock Works**

Most dial combination locks on safes use a simple mechanism called a **wheel pack**.
1.  The dial is connected to a spindle.
2.  The spindle runs through several wheels (usually 3 or 4 for a 3 or 4-digit code).
3.  Each wheel has a notch (called a "gate") cut into it.
4.  When you turn the dial, you are aligning these wheels.
5.  A metal lever (the "fence") drops into these aligned notches only when the correct combination is entered, allowing the bolt to be retracted and the safe to open.

The trick is to find where those notches are *without* knowing the combination.

---

### **The Hacker's Toolbox: What You Need**

1.  **Your Hands:** For feeling subtle resistance.
2.  **Your Ears:** For listening to clicks. A quiet environment is crucial.
3.  **A Stethoscope (Optional but highly recommended):** The classic tool. The cup is placed on the safe's door near the lock mechanism to amplify internal sounds. In a pinch, a glass or a screwdriver held to your ear and against the door can work.
4.  **Patience and a Notepad:** This is a methodical process.

---

### **Step-by-Step Attack Plan: Cracking the Code**

Let's assume a standard 3-number combination lock (e.g., 45-20-75). The process is similar for 4-number locks, just longer.

**Step 1: Find the Contact Points (The "Sticking" Points)**

This step finds the last number of the combination.
1.  **Turn the dial clockwise (to the right) at least four full rotations.** This ensures all wheels are engaged and you're starting from a known state.
2.  **Place your stethoscope on the door near the dial.**
3.  **Very, very slowly turn the dial counterclockwise (to the left).** Listen and feel intently.
4.  You will feel and hear a subtle **"click"** or a point of increased resistance approximately every few numbers. These are not the gates, but points where the drive pin contacts a wheel.
5.  **Write down all these "sticking" points.** You might get 6-12 of them. For example: `12, 19, 32, 45, 58, 63, 75, 88`.
6.  **The correct last number is usually the one that sticks the *most* or has the loudest click, and it will be a number that is a multiple of the lock's "drive point" (often 4 or 5 on many locks).** From our list, `45`, `75`, and `88` are candidates. `75` is often a very common one. **Let's assume our last number is 75.**

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dzxpnlffg4l6abeszwws.png)

**Step 2: Find the First Number**

This is the most time-consuming part.
1.  **Turn the dial clockwise (right) at least four full rotations** to reset.
2.  **Turn left to your first candidate for the first number.** Let's start with `12`.
3.  **Now, with slight pressure on the dial, turn it slowly to the right.** You are trying to feel for the point where the fence "drops" into the gate of the first wheel.
4.  You are looking for a **"false gate"** – a point where the dial becomes slightly harder to turn for a few numbers and then gets easier again. The *width* of this hard spot is important. Note it down.
5.  Repeat this process for every single number on the dial (0-99) or at least for all the sticking points you found in Step 1. This is why it takes hours.
6.  **The correct first number will have a noticeably *wider* "false gate" area than all the others.** For example, all other numbers might have a hard spot 2 numbers wide, but when you hit `45`, the hard spot might be 5-6 numbers wide.
7.  **Let's assume we found that `45` has the widest false gate. So our first number is 45.** Our partial combo is `45 - ?? - 75`.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dzxpnlffg4l6abeszwws.png)

**Step 3: Find the Middle Number (Brute Force the Last Digit)**

Now that you have the first and last numbers, the middle number is easy to brute force.
1.  **Turn the dial right at least four times.**
2.  **Turn left and stop on your first number: `45`.**
3.  **Turn right one full rotation and stop on `45` again.** (This is crucial to engage the second wheel).
4.  **Now, turn right very slowly from `45` towards your last number (`75`).**
5.  As you turn, you will feel the fence "fall" into the gate of the middle wheel. **The dial will "hiccup," stutter, or feel loose for a moment.** This is the magic moment!
6.  **Note the number where this "hiccup" occurs.** This is your middle number. Let's say it happens at `20`.
7.  **You now have the full combination: `45 - 20 - 75`.**

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dzxpnlffg4l6abeszwws.png)

**Step 4: Open the Safe**
1.  Turn right 4 times to reset.
2.  Turn left, stop on **45**.
3.  Turn right, pass **45** once, stop on **20** on the second time.
4.  Turn left, stop directly on **75**.
5.  Turn the handle. The safe should open.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gl1w904qrjs99caqsmlt.png)

### **Example of a Successful Session (Abridged)**

*   **Investigator:** (After 30 mins of slow turning) "Okay, sticking points noted: 10, 25, 40, 55, 70, 85. The one at 40 feels the strongest. Let's tentatively set the last number to 40."
*   **Investigator:** (After 2 hours of testing every number) "All the false gates are 2 numbers wide except for 70. The dial got really stiff from 68 to 74. That's 6 numbers wide! First number must be 70."
*   **Investigator:** "Okay, last number 40, first number 70. Now to find the middle... Let's go to 70, rotate right once, back to 70, and now slowly towards 40... come on... come on... There! A tiny slip at 15! Got it!"
*   **Investigator:** "Combination is 70... 15... 40." (Turns dial: R4, L to 70, R past 70 once to 15, L directly to 40). *CLUNK*.
*   **Investigator:** "And we're in."

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gl1w904qrjs99caqsmlt.png)

*What's your most memorable hacking challenge? Share your stories below!* 
