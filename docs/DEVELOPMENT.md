# Development Guide

## Quick Start

```bash
open GeminiDesktop.xcodeproj
# In Xcode: Cmd+R to build and run
```

## Build Actions

| Action | Shortcut | What it does | When to use |
|--------|----------|--------------|-------------|
| **Build & Run** | Cmd+R | Compiles and launches app from Xcode | Development/testing |
| **Build only** | Cmd+B | Compiles without running | Check for errors |
| **Clean Build** | Cmd+Shift+K | Removes all build artifacts | Fix weird build issues |
| **Archive** | Product → Archive | Creates signed `.app` for distribution | Release builds |

## Where Does the App Live?

| Build Type | Location | Persists? |
|------------|----------|-----------|
| Debug (Cmd+R) | `~/Library/Developer/Xcode/DerivedData/` | Until clean |
| Archive | Xcode Organizer | Yes |
| Installed | `/Applications/` | Yes |

## Development Workflow

**For daily development:**
1. Open `GeminiDesktop.xcodeproj`
2. Press Cmd+R to build and run
3. App launches directly—no installation needed
4. Make changes, Cmd+R again to test

**For permanent installation:**
1. Product → Archive (builds release version)
2. Window → Organizer
3. Select archive → Distribute App → Copy App
4. Drag `.app` to `/Applications`

## Creating a DMG (Distribution)

Only needed if sharing the app with others:

```bash
# 1. Archive in Xcode first, export .app to ~/Downloads/GeminiDesktop/

# 2. Run the DMG script
./scripts/create-dmg.sh

# Output: ~/Downloads/GeminiDesktop/GeminiDesktop.dmg
```

## Debug vs Release

| Build | Performance | Logging | Use for |
|-------|-------------|---------|---------|
| Debug | Slower | Console.log bridged to Xcode | Development |
| Release | Optimized | Minimal logging | Distribution |

To switch: Product → Scheme → Edit Scheme → Run → Build Configuration

## Common Issues

**"App not signed" warning:**
- Normal for local builds without Apple Developer account
- Right-click app → Open to bypass Gatekeeper

**WebView not loading:**
- Check internet connection
- Reset website data in Settings

**Build fails after Xcode update:**
- Clean build folder (Cmd+Shift+K)
- Delete DerivedData: `rm -rf ~/Library/Developer/Xcode/DerivedData/GeminiDesktop-*`
