# Challenges & How I Solved Them — Day 1

This file tracks blockers I encountered during labs and how I resolved them. Documenting this helps me learn from mistakes and may help others hitting the same issues.

---

## Challenge 1: MFA QR Code Not Scanning

**What happened:**
The Authenticator app wouldn't scan the QR code shown during MFA setup in the AWS Console.

**What I tried:**
- Refreshed the page (this resets the QR code — worked after a fresh load)
- Made sure phone camera permissions were enabled for the app

**Resolution:**
Refreshing the page generated a new QR code that scanned successfully on the first attempt.

**Lesson learned:**
Don't close or refresh the QR code screen mid-setup or you'll need to restart the MFA flow.

---

## Challenge 2: Navigating to Billing Dashboard

**What happened:**
Couldn't find the Billing and Cost Management dashboard from the main console search.

**What I tried:**
- Searched "billing" in the services search bar — found it as "Billing and Cost Management"
- Used the account dropdown in the top-right corner as an alternative route

**Resolution:**
Both methods work. The account dropdown → "Billing and Cost Management" is the fastest route.

**Lesson learned:**
Billing is not under "Services" by default — it lives under the account menu.

---

## Challenge 3: Budget Alert Email Not Arriving Immediately

**What happened:**
After setting up the Zero Spend Budget with an email notification, no confirmation email arrived right away.

**What I tried:**
- Checked spam/junk folder
- Verified the email address entered was correct

**Resolution:**
The confirmation email arrived after approximately 10 minutes. AWS budget alert emails are not instant.

**Lesson learned:**
Budget alerts are not real-time. There can be a delay between the alert trigger and the email notification.

---

*Add new challenges here as they come up in future days.*
