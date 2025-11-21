[한국어](README-KR.md)

# Obsidian Excalidraw – Temporary Eraser via S Pen Button (Freedraw Only)

This script allows you to **temporarily switch to the eraser tool by holding the S Pen side button** while using Excalidraw inside Obsidian.

## Overview

In Excalidraw, double-tapping the canvas can switch to the eraser, but it’s inconvenient to perform that gesture repeatedly. I created this script to make the workflow smoother: **Draw → Hold S Pen button → Erase → Resume drawing**

This script has only been tested on **Samsung tablets with the S Pen**, so stability may vary in other environments.

Due to system limitations, it is technically impossible to switch to the eraser _only by pressing the S Pen button_. Instead, the script uses **button + one touch/drag** as a trigger. Mouse right-click support is limited because this script was built primarily for S Pen usage.

---

## ✨ Features

- Active only while using the **Freedraw** tool
- While the S Pen button is held:
  - Touch or slightly drag the canvas **once** → instantly switches to the **eraser**
- After you finish erasing and lift the pen:
  - Automatically returns to the previous drawing tool (freedraw)

---

## 🖊️ How to Use (Recommended for S Pen)

1. In Excalidraw Automate, run:  
   **`Toggle Eraser on Right Click in Freedraw`**
2. Select a drawing tool (freedraw)
3. Draw normally
4. When you want to erase:
   - Hold the **S Pen side button**
   - Touch the screen **once** or lightly drag  
     → Tool switches to **eraser**
5. After one erase action:
   - The tool automatically switches back to freedraw

---

## 🛠 Installation

1. Ensure the **Excalidraw plugin** is installed.
2. In Obsidian, go to:  
   **Settings → Community Plugins → Excalidraw**
3. In the **Basic** tab, check the path for  
   **Excalidraw Automate script folder**
4. Copy all repository files (except the README) into that folder  
   (**Case-sensitive filenames!**)

---

## ⚠️ Notes

- Optimized for Samsung S Pen users.
- Some behaviors may not work correctly with a mouse.
- Automatic switching only occurs while the active tool is **freedraw**.
- Running the script again removes previously registered event listeners to avoid duplication.
