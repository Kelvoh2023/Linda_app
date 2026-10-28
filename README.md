# Linda: presentation app

Two pages that work together:

- `www/index.html`: the Linda app (full screen on a phone, in a device frame on a laptop)
- `www/dashboard.html`: the Safaricom fraud-team console

The rest of `www/` is what makes it installable: `manifest.json`, `sw.js` (works offline), and `icons/`.

## 1. Try it on your laptop (2 minutes)

```
cd www
python3 -m http.server 8000
```

Open http://localhost:8000/index.html in Chrome. Open http://localhost:8000/dashboard.html in a second tab.

Why a server and not a double-click: the install and offline features only work over http(s), not file://.

## 2. Put it online so phones can install it (5 minutes)

1. Go to https://app.netlify.com/drop and drag the `www` folder onto the page.
2. Netlify gives you an https link. Open it in Chrome on an Android phone.
3. Chrome menu, then "Install app" (or "Add to Home screen"). Linda now sits on the home screen, opens full screen and works offline.

GitHub Pages or Vercel work the same way if you prefer them.

## 3. Turn it into an APK

### Easiest: PWABuilder (no Android Studio)

1. Go to https://www.pwabuilder.com and paste your Netlify link.
2. Click Package for stores, then Android.
3. Download the zip. The `.apk` inside installs on any Android phone (allow "install unknown apps").

### Alternative: Capacitor (needs Android Studio)

```
npm init -y
npm i @capacitor/core @capacitor/cli @capacitor/android
npx cap init Linda ke.linda.demo --web-dir www
npx cap add android
npx cap sync
npx cap open android
```

In Android Studio press Run, or Build > Build APK(s).

## 3c. Real messages (Android APK only)

The APK can read the texts on your phone and check them with Linda. In the app, Home shows "Protect my messages". Tap **Allow and turn on**.

What happens:
- Linda reads the last 150 texts and your contacts, checks them on the phone, and lists them in Activity (Alerts and All messages).
- A new scam text while Linda is open gives a notification. Tap it to see why.
- When Linda is closed, a quick check looks for scam words (reverse, you won, refund, links) and sends a notification. Open Linda and it checks everything in full.
- Messages never leave the phone. Only the Report button sends anything, and in this prototype it goes to the demo console.

If Android refuses the permission (common for apps installed from a file, on Android 13 and newer):
1. Open Settings, Apps, Linda.
2. Tap the three dots at the top right and choose **Allow restricted settings**.
3. Go back to Linda and tap Allow again.

How it is built: `android-native/` holds a small Android plugin (reads the inbox and contacts, hears new texts) and a receiver. The GitHub workflow copies them into the Android project and adds the permissions. Calls are still shown from demo triggers: reading real calls needs a different Android service.

Not yet tested on a real phone. If the Actions build fails, send me the error text from the failed step.

## 3b. Real notifications and presenter mode

- In the APK, scam alerts arrive as real Android notifications (the first time you open the app, allow notifications). Tap one and Linda opens that alert. In a browser or the Netlify version, Linda draws a similar banner instead.
- The Demo button is hidden so the app looks like a real app. To trigger scams, double-tap the Linda logo on Home, or open Home, How Linda decides, Presenter mode.
- Calls show as a Linda Call Shield card. A real Android version would use the call-screening service, which needs native code and permissions.

## 4. Demo script (about 3 minutes)

Open the app on the phone and the console on the laptop.

1. Home: "Linda is on your phone and synced with Safaricom's fraud list."
2. Presenter mode (double-tap the logo), scenario 1 (scam SMS): the alert appears before the message is opened. Tap it. Point at the highlighted words and the plain reasons. Tap Report to Safaricom.
3. Switch to the console: the report is there. Tap Confirm and block.
4. Back on the phone, scenario 4 (unknown number, then report it) and scenario 2 (reported number): the call screen is red before you answer.
5. Scenario 3: Customer Care shows a green Verified label. Say: "Only Safaricom can prove this, an outside app can only guess."
6. Scenario 5: the call plus a payment prompt. Linda says stop before the PIN is entered.
7. Scenario 6 (call from a saved contact: blue "In your contacts" label) and scenario 7 (an SMS pretending to be Brian from a new number: Linda notices Brian is saved with a different number).
8. Scenario 8, or the red "I've been scammed" button on Home: Linda asks a few questions, pulls the 10-character M-PESA code and amount out of the pasted message, gives the steps (forward to 456 and 333, call 999, change PIN) and writes a report. Press "Send to Safaricom fraud team" and show the Scam cases panel on the console.
9. Scenario 9 (Send money): the number from the "Brian" message. Tap Check and send: Linda stops you, says the message claimed Brian but M-PESA shows PETER OTIENO. Tap Cancel payment. Scenario 10: a large payment to a new number gets a caution.
10. Family Guardian: Home, Family Guardian, Turn on. Run scenario 1 or 7 or cancel a payment, then scenario 11 (Guardian's phone) to show what the family member sees. They never see messages, PIN, balance or contacts.
11. Console, Scam Radar: regions, last 12 hours, trending wording. Press Simulate customer report a few times, then "Warn all Linda phones". The warning appears on the app's Home screen.
12. Ask Linda: paste a scam message in English and a Sheng one. Toggle EN/SW.
13. How Linda decides: three layers, the model numbers, what Linda covers (Safaricom, any wording, banks and other networks).

## 5. What is real and what is simulated

Real:
- The classifier. A naive Bayes text model trained in the page on 80 example messages (English, Swahili, Sheng). About 87% accuracy on its own 5-fold cross-check.
- The rules (PIN request, reversal, upfront fee, fake prize, link, urgency) and the plain-language reasons.
- The scam-recovery questions, the M-PESA code and amount extraction, the report text and the contact check.
- The report loop between the app and the console (same device, same site).
- Installable, offline, EN/SW toggle.

Simulated (say this out loud):
- Family Guardian alerts, the console warning and the Scam Radar regions use the same shared demo data (same device only). A real version needs a small server and the phone's coarse location or cell data.
- M-PESA names for the send-money check are made-up samples. A real version uses the name M-PESA shows before you confirm.
- The contact list is two sample contacts. A real app reads the phone's contacts with permission.
- For banks and other networks Linda gives steps and a report only. It cannot freeze a bank account or an Airtel line.
- Safaricom's fraud list is a sample list. Production would read the real feed.
- Incoming SMS and calls are triggered by the Demo button. A production Android app would use the SMS and caller-ID services, which need special permissions.
- The payment prompt is a drawn mock of the M-PESA prompt.
- The training messages were written by us. A real version trains on confirmed reports.
- The console and the phone share data only on the same device. A real version needs a small server (Firebase is the quickest way if you want two devices live).

## 6. Edit it

Source is in `src/`. After a change run `node src/build.js` to rebuild `www/`.

- `src/model.js`: training messages, classifier, rules, bilingual texts
- `src/app.html`: the app
- `src/dashboard.html`: the console
- `src/shared.js`: the shared demo state
