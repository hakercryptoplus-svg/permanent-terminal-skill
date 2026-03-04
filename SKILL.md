---
name: permanent-terminal
description: "Build a permanent, web-accessible terminal service with keyboard shortcuts and authentication. Use when creating a Terminal-as-a-Service platform, deploying ttyd with Manus web hosting, adding keyboard shortcuts to terminal interfaces, or building authenticated terminal access systems."
---

# Permanent Terminal

## Overview

This skill enables building a production-ready, permanent terminal service accessible via web browser. It integrates ttyd (web-based terminal) with Manus web hosting, adds 11 popular keyboard shortcuts (Ctrl+L, Ctrl+U, Ctrl+W, etc.), implements user authentication, and provides a full-screen terminal interface.

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

### 3. Keyboard Shortcuts

Implement 11 standard Unix/Linux terminal shortcuts:

| Shortcut | Function | Code |
|----------|----------|------|
| Ctrl+L | Clear screen | `clear` |
| Ctrl+U | Clear line | `\u0015` |
| Ctrl+W | Delete word | `\u0017` |
| Ctrl+A | Beginning of line | `\u0001` |
| Ctrl+E | End of line | `\u0005` |
| Ctrl+R | Reverse search | `\u0012` |
| Ctrl+D | Exit/Delete | `\u0004` |
| Ctrl+Z | Suspend | `\u001a` |
| Ctrl+C | Interrupt | `\u0003` |
| Ctrl+S | Stop output | `\u0013` |
| Ctrl+Q | Resume output | `\u0011` |

**Implementation:** Use React `useEffect` hook to listen for `keydown` events and send commands to iframe via `postMessage`.

### 4. Full-Screen Terminal Interface

- Removes header/footer for maximum screen real estate
- Uses iframe to embed ttyd
- Supports full-screen mode
- Dark theme optimized for terminal work

### 5. Authentication

- Built-in OAuth login via Manus
- User session management
- Logout functionality
- Protected routes

## Quick Start

### Step 1: Initialize Manus Web Project

```bash
# Create project with database and user features
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

See `references/terminal-component.md` for complete React component implementation with keyboard shortcuts.

### Step 4: Update Home Page

Replace the default home page with the terminal interface component.

### Step 5: Deploy

Use Manus web hosting to deploy. The service will be accessible at a permanent domain.

## Implementation Details

### Terminal Component Structure

The `TerminalWithShortcuts` component:

1. Renders an iframe pointing to ttyd instance
2. Attaches keyboard event listener to window
3. Intercepts Ctrl+X key combinations
4. Sends commands to terminal via iframe postMessage API
5. Prevents default browser behavior for shortcuts

### Keyboard Event Handling

```typescript
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.ctrlKey && e.key === "l") {
      e.preventDefault();
      sendCommandToTerminal("clear\n");
    }
    // ... other shortcuts
  };
  
  window.addEventListener("keydown", handleKeyDown);
  return () => window.removeEventListener("keydown", handleKeyDown);
}, []);
```

### Authentication Flow

1. User visits app without login
2. Redirected to OAuth login page
3. After authentication, user can access terminal
4. Session persists across page reloads
5. Logout clears session and redirects to login

## Testing

### Unit Tests

Use vitest to test:
- Shortcut key mappings
- Correct control character codes
- Component rendering
- Event listener attachment

**Run tests:**
```bash
pnpm test
```

### Manual Testing

1. Access the web app
2. Log in with test credentials
3. Test each keyboard shortcut
4. Verify terminal responsiveness
5. Test logout functionality

## Deployment

### Using Manus Web Hosting

1. Create checkpoint: `webdev_save_checkpoint`
2. Click "Publish" button in Management UI
3. Domain assigned automatically (e.g., `manusterminal-xxx.manus.space`)
4. Service is immediately live and permanent

### Custom Domain

Add custom domain in Settings → Domains panel within Manus UI.

## Troubleshooting

### ttyd Not Responding

- Check if process is running: `ps aux | grep ttyd`
- Verify port 7681 is open: `netstat -tuln | grep 7681`
- Check logs: `tail /tmp/ttyd.log`
- Restart: `pkill ttyd && /usr/local/bin/ttyd -p 7681 -c admin:root2026 bash &`

### Keyboard Shortcuts Not Working

- Verify iframe is focused (click inside terminal)
- Check browser console for errors
- Ensure event listener is attached
- Test with simple shortcuts first (Ctrl+L)

### Authentication Issues

- Clear browser cookies
- Check OAuth configuration in server logs
- Verify JWT_SECRET is set
- Restart dev server

## Advanced Features

### Session Recording

Store command history in database:

```typescript
// In server API
POST /api/terminal/command
{
  userId: number,
  command: string,
  output: string,
  timestamp: Date
}
```

### Multi-User Sessions

Implement WebSocket for real-time terminal sharing:

```typescript
// Share terminal session with other users
POST /api/terminal/share
{
  sessionId: string,
  userId: number,
  permissions: "view" | "edit"
}
```

### Custom Themes

Add terminal color scheme customization:

```typescript
// Terminal theme configuration
const themes = {
  dark: { bg: "#1e1e1e", fg: "#ffffff" },
  light: { bg: "#ffffff", fg: "#000000" },
  solarized: { bg: "#002b36", fg: "#839496" }
};
```

## Resources

See bundled references for:
- Complete React component code
- Database schema for session storage
- API endpoint specifications
- Deployment checklist

## References

- [ttyd GitHub](https://github.com/tsl0922/ttyd)
- [Manus Web Hosting Docs](https://docs.manus.im)
- [React Hooks Documentation](https://react.dev/reference/react)
- [Tailwind CSS](https://tailwindcss.com)
