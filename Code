"""
Hand Gesture Mouse Controller
------------------------------
Control your mouse cursor using hand gestures via webcam.

Tech stack: OpenCV, MediaPipe (via cvzone.HandTrackingModule), PyAutoGUI

Gestures:
    - Index finger up only            -> Move cursor
    - Index + Middle up (separated)    -> Scroll up
    - Index + Middle together (touch)  -> Scroll down
    - Thumb + Index pinch              -> Left click
    - Thumb + Index pinch (held)       -> Drag and drop
    - Thumb + Pinky up                 -> Right click

Author: the-sarvesh-k
"""

import cv2
import numpy as np
import time
import math
import pyautogui
from cvzone.HandTrackingModule import HandDetector

# ---------------------------------------------------------
# CONFIGURATION
# ---------------------------------------------------------
CAM_WIDTH, CAM_HEIGHT = 1280, 720          # Camera resolution
FRAME_REDUCTION = 100                       # Interaction box border
SMOOTHENING = 5                             # Cursor smoothing factor
CLICK_DISTANCE_THRESHOLD = 35               # Pixel distance for pinch/click
DRAG_HOLD_TIME = 0.35                        # Seconds pinch must be held to start drag
SCROLL_AMOUNT = 40                          # Scroll step size

SCREEN_WIDTH, SCREEN_HEIGHT = pyautogui.size()
pyautogui.FAILSAFE = False                  # Prevent PyAutoGUI corner-abort

# ---------------------------------------------------------
# INITIALIZATION
# ---------------------------------------------------------
cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)    # CAP_DSHOW helps camera detection on Windows
cap.set(3, CAM_WIDTH)
cap.set(4, CAM_HEIGHT)

detector = HandDetector(maxHands=1, detectionCon=0.8)

prev_x, prev_y = 0, 0
curr_x, curr_y = 0, 0

pTime = 0                       # For FPS calculation
is_dragging = False
pinch_start_time = 0
click_cooldown = 0              # Prevents multiple rapid clicks


def get_distance(p1, p2):
    """Euclidean distance between two (x, y) points."""
    return math.hypot(p2[0] - p1[0], p2[1] - p1[1])


def draw_ui_overlay(img, mode_text, fps):
    """Draws the top status bar, mode text, and FPS counter."""
    overlay = img.copy()
    cv2.rectangle(overlay, (0, 0), (CAM_WIDTH, 90), (0, 0, 0), -1)
    img = cv2.addWeighted(overlay, 0.5, img, 0.5, 0)

    # Mode text (color-coded)
    color = (0, 255, 255)
    if "CLICK" in mode_text:
        color = (0, 255, 0)
    elif "SCROLL UP" in mode_text:
        color = (0, 255, 0)
    elif "SCROLL DOWN" in mode_text:
        color = (0, 0, 255)
    elif "DRAG" in mode_text:
        color = (255, 0, 255)
    elif "RIGHT CLICK" in mode_text:
        color = (255, 165, 0)

    cv2.putText(img, mode_text, (20, 55), cv2.FONT_HERSHEY_SIMPLEX, 1.2, color, 3)
    cv2.putText(img, f'FPS: {int(fps)}', (CAM_WIDTH - 200, 55),
                cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 0, 0), 2)
    return img


# ---------------------------------------------------------
# MAIN LOOP
# ---------------------------------------------------------
while True:
    success, img = cap.read()
    if not success:
        print("Camera not detected. Check if another app is using it.")
        break

    img = cv2.flip(img, 1)  # Mirror view so movement feels natural
    hands, img = detector.findHands(img, flipType=False)

    # Draw the interaction box (region mapped to full screen)
    cv2.rectangle(img, (FRAME_REDUCTION, FRAME_REDUCTION),
                   (CAM_WIDTH - FRAME_REDUCTION, CAM_HEIGHT - FRAME_REDUCTION),
                   (255, 0, 255), 2)

    mode_text = "MOVE"

    if hands:
        hand = hands[0]
        lm_list = hand["lmList"]
        fingers = detector.fingersUp(hand)

        # Landmark positions
        x_index, y_index = lm_list[8][0], lm_list[8][1]     # Index fingertip
        x_middle, y_middle = lm_list[12][0], lm_list[12][1]  # Middle fingertip
        x_thumb, y_thumb = lm_list[4][0], lm_list[4][1]      # Thumb tip
        x_pinky, y_pinky = lm_list[20][0], lm_list[20][1]    # Pinky tip

        # -----------------------------------------------------
        # MODE 1: MOVE CURSOR — only index finger up
        # -----------------------------------------------------
        if fingers == [0, 1, 0, 0, 0]:
            # Map camera coordinates -> screen coordinates
            x_mapped = np.interp(x_index, (FRAME_REDUCTION, CAM_WIDTH - FRAME_REDUCTION),
                                  (0, SCREEN_WIDTH))
            y_mapped = np.interp(y_index, (FRAME_REDUCTION, CAM_HEIGHT - FRAME_REDUCTION),
                                  (0, SCREEN_HEIGHT))

            # Smoothen movement to reduce jitter
            curr_x = prev_x + (x_mapped - prev_x) / SMOOTHENING
            curr_y = prev_y + (y_mapped - prev_y) / SMOOTHENING

            pyautogui.moveTo(SCREEN_WIDTH - curr_x, curr_y)  # Invert X for mirror feel
            cv2.circle(img, (x_index, y_index), 10, (255, 0, 255), cv2.FILLED)

            prev_x, prev_y = curr_x, curr_y
            mode_text = "MOVE"

            if is_dragging:
                pyautogui.mouseUp()
                is_dragging = False

        # -----------------------------------------------------
        # MODE 2: SCROLL — Index + Middle up
        # -----------------------------------------------------
        elif fingers == [0, 1, 1, 0, 0]:
            distance = get_distance((x_index, y_index), (x_middle, y_middle))

            if distance > 40:
                # Fingers apart -> scroll up
                pyautogui.scroll(SCROLL_AMOUNT)
                mode_text = "SCROLL UP"
                cv2.putText(img, "SCROLL UP", (500, 150),
                            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
            else:
                # Fingers together -> scroll down
                pyautogui.scroll(-SCROLL_AMOUNT)
                mode_text = "SCROLL DOWN"
                cv2.putText(img, "SCROLL DOWN", (500, 150),
                            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)

            cv2.line(img, (x_index, y_index), (x_middle, y_middle), (0, 255, 0), 3)

        # -----------------------------------------------------
        # MODE 3: LEFT CLICK / DRAG — Thumb + Index pinch
        # -----------------------------------------------------
        elif fingers[0] == 1 and fingers[1] == 1 and fingers[2] == 0:
            distance = get_distance((x_thumb, y_thumb), (x_index, y_index))
            cv2.line(img, (x_thumb, y_thumb), (x_index, y_index), (255, 0, 0), 3)

            if distance < CLICK_DISTANCE_THRESHOLD:
                cv2.circle(img, (x_index, y_index), 15, (0, 255, 0), cv2.FILLED)

                if pinch_start_time == 0:
                    pinch_start_time = time.time()

                held_duration = time.time() - pinch_start_time

                if held_duration > DRAG_HOLD_TIME:
                    # Held long enough -> start/continue dragging
                    if not is_dragging:
                        pyautogui.mouseDown()
                        is_dragging = True
                    mode_text = "DRAG"
                elif time.time() > click_cooldown:
                    pyautogui.click()
                    click_cooldown = time.time() + 0.4  # Debounce clicks
                    mode_text = "CLICK"
            else:
                pinch_start_time = 0
                if is_dragging:
                    pyautogui.mouseUp()
                    is_dragging = False

        # -----------------------------------------------------
        # MODE 4: RIGHT CLICK — Thumb + Pinky up
        # -----------------------------------------------------
        elif fingers == [1, 0, 0, 0, 1]:
            if time.time() > click_cooldown:
                pyautogui.click(button="right")
                click_cooldown = time.time() + 0.5
            mode_text = "RIGHT CLICK"
            cv2.line(img, (x_thumb, y_thumb), (x_pinky, y_pinky), (255, 165, 0), 3)

        else:
            pinch_start_time = 0
            if is_dragging:
                pyautogui.mouseUp()
                is_dragging = False

    # -----------------------------------------------------
    # FPS CALCULATION
    # -----------------------------------------------------
    cTime = time.time()
    fps = 1 / (cTime - pTime) if (cTime - pTime) > 0 else 0
    pTime = cTime

    img = draw_ui_overlay(img, mode_text, fps)

    cv2.imshow("Hand Gesture Control System v1.0", img)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
