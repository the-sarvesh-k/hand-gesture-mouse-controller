# 🖐️ Hand Gesture Mouse Controller

Control your laptop's mouse cursor using just your hand — no mouse, no touchpad, just a webcam and hand gestures.

## 💡 About the Project

I wanted to see if I could control my computer using only hand movements — no physical mouse at all. What started as a simple idea turned into a real debugging journey: camera detection failures, Python version conflicts, MediaPipe installation issues, gesture recognition bugs, and scroll logic that didn't feel natural.

After fixing issue after issue, I ended up with a fully working, smooth, and responsive hand-controlled mouse system.

## ✨ Features

- 🖱️ **Cursor movement** — move your hand to move the mouse pointer
- 👆 **Left click** — via finger gesture recognition
- 🤏 **Drag and drop** support
- 🖱️ **Right click** gesture
- 📜 **Gesture-based scrolling** (up/down)
- ✨ **Smooth cursor movement** with jitter filtering
- 🎯 On-screen gesture status UI (shows current mode: MOVE / CLICK / SCROLL)
- 📊 Live FPS counter
- 🟪 Visual interaction frame + cursor indicator on camera feed

## 🛠️ Tech Stack

- **Python 3**
- **OpenCV** — camera feed & UI rendering
- **MediaPipe** — hand landmark detection
- **CVZone** — simplified hand tracking wrapper
- **PyAutoGUI** — controlling the actual mouse cursor

## 📦 Installation

```bash
git clone https://github.com/the-sarvesh-k/hand-gesture-mouse-controller.git
cd hand-gesture-mouse-controller
pip install -r requirements.txt
python main.py
```

### requirements.txt
```
opencv-python
mediapipe
cvzone
pyautogui
numpy
```

> ⚠️ Note: MediaPipe requires a compatible Python version (3.8–3.11 recommended). If your camera isn't detected, check that no other app is using it and that you're on the correct Python environment.

## 🎮 How to Use

| Gesture | Action |
|---|---|
| Index finger up | Move cursor |
| Index + Middle finger up (in air) | Scroll up |
| Index + Middle finger together (touching) | Scroll down |
| Thumb + Index pinch | Left click |
| Other gesture combos | Right click / Drag |

*(Update this table to match your final gesture mapping.)*

## 🚧 Future Improvements

- 🔊 Gesture-based volume control
- 🔆 Gesture-based brightness control
- 🎨 Full GUI panel with buttons/toggles
- 🔉 Sound feedback on click
- 📦 Packaged as a standalone `.exe` app

## 🙋‍♂️ Lessons Learned

This project taught me more about **patient debugging** than about any single library. Every error forced me to actually understand what MediaPipe, OpenCV, and my webcam were doing — not just copy-paste code.

## 📄 License

This project is licensed under the MIT License.

---

⭐ If you found this project interesting, consider giving it a star!
