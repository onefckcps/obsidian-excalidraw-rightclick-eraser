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

let container = null;
if (ea.targetView?.excalidrawContainer) {
  container = ea.targetView.excalidrawContainer;
} else if (ea.targetView?.contentEl) {
  container =
    ea.targetView.contentEl.querySelector('[class*=\"excalidraw\"]') ||
    ea.targetView.contentEl;
}

const targetElement = container || window;

if (targetElement._rightClickEraserHandlers) {
  const handlers = targetElement._rightClickEraserHandlers;
  targetElement.removeEventListener("pointerdown", handlers.pointerdown, true);
  targetElement.removeEventListener("pointermove", handlers.pointermove, true);
  targetElement.removeEventListener("pointerup", handlers.pointerup, true);
  targetElement.removeEventListener("pointercancel", handlers.pointercancel, true);
  targetElement.removeEventListener("contextmenu", handlers.contextmenu, true);
}

let isEraserMode = false;
let originalTool = null;
let syntheticFlag = false;

// Check both e.button (which button triggered) and e.buttons (which are held)
const isEraseButton = (e) => {
  return e.button === 2 || (e.buttons & 2) === 2;
};

const isFreeDraw = () => {
  const appState = api.getAppState();
  return appState.activeTool?.type === "freedraw";
};

const switchToEraser = () => {
  if (isEraserMode) return;
  if (!isFreeDraw()) return;

  const appState = api.getAppState();
  originalTool = appState.activeTool?.type || "freedraw";
  isEraserMode = true;
  api.setActiveTool({ type: "eraser" });
};

const restoreTool = () => {
  if (!isEraserMode) return;

  isEraserMode = false;
  api.setActiveTool({ type: originalTool || "freedraw" });
  originalTool = null;
};

// Re-dispatch the blocked right-click as a left-click so Excalidraw starts the eraser stroke
const dispatchSyntheticDown = (original) => {
  syntheticFlag = true;
  const synth = new PointerEvent("pointerdown", {
    bubbles: true,
    cancelable: true,
    composed: true,
    pointerId: original.pointerId,
    pointerType: original.pointerType,
    clientX: original.clientX,
    clientY: original.clientY,
    screenX: original.screenX,
    screenY: original.screenY,
    pageX: original.pageX,
    pageY: original.pageY,
    width: original.width,
    height: original.height,
    pressure: original.pressure || 0.5,
    tiltX: original.tiltX,
    tiltY: original.tiltY,
    isPrimary: original.isPrimary,
    button: 0,
    buttons: 1,
  });
  original.target.dispatchEvent(synth);
  syntheticFlag = false;
};

// Block the right-click, switch to eraser, then re-dispatch as left-click
const onPointerDown = (e) => {
  if (syntheticFlag) return;

  if (isEraseButton(e) && (isFreeDraw() || isEraserMode)) {
    e.preventDefault();
    e.stopPropagation();
    switchToEraser();
    dispatchSyntheticDown(e);
  }
};

const onPointerMove = (e) => {
  if (isEraseButton(e) && !isEraserMode && isFreeDraw()) {
    switchToEraser();
  }
};

// Only restore on pen lift, not on button release
const onPointerUp = (e) => {
  if (isEraserMode) {
    restoreTool();
  }
};

const onPointerCancel = (e) => {
  if (isEraserMode) restoreTool();
};

// Fallback: activate eraser via contextmenu if pointerdown detection missed it
const onContextMenu = (e) => {
  if (isFreeDraw() || isEraserMode) {
    e.preventDefault();
    e.stopPropagation();
    if (!isEraserMode) switchToEraser();
  }
};

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
