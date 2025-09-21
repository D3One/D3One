
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

**Step 2: Find the First Number**

This is the most time-consuming part.
1.  **Turn the dial clockwise (right) at least four full rotations** to reset.
2.  **Turn left to your first candidate for the first number.** Let's start with `12`.
3.  **Now, with slight pressure on the dial, turn it slowly to the right.** You are trying to feel for the point where the fence "drops" into the gate of the first wheel.
4.  You are looking for a **"false gate"** – a point where the dial becomes slightly harder to turn for a few numbers and then gets easier again. The *width* of this hard spot is important. Note it down.
5.  Repeat this process for every single number on the dial (0-99) or at least for all the sticking points you found in Step 1. This is why it takes hours.
6.  **The correct first number will have a noticeably *wider* "false gate" area than all the others.** For example, all other numbers might have a hard spot 2 numbers wide, but when you hit `45`, the hard spot might be 5-6 numbers wide.
7.  **Let's assume we found that `45` has the widest false gate. So our first number is 45.** Our partial combo is `45 - ?? - 75`.

**Step 3: Find the Middle Number (Brute Force the Last Digit)**

Now that you have the first and last numbers, the middle number is easy to brute force.
1.  **Turn the dial right at least four times.**
2.  **Turn left and stop on your first number: `45`.**
3.  **Turn right one full rotation and stop on `45` again.** (This is crucial to engage the second wheel).
4.  **Now, turn right very slowly from `45` towards your last number (`75`).**
5.  As you turn, you will feel the fence "fall" into the gate of the middle wheel. **The dial will "hiccup," stutter, or feel loose for a moment.** This is the magic moment!
6.  **Note the number where this "hiccup" occurs.** This is your middle number. Let's say it happens at `20`.
7.  **You now have the full combination: `45 - 20 - 75`.**

**Step 4: Open the Safe**
1.  Turn right 4 times to reset.
2.  Turn left, stop on **45**.
3.  Turn right, pass **45** once, stop on **20** on the second time.
4.  Turn left, stop directly on **75**.
5.  Turn the handle. The safe should open.

### **Example of a Successful Session (Abridged)**

*   **Investigator:** (After 30 mins of slow turning) "Okay, sticking points noted: 10, 25, 40, 55, 70, 85. The one at 40 feels the strongest. Let's tentatively set the last number to 40."
*   **Investigator:** (After 2 hours of testing every number) "All the false gates are 2 numbers wide except for 70. The dial got really stiff from 68 to 74. That's 6 numbers wide! First number must be 70."
*   **Investigator:** "Okay, last number 40, first number 70. Now to find the middle... Let's go to 70, rotate right once, back to 70, and now slowly towards 40... come on... come on... There! A tiny slip at 15! Got it!"
*   **Investigator:** "Combination is 70... 15... 40." (Turns dial: R4, L to 70, R past 70 once to 15, L directly to 40). *CLUNK*.
*   **Investigator:** "And we're in."
