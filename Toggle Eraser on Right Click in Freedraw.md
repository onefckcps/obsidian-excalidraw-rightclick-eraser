/* 
This script enables a temporary eraser mode while using the freedraw (pen) tool.
When you hold the right mouse button (or the S Pen side button) and touch the canvas in freedraw mode, the active tool immediately switches to the eraser so you can erase.
When you release the button / lift the pen, the tool automatically returns to the original freedraw tool.

```javascript
*/
if (!ea.verifyMinimumPluginVersion || !ea.verifyMinimumPluginVersion("1.9.11")) {
  new Notice("This script requires a newer version of Excalidraw. Please install the latest version.");
  return;
}

const api = ea.getExcalidrawAPI();
if (!api) {
  new Notice("Excalidraw API not available.");
  return;
}

// Find Excalidraw container
let container = null;
if (ea.targetView?.excalidrawContainer) {
  container = ea.targetView.excalidrawContainer;
} else if (ea.targetView?.contentEl) {
  container =
    ea.targetView.contentEl.querySelector('[class*=\"excalidraw\"]') ||
    ea.targetView.contentEl;
}

const targetElement = container || window;

// Remove existing listeners (to avoid duplicates)
if (targetElement._rightClickEraserHandlers) {
  const handlers = targetElement._rightClickEraserHandlers;
  targetElement.removeEventListener("pointerdown", handlers.pointerdown, true);
  targetElement.removeEventListener("pointermove", handlers.pointermove, true);
  targetElement.removeEventListener("pointerup", handlers.pointerup, true);
  targetElement.removeEventListener("pointercancel", handlers.pointercancel, true);
  targetElement.removeEventListener("contextmenu", handlers.contextmenu, true);
}

// -------------------------------
// State and helper functions
// -------------------------------

let isEraserMode = false;
let originalTool = null;

// Determine whether the eraser button (secondary button) is pressed
const isEraseButtonDown = (e) => {
  // Typical buttons bitfield:
  // 1 = primary, 2 = secondary (right), 4 = middle
  return (e.buttons & 2) === 2;
};

// Check if current tool is freedraw
const isFreeDraw = () => {
  const appState = api.getAppState();
  return appState.activeTool?.type === "freedraw";
};

// Switch to eraser mode
const switchToEraser = () => {
  if (isEraserMode) return;
  if (!isFreeDraw()) return;

  const appState = api.getAppState();
  originalTool = appState.activeTool?.type || "freedraw";
  api.setActiveTool({ type: "eraser" });

  isEraserMode = true;
};

// Restore original tool
const restoreTool = () => {
  if (!isEraserMode) return;

  api.setActiveTool({ type: originalTool || "freedraw" });
  isEraserMode = false;
  originalTool = null;
};

// -------------------------------
// Event handlers
// -------------------------------

// pointerdown: handles cases where the button is already pressed at touch-down
const onPointerDown = (e) => {
  if (isEraseButtonDown(e) && isFreeDraw()) {
    e.preventDefault();
    e.stopPropagation();
    switchToEraser();
  }
};

// pointermove: detects S Pen button press even if it appears only after small movement
const onPointerMove = (e) => {
  if (isEraseButtonDown(e)) {
    if (!isEraserMode && isFreeDraw()) {
      switchToEraser();
    }
  } else {
    if (isEraserMode && e.buttons === 0) {
      restoreTool();
    }
  }
};

// pointerup: restore when all buttons are released
const onPointerUp = (e) => {
  if (isEraserMode && !isEraseButtonDown(e) && e.buttons === 0) {
    restoreTool();
  }
};

// pointercancel: gesture canceled (leaving window etc.)
const onPointerCancel = (e) => {
  if (isEraserMode) restoreTool();
};

// contextmenu: prevents browser menu + supports environments that trigger right-click only here
const onContextMenu = (e) => {
  const appState = api.getAppState();
  const currentTool = appState.activeTool?.type;

  if (currentTool === "freedraw") {
    e.preventDefault();
    e.stopPropagation();
    if (!isEraserMode) switchToEraser();
  } else if (isEraserMode) {
    e.preventDefault();
    e.stopPropagation();
  }
};

// -------------------------------
// Register listeners
// -------------------------------

targetElement._rightClickEraserHandlers = {
  pointerdown: onPointerDown,
  pointermove: onPointerMove,
  pointerup: onPointerUp,
  pointercancel: onPointerCancel,
  contextmenu: onContextMenu
};

targetElement.addEventListener("pointerdown", onPointerDown, true);
targetElement.addEventListener("pointermove", onPointerMove, true);
targetElement.addEventListener("pointerup", onPointerUp, true);
targetElement.addEventListener("pointercancel", onPointerCancel, true);
targetElement.addEventListener("contextmenu", onContextMenu, true);

new Notice("Temporary eraser on right-click / S Pen button enabled for the freedraw tool.");