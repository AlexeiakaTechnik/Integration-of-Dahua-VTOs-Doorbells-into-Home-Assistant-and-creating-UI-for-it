![image](PLACEHOLDER FOR COVER IMAGE)

# 🚪 Integration of Dahua VTOs (Doorbells) into Home Assistant and creating UI/Scripts/Automations for it. (🚧UNDER CONSTRUCTION🚧)

_This article explains how to fully integrate Dahua VTO (Video Intercom Doorbell) devices into Home Assistant.  
It covers stream handling, actionable UI pop-ups, automation logic, DMSS app fallback, and honest evaluation of current 2-way audio limitations._

---

## 📚 Table of Contents

1. [🔍 Introduction](#-1-introduction)  
2. [🧰 Requirements & Setup Overview](#-2-requirements--setup-overview)  
3. [⚙️ Integrating Dahua VTO in Home Assistant](#-3-integrating-dahua-vto-in-home-assistant)  
4. [📹 Live Stream Access via go2rtc](#-4-live-stream-access-via-go2rtc)  
5. [📞 Handling Doorbell Calls in Home Assistant](#-5-handling-doorbell-calls-in-home-assistant)  
   - 5.1 [Input Booleans, Triggers, and Helpers](#-51-input-booleans-triggers-and-helpers)  
   - 5.2 [TTS Notifications & Timeout Logic](#-52-tts-notifications--timeout-logic)  
   - 5.3 [Pop-up UI via Browser Mod](#-53-pop-up-ui-via-browser-mod)  
6. [📱 Opening the DMSS App from HA](#-6-opening-the-dmss-app-from-ha)  
7. [🔁 Automation Flow Breakdown](#-7-automation-flow-breakdown)  
   - 7.1 [Main Call Automation](#-71-main-call-automation)  
   - 7.2 [Scripts Used in the Workflow](#-72-scripts-used-in-the-workflow)  
8. [🎙️ Attempts at 2-Way Audio (go2rtc & SIP)](#-8-attempts-at-2-way-audio-go2rtc--sip)  
   - 8.1 [What Works & What Breaks](#-81-what-works--what-breaks)  
   - 8.2 [Why HA Can’t Do 2-Way Audio Natively](#-82-why-ha-cant-do-2-way-audio-natively)  
   - 8.3 [SIP Setup Projects (for Advanced Users)](#-83-sip-setup-projects-for-advanced-users)  
9. [🔐 Gate Unlock, Hangup, and Other Actions](#-9-gate-unlock-hangup-and-other-actions)  
10. [🎛️ Dashboard Integration Ideas](#-10-dashboard-integration-ideas)  
11. [🧠 Wishlist: Call Log, Snapshots, Missed Events](#-11-wishlist-call-log-snapshots-missed-events)  
12. [🎬 Live Demo Walkthrough](#-12-live-demo-walkthrough)  
13. [🪪 License](#-13-license)  
14. [👨‍💻 Author and Related Projects](#-14-author-and-related-projects)  

---

## 🔍 1. Introduction [↑](#-table-of-contents)

- What Dahua VTOs are
- Why integrate with HA?
- Features targeted: camera feed, popup alerts, gate unlock, voice comms

---

## 🧰 2. Requirements & Setup Overview [↑](#-table-of-contents)

- Dahua VTO (model used)  
- Home Assistant OS / Supervised  
- go2rtc addon  
- Browser Mod  
- Fully Kiosk on Android (optional but useful)  
- DMSS app (fallback path)

---

## ⚙️ 3. Integrating Dahua VTO in Home Assistant [↑](#-table-of-contents)

- Using the official Dahua integration  
- `button_pressed` and `audio_muted` entities  
- No SIA, MQTT, or SIP required  
- Basic entity structure created by integration

![image](PLACEHOLDER FOR INTEGRATION SCREENSHOTS)

---

## 📹 4. Live Stream Access via go2rtc [↑](#-table-of-contents)

- Configuring go2rtc addon  
- Creating HA camera entities from go2rtc  
- Verifying latency and resolution  
- Bonus: enable audio stream

![image](PLACEHOLDER FOR CAMERA VIEW / STREAM TEST)

---

## 📞 5. Handling Doorbell Calls in Home Assistant [↑](#-table-of-contents)

### 5.1 Input Booleans, Triggers, and Helpers
- Using `input_boolean.doorbell_replied_binary_state` to control repeat behavior  
- Test triggers via dummy `input_button` for manual testing  
- Entity trigger vs helper override

### 5.2 TTS Notifications & Timeout Logic
- Repeating TTS call announcements  
- Stopping only when someone answers  
- Timeout fallback → persistent notification

```yaml
repeat:
  while:
    - condition: state
      entity_id: input_boolean.doorbell_replied_binary_state
      state: "on"
  sequence:
    - service: tts.speak
      data:
        message: "Incoming Call at the Gate"
```

### 5.3 Pop-up UI via Browser Mod
- Pop-up with camera feed (picture-elements)  
- Action buttons: unlock, DMSS, hangup, 2-way  
- Duplicate popups for phone and tablet browsers

![image](PLACEHOLDER FOR POPUP UI SCREENSHOTS)

---

## 📱 6. Opening the DMSS App from HA [↑](#-table-of-contents)

- Using Fully Kiosk `start_application`  
- Fallback when go2rtc 2-way fails  
- Works on tablets and phones

```yaml
service: fully_kiosk.start_application
data:
  application: com.mm.android.DMSSHD
```

---

## 🔁 7. Automation Flow Breakdown [↑](#-table-of-contents)

### 7.1 Main Call Automation
- Triggering event  
- Waking devices  
- Navigating to dashboard  
- Popping up UI  
- Repeating flow to ensure reliability

### 7.2 Scripts Used in the Workflow
- `l1_doorbell_opengate_pressed`  
- `l1_doorbell_hangup`  
- `l1_0_doorbell_opendmss`  
- `l1_doorbell_open_2way_audio`  
- `l1_doorbell_camera_pressed`

```yaml
alias: l1_0_doorbell_opendmss
sequence:
  - service: input_boolean.turn_off
    target:
      entity_id: input_boolean.doorbell_replied_binary_state
  - service: fully_kiosk.start_application
    data:
      application: com.mm.android.DMSSHD
```

---

## 🎙️ 8. Attempts at 2-Way Audio (go2rtc & SIP) [↑](#-table-of-contents)

### 8.1 What Works & What Breaks
- You tested go2rtc 2-way via iframe  
- Works *sometimes*, usually only after doorbell reboot  
- Cannot rely on it for production

### 8.2 Why HA Can’t Do 2-Way Audio Natively
- HA lacks a SIP/media stack  
- No core support for microphone input/output control  
- Audio routing is non-existent

### 8.3 SIP Setup Projects (for Advanced Users)
- Asterisk server examples  
- GitHub references:  
  - [DIY SIP to HA integration (GitHub)](https://github.com/search?q=home+assistant+asterisk+doorbell)  
  - [FOSCAM SIP Doorbell Proxy](https://community.home-assistant.io/t/intercom-doorbell-with-2-way-audio/469083)

---

## 🔐 9. Gate Unlock, Hangup, and Other Actions [↑](#-table-of-contents)

- Unlock button triggers physical relay  
- Hangup stops announcements  
- DMSS buttons cancel popup + launch app  
- Optional feedback in UI or log

---

## 🎛️ 10. Dashboard Integration Ideas [↑](#-table-of-contents)

- Dedicated “Incoming Call” tab  
- Picture Glance card with latest snapshot  
- Manual DMSS launch / unlock buttons  
- "Recent Events" log

---

## 🧠 11. Wishlist: Call Log, Snapshots, Missed Events [↑](#-table-of-contents)

- Automate snapshot on button press  
- Archive into media folder or Google Drive  
- Create Missed Call dashboard card  
- Track answered/unanswered calls

---

## 🎬 12. Live Demo Walkthrough [↑](#-table-of-contents)

_A short YouTube or MP4 video showing:_

- Real-time call trigger  
- Popup on both tablet and phone  
- Unlocking the gate  
- Switching to DMSS  
- 2-way audio demo attempt

![image](PLACEHOLDER FOR YOUTUBE THUMBNAIL OR GIF)

---

## 🪪 13. License [↑](#-table-of-contents)

MIT or CC BY-NC-SA — choose your flavor of open source.

---

## 👨‍💻 14. Author and Related Projects [↑](#-table-of-contents)

_Alexei aka Technik_  
Building practical smart home solutions with robust UI and fallback logic.

- [📱 HA Dashboards for Tablets & Phones](https://github.com/AlexeiakaTechnik/Practial-and-stylish-Home-Assistant-Dashboards-for-Tablets-and-Mobile-Phones)  
- [🎥 Dahua CCTV System Integration](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant)  
- [🔐 AJAX Security Sensors in HA](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant)
