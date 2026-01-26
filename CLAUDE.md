# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gemini Desktop is an unofficial macOS desktop wrapper for Google Gemini (gemini.google.com). It's a native SwiftUI app that embeds the Gemini web interface in a WKWebView, providing a floating chat bar and menu bar integration.

**Target:** macOS 14.0 (Sonoma)+
**Dependency:** [KeyboardShortcuts](https://github.com/sindresorhus/KeyboardShortcuts) (via Swift Package Manager)

## Build & Run

```bash
# Open in Xcode
open GeminiDesktop.xcodeproj

# Build from command line
xcodebuild -project GeminiDesktop.xcodeproj -scheme GeminiDesktop -configuration Debug build

# Create release DMG (requires built app in ~/Downloads/GeminiDesktop/)
./scripts/create-dmg.sh
```

## Architecture

### Core Pattern: Single WebView, Multiple Containers

The app uses a **shared WKWebView** that moves between the main window and chat bar panel. This is critical because:
- WebView maintains session state (Google login)
- Only one view hierarchy can contain the WebView at a time
- `WebViewContainer` handles attachment when its window becomes key

```
GeminiDesktopApp (@main)
    ├── Window (main)
    │   └── MainWindowView → GeminiWebView → WebViewContainer
    ├── Settings
    │   └── SettingsView
    └── MenuBarExtra
        └── MenuBarView

AppCoordinator (@Observable)
    ├── webViewModel: WebViewModel (owns the single WKWebView)
    └── chatBar: ChatBarPanel? (floating NSPanel)
```

### Key Components

| File | Responsibility |
|------|---------------|
| `AppCoordinator.swift` | Central state manager. Handles navigation, zoom, chat bar lifecycle, and window switching. |
| `WebViewModel.swift` | Owns the WKWebView. Handles navigation state (canGoBack/canGoForward), zoom persistence, URL observation. |
| `ChatBarPanel.swift` | Floating NSPanel that stays on top. Auto-expands when conversation starts (via JS polling). Dismisses on outside click or ESC. |
| `GeminiWebView.swift` | NSViewRepresentable wrapper. Coordinator handles navigation delegate, downloads, media permissions. |
| `UserScripts.swift` | JavaScript injected into WebView: IME fix for Chinese/Japanese input, console.log bridge (debug only). |

### Window Management

- **Main Window ↔ Chat Bar**: Mutually exclusive. Opening one hides the other.
- **Chat Bar positioning**: Always appears on the screen containing the mouse cursor.
- **Dock icon hiding**: When `hideDockIcon` is true, app runs as `accessory` (menu bar only).

### Important Behaviors

1. **External links**: URLs not on `gemini.google.com`, `accounts.google.com`, or `*.googleapis.com`/`*.gstatic.com` open in the system browser (see `isExternalURL()` in `GeminiWebView.swift:133`).

2. **Chat bar auto-expand**: Polls every 1 second to detect if user is in a conversation (presence of `response-container` or rating buttons). Expands to 70% screen height when detected.

3. **IME fix**: JavaScript in `UserScripts.swift` handles the double-enter bug when using Chinese/Japanese/Korean input methods.

## User Preferences (UserDefaults)

| Key | Type | Purpose |
|-----|------|---------|
| `panelWidth`, `panelHeight` | Double | Chat bar panel size (persisted on resize) |
| `pageZoom` | Double | WebView zoom level (0.6–1.4) |
| `hideWindowAtLaunch` | Bool | Start with main window hidden |
| `hideDockIcon` | Bool | Run as menu bar-only app |

## Debug Mode

In DEBUG builds:
- Console.log messages from the WebView are bridged to Xcode console with `[WebView]` prefix
- IME-related debug logs appear with `[IME Debug]` prefix
