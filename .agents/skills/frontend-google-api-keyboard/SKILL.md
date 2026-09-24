---
name: frontend-google-api-keyboard
description: >-
  Guidelines and design patterns for building frontend-only virtual keyboards
  using Google APIs (Translate TTS, Input Tools) and a 12-column flexbox grid.
---

# Frontend Google API Keyboard

## Overview
This skill instructs the agent on how to build and maintain pure frontend (HTML/JS/CSS) virtual keyboards or language tools. It enforces specific design patterns for integrating undocumented Google APIs (such as Translate TTS and Input Tools) without a backend, bypassing hotlinking restrictions, and adhering to the user's strict 12-column flexbox grid layout preferences.

## Dependencies
None

## Quick Start
When asked to modify or create a virtual keyboard in this workspace, refer to the **Workflow & Guidelines** section below to ensure the layout matches the user's visual preferences and the API calls bypass browser security blocks correctly.

## Workflow & Guidelines

### 1. Audio and TTS (Text-to-Speech)
- **API Endpoint:** Use Google Translate TTS (`https://translate.googleapis.com/translate_tts?client=tw-ob&tl=zh-TW&q=...`).
- **Bypassing Hotlink Protection:** When deploying to GitHub Pages or running locally, Google TTS will block requests containing an unexpected `Referer` header. 
- **Implementation:** You **MUST** instantiate the audio object in JavaScript and set `referrerPolicy = "no-referrer"` before assigning the `src`.
  ```javascript
  const url = `https://translate.googleapis.com/translate_tts?client=tw-ob&ie=UTF-8&tl=zh-TW&q=${encodeURIComponent(text)}`;
  const audio = new Audio();
  audio.referrerPolicy = "no-referrer"; // CRITICAL to bypass GitHub Pages 403 Forbidden
  audio.src = url;
  audio.play();
  ```
- **DO NOT** use `crossOrigin = "anonymous"`, as it will cause local `file://` execution to fail.
- **Fallback:** If Google TTS completely fails in the future, automatically downgrade to `window.speechSynthesis`.

### 2. Context-Aware Character Selection (Input Tools)
- Use the undocumented Google Input Tools API for Pinyin or Zhuyin conversion:
  `https://inputtools.google.com/request?text=[PINYIN_OR_BOPOMOFO]&itc=zh-hant-t-i0-pinyin&num=5`
- Note that `itc=zh-hant-t-i0-pinyin` works better for full sentence context than the Bopomofo equivalent.

### 3. Keyboard Layout & Grid Design
- **Flexbox Auto-Scaling:** Do NOT use CSS Grid or fixed aspect ratios. Use Flexbox (`flex: 1 1 0`) on the `.key` elements so that all keys automatically and proportionally scale to fill `100%` of the row width.
- **Identical Row Widths:** The total width of all rows (e.g. Row 1 and Row 2) must perfectly match without using fixed widths. 
- **Spacers:** If a row has fewer structural keys than the row above it, remove the spacer entirely to let Flexbox proportionally widen the remaining keys. Currently, the preference is that keys scale automatically and rows share identical total width.
- **Punctuation Placement:** Punctuation keys (e.g., `,`, `.`, `?`) must be placed sequentially on the far right edge, positioned logically underneath the Backspace (`⌫`) key.

### 4. Fullscreen Immersion
- Provide a Fullscreen button (`⛶ 全螢幕`).
- Call `document.documentElement.requestFullscreen()`.
- Upon entering fullscreen, toggle a CSS class on the main wrapper to stretch it fully:
  ```css
  .wrapper.fullscreen {
      width: 100vw !important;
      height: 100vh !important;
      max-width: none;
      position: fixed;
      top: 0; left: 0;
      border-radius: 0; border: none; resize: none; z-index: 9999;
  }
  ```

## Common Mistakes
1. **Forgetting `no-referrer`:** Reverting to `new Audio(url)` will break audio on GitHub Pages.
2. **Hardcoding key widths:** Using `flex: 0 0 calc(...)` prevents proportional scaling. Stick to `flex: 1 1 0` without spacers if you want rows of different item counts to share the same total width.
