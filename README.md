# Permanent Terminal Skill

A Manus skill for building production-ready, web-accessible terminal services with keyboard shortcuts and authentication.

## Overview

This skill enables you to:

- Deploy ttyd (web-based terminal) on Manus hosting
- Integrate terminal with React frontend
- Add 11 popular keyboard shortcuts (Ctrl+L, Ctrl+U, Ctrl+W, etc.)
- Implement user authentication
- Create a full-screen terminal interface
- Deploy to a permanent domain

## Quick Start

1. **Initialize Manus Web Project**
   ```bash
   webdev_init_project --name permanent-terminal-web --scaffold web-db-user
   ```

2. **Install ttyd**
   ```bash
   curl -L https://github.com/tsl0922/ttyd/releases/download/1.7.3/ttyd.x86_64 -o ttyd
   chmod +x ttyd && sudo mv ttyd /usr/local/bin/ttyd
   ```

3. **Run ttyd**
   ```bash
   /usr/local/bin/ttyd -p 7681 -c admin:root2026 bash &
   ```

4. **Add Terminal Component** - See `references/terminal-component.md`

5. **Deploy** - Use Manus web hosting for permanent domain

## Features

### Keyboard Shortcuts

| Shortcut | Function |
|----------|----------|
| Ctrl+L | Clear screen |
| Ctrl+U | Clear line |
| Ctrl+W | Delete word |
| Ctrl+A | Beginning of line |
| Ctrl+E | End of line |
| Ctrl+R | Reverse search |
| Ctrl+D | Exit/Delete |
| Ctrl+Z | Suspend |
| Ctrl+C | Interrupt |
| Ctrl+S | Stop output |
| Ctrl+Q | Resume output |

### Authentication

- OAuth login via Manus
- User session management
- Protected terminal access

### Deployment

- Permanent domain (e.g., `manusterminal-xxx.manus.space`)
- Automatic HTTPS
- Built-in database
- User management

## Files

- **SKILL.md** - Main skill documentation
- **references/terminal-component.md** - React component implementation
- **references/ttyd-deployment.md** - ttyd setup and configuration

## Requirements

- Ubuntu 22.04 LTS or similar
- Node.js 22.x
- pnpm package manager
- Manus web hosting account

## Documentation

See `SKILL.md` for complete documentation and advanced features.

## License

MIT

## Support

For issues or questions, refer to:
- [ttyd GitHub](https://github.com/tsl0922/ttyd)
- [Manus Documentation](https://docs.manus.im)
- [React Documentation](https://react.dev)
