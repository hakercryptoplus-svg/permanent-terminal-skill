---
name: permanent-terminal
description: "Build a permanent, web-accessible terminal service with keyboard shortcuts and authentication. Use when creating a Terminal-as-a-Service platform, deploying ttyd with Manus web hosting, adding keyboard shortcuts to terminal interfaces, or building authenticated terminal access systems."
---

# Permanent Terminal

## Overview

This skill enables building a production-ready, permanent terminal service accessible via web browser. It integrates ttyd (web-based terminal) with Manus web hosting, adds 22 popular keyboard shortcuts and commands (Ctrl+L, Ctrl+U, Ctrl+W, Tab, Enter, etc.), implements user authentication, and provides a full-screen terminal interface with optional shortcuts bar for mobile users.

The resulting service is immediately deployable and accessible via a permanent domain (e.g., `manusterminal-xxx.manus.space`).

## Core Capabilities

### 1. Terminal Deployment (ttyd)

Deploy a web-accessible terminal using ttyd binary:

- Download and install ttyd from GitHub releases
- Expose on port 7681 with credentials
- Support for bash shell with full command execution
- Persistent background service

**Key command:**
```bash
/usr/local/bin/ttyd -p 7681 -c admin:root2026 bash
```

### 2. Web Integration with Manus

Embed the terminal in a Manus web project:

- Use `web-db-user` scaffold for full-stack capabilities
- React frontend with TypeScript
- Built-in authentication system
- Responsive design with Tailwind CSS

### 3. Keyboard Shortcuts (Desktop & Mobile)

Implement 22 standard Unix/Linux terminal shortcuts organized in 6 categories:

**Control Keys (4):**
| Shortcut | Function | Code |
|----------|----------|------|
| Ctrl+C | Interrupt | `\u0003` |
| Ctrl+D | Exit/Delete | `\u0004` |
| Ctrl+Z | Suspend | `\u001a` |
| Ctrl+L | Clear screen | `clear\n` |

**Line Editing (5):**
| Shortcut | Function | Code |
|----------|----------|------|
| Ctrl+A | Beginning of line | `\u0001` |
| Ctrl+E | End of line | `\u0005` |
| Ctrl+U | Clear line | `\u0015` |
| Ctrl+W | Delete word | `\u0017` |
| Ctrl+R | Reverse search | `\u0012` |

**Flow Control (2):**
| Shortcut | Function | Code |
|----------|----------|------|
| Ctrl+S | Stop output | `\u0013` |
| Ctrl+Q | Resume output | `\u0011` |

**Common Keys (4):**
| Shortcut | Function | Code |
|----------|----------|------|
| Tab | Tab completion | `\t` |
| Enter | New line | `\n` |
| Backspace | Delete character | `\u0008` |
| Space | Space character | ` ` |

**Navigation (4):**
| Shortcut | Function | Code |
|----------|----------|------|
| ↑ Up | Previous command | `\u001b[A` |
| ↓ Down | Next command | `\u001b[B` |
| ← Left | Move left | `\u001b[D` |
| → Right | Move right | `\u001b[C` |

**Special Keys (4):**
| Shortcut | Function | Code |
|----------|----------|------|
| Escape | Cancel | `\u001b` |
| Delete | Delete character | `\u001b[3~` |
| Home | Start of line | `\u001b[H` |
| End | End of line | `\u001b[F` |

### 4. Mobile-Friendly Shortcuts Bar

Optional floating shortcuts bar at bottom of screen:

- Collapsible/expandable design
- 6 category tabs for organization
- 22 clickable buttons for all shortcuts
- Responsive grid layout (3-6 columns)
- Works on phones, tablets, and desktop
- Dark theme optimized for terminal

**Features:**
- Toggle button to show/hide
- Category filtering
- One-click command execution
- Touch-friendly button sizes
- Keyboard support for desktop

### 5. Full-Screen Terminal Interface

- Removes header/footer for maximum screen real estate
- Uses iframe to embed ttyd
- Supports full-screen mode
- Dark theme optimized for terminal work
- Responsive padding for shortcuts bar

### 6. Authentication

- Built-in OAuth login via Manus
- User session management
- Logout functionality
- Protected routes

## Quick Start

### Step 1: Initialize Manus Web Project

```bash
webdev_init_project \
  --name permanent-terminal-web \
  --scaffold web-db-user \
  --title "Permanent Terminal"
```

### Step 2: Install and Run ttyd

```bash
# Download ttyd
curl -L https://github.com/tsl0922/ttyd/releases/download/1.7.3/ttyd.x86_64 \
  -o ttyd && chmod +x ttyd && sudo mv ttyd /usr/local/bin/ttyd

# Run in background
/usr/local/bin/ttyd -p 7681 -c admin:root2026 bash > /tmp/ttyd.log 2>&1 &
```

### Step 3: Create Terminal Component

See `references/terminal-component.md` for complete React component implementation.

### Step 4: Add Shortcuts Bar

See `references/shortcuts-bar-component.md` for mobile-friendly shortcuts bar implementation.

### Step 5: Update Home Page

Replace the default home page with the terminal interface and shortcuts bar.

### Step 6: Deploy

Use Manus web hosting to deploy. The service will be accessible at a permanent domain.

## Implementation Details

### Terminal Component Structure

The `TerminalWithShortcuts` component:

1. Renders an iframe pointing to ttyd instance
2. Attaches keyboard event listener to window
3. Intercepts Ctrl+X key combinations
4. Sends commands to terminal via iframe postMessage API
5. Prevents default browser behavior for shortcuts
6. Exposes global `sendTerminalCommand` function

### Shortcuts Bar Component

The `ShortcutsBar` component:

1. Displays 22 shortcuts organized in 6 categories
2. Collapsible/expandable design
3. Category tabs for filtering
4. Responsive grid layout
5. Touch-friendly button sizes
6. Calls `sendTerminalCommand` on button click

### Integration Flow

```
User clicks button in ShortcutsBar
    ↓
ShortcutsBar calls onCommand callback
    ↓
Home.tsx calls window.sendTerminalCommand()
    ↓
TerminalWithShortcuts sends command to iframe
    ↓
ttyd receives command and executes it
```

## Testing

### Unit Tests

Use vitest to test:

- All 22 shortcuts and their codes
- Correct ASCII/control character codes
- Category organization
- Component rendering
- Mobile button functionality

**Run tests:**
```bash
pnpm test
```

### Manual Testing

1. Access the web app
2. Log in with test credentials
3. Click shortcuts bar toggle
4. Test each button category
5. Verify commands execute in terminal
6. Test on mobile device
7. Test keyboard shortcuts on desktop

## Deployment

### Using Manus Web Hosting

1. Create checkpoint: `webdev_save_checkpoint`
2. Click "Publish" button in Management UI
3. Domain assigned automatically (e.g., `manusterminal-xxx.manus.space`)
4. Service is immediately live and permanent

### Custom Domain

Add custom domain in Settings → Domains panel within Manus UI.

## Troubleshooting

### Shortcuts Bar Not Showing

- Check if component is imported in Home.tsx
- Verify CSS classes are applied
- Check browser console for errors
- Ensure Tailwind CSS is loaded

### Buttons Not Working

- Verify iframe is focused
- Check if `sendTerminalCommand` is exposed
- Test with keyboard shortcuts first
- Clear browser cache

### Mobile Layout Issues

- Test on actual mobile device
- Check viewport meta tags
- Verify responsive grid classes
- Test touch interactions

## Advanced Features

### Session Recording

Store command history in database:

```typescript
POST /api/terminal/command
{
  userId: number,
  command: string,
  timestamp: Date
}
```

### Custom Button Groups

Add custom button groups for specific workflows:

```typescript
const customButtons = [
  { label: "ls -la", command: "ls -la\n" },
  { label: "git status", command: "git status\n" },
  { label: "npm start", command: "npm start\n" },
];
```

### Theme Customization

Customize terminal colors and shortcuts bar theme:

```typescript
const themes = {
  dark: { bg: "#1e1e1e", fg: "#ffffff" },
  light: { bg: "#ffffff", fg: "#000000" },
};
```

## Resources

See bundled references for:
- Complete React component code
- Shortcuts bar implementation
- Database schema for session storage
- API endpoint specifications
- Deployment checklist

## References

- [ttyd GitHub](https://github.com/tsl0922/ttyd)
- [Manus Web Hosting Docs](https://docs.manus.im)
- [React Hooks Documentation](https://react.dev/reference/react)
- [Tailwind CSS](https://tailwindcss.com)
