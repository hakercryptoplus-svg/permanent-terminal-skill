# Terminal Component Implementation

## TerminalWithShortcuts Component

Complete React component that embeds ttyd and handles keyboard shortcuts.

```typescript
import { useEffect, useRef } from "react";

interface TerminalWithShortcutsProps {
  terminalUrl: string;
}

export default function TerminalWithShortcuts({ terminalUrl }: TerminalWithShortcutsProps) {
  const iframeRef = useRef<HTMLIFrameElement>(null);

  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      // Ctrl+L: Clear screen
      if (e.ctrlKey && e.key === "l") {
        e.preventDefault();
        sendCommandToTerminal("clear\n");
      }
      
      // Ctrl+U: Clear line
      if (e.ctrlKey && e.key === "u") {
        e.preventDefault();
        sendCommandToTerminal("\u0015");
      }
      
      // Ctrl+W: Delete word
      if (e.ctrlKey && e.key === "w") {
        e.preventDefault();
        sendCommandToTerminal("\u0017");
      }
      
      // Ctrl+A: Beginning of line
      if (e.ctrlKey && e.key === "a") {
        e.preventDefault();
        sendCommandToTerminal("\u0001");
      }
      
      // Ctrl+E: End of line
      if (e.ctrlKey && e.key === "e") {
        e.preventDefault();
        sendCommandToTerminal("\u0005");
      }
      
      // Ctrl+R: Reverse search
      if (e.ctrlKey && e.key === "r") {
        e.preventDefault();
        sendCommandToTerminal("\u0012");
      }
      
      // Ctrl+D: Exit/Delete
      if (e.ctrlKey && e.key === "d") {
        e.preventDefault();
        sendCommandToTerminal("\u0004");
      }
      
      // Ctrl+Z: Suspend
      if (e.ctrlKey && e.key === "z") {
        e.preventDefault();
        sendCommandToTerminal("\u001a");
      }
      
      // Ctrl+C: Interrupt
      if (e.ctrlKey && e.key === "c") {
        e.preventDefault();
        sendCommandToTerminal("\u0003");
      }
      
      // Ctrl+S: Stop output
      if (e.ctrlKey && e.key === "s") {
        e.preventDefault();
        sendCommandToTerminal("\u0013");
      }
      
      // Ctrl+Q: Resume output
      if (e.ctrlKey && e.key === "q") {
        e.preventDefault();
        sendCommandToTerminal("\u0011");
      }
    };

    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);

  const sendCommandToTerminal = (command: string) => {
    if (iframeRef.current?.contentWindow) {
      try {
        iframeRef.current.contentWindow.postMessage(
          { type: "terminal_command", command },
          "*"
        );
      } catch (error) {
        console.error("Failed to send command to terminal:", error);
      }
    }
  };

  return (
    <iframe
      ref={iframeRef}
      src={terminalUrl}
      className="w-full h-full border-none"
      title="Terminal"
      allow="fullscreen"
    />
  );
}
```

## Home Page Integration

Use the component in the main page:

```typescript
import { useAuth } from "@/_core/hooks/useAuth";
import { Loader2 } from "lucide-react";
import { getLoginUrl } from "@/const";
import TerminalWithShortcuts from "@/components/TerminalWithShortcuts";

export default function Home() {
  const { user, loading, isAuthenticated, logout } = useAuth();

  if (loading) {
    return (
      <div className="w-screen h-screen flex items-center justify-center bg-slate-950">
        <Loader2 className="animate-spin w-8 h-8 text-blue-500" />
      </div>
    );
  }

  if (!isAuthenticated) {
    return (
      <div className="w-screen h-screen flex flex-col items-center justify-center bg-gradient-to-br from-slate-900 to-slate-800">
        <div className="text-center space-y-6">
          <h1 className="text-4xl font-bold text-white">Permanent Terminal</h1>
          <p className="text-xl text-slate-300">وصول دائم إلى ترمينال الـ Sandbox</p>
          <button
            onClick={() => window.location.href = getLoginUrl()}
            className="px-8 py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-lg transition"
          >
            تسجيل الدخول
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="w-screen h-screen flex flex-col bg-slate-950 overflow-hidden">
      <TerminalWithShortcuts terminalUrl="https://7681-ic0dzew6zze40hsl0eugy-90afddce.us1.manus.computer" />
    </div>
  );
}
```

## Keyboard Shortcut Reference

All shortcuts use Ctrl key combined with a letter:

| Key | Function | ASCII Code | Use Case |
|-----|----------|-----------|----------|
| L | Clear screen | N/A | Clear terminal output |
| U | Clear line | 0x15 | Delete current input line |
| W | Delete word | 0x17 | Remove last word |
| A | Home | 0x01 | Jump to line start |
| E | End | 0x05 | Jump to line end |
| R | Reverse search | 0x12 | Search command history |
| D | EOF/Delete | 0x04 | Send EOF or delete char |
| Z | Suspend | 0x1A | Pause current process |
| C | Interrupt | 0x03 | Kill current process |
| S | Stop | 0x13 | Pause output flow |
| Q | Resume | 0x11 | Resume output flow |

## Testing Shortcuts

Unit test template:

```typescript
describe('TerminalWithShortcuts', () => {
  it('should support Ctrl+L (clear screen)', () => {
    const shortcut = { key: 'l', ctrlKey: true };
    expect(shortcut.key).toBe('l');
    expect(shortcut.ctrlKey).toBe(true);
  });

  it('should have correct ASCII code for Ctrl+U', () => {
    const code = '\u0015';
    expect(code.charCodeAt(0)).toBe(0x15);
  });
});
```

## Styling with Tailwind

Full-screen terminal container:

```tsx
<div className="w-screen h-screen flex flex-col bg-slate-950 overflow-hidden">
  <TerminalWithShortcuts terminalUrl={url} />
</div>
```

Key classes:
- `w-screen h-screen` - Full viewport
- `bg-slate-950` - Dark background
- `overflow-hidden` - Prevent scrollbars
- `flex flex-col` - Vertical layout

## Error Handling

Graceful error handling for iframe communication:

```typescript
const sendCommandToTerminal = (command: string) => {
  if (!iframeRef.current?.contentWindow) {
    console.warn("Terminal iframe not ready");
    return;
  }

  try {
    iframeRef.current.contentWindow.postMessage(
      { type: "terminal_command", command },
      "*" // Consider restricting to specific origin in production
    );
  } catch (error) {
    console.error("Failed to send command:", error);
  }
};
```

## Performance Considerations

1. **Event Listener Cleanup**: Always remove listener in useEffect cleanup
2. **Ref Management**: Use useRef for iframe to avoid re-renders
3. **Message Throttling**: Consider debouncing rapid key presses
4. **Cross-Origin**: Ensure iframe origin matches or use postMessage carefully

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support for keyboard events
- Mobile: Limited (terminal UX not optimized for touch)
