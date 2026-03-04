# ttyd Deployment Guide

## What is ttyd?

ttyd is a simple command-line tool that shares your terminal over the web. It allows you to access a bash shell through a web browser with full terminal functionality.

**GitHub:** https://github.com/tsl0922/ttyd

## Installation

### Download Binary

```bash
# Download latest release
curl -L https://github.com/tsl0922/ttyd/releases/download/1.7.3/ttyd.x86_64 \
  -o ttyd

# Make executable
chmod +x ttyd

# Move to system path
sudo mv ttyd /usr/local/bin/ttyd

# Verify installation
ttyd --version
```

### Alternative: Build from Source

```bash
git clone https://github.com/tsl0922/ttyd.git
cd ttyd
mkdir build && cd build
cmake ..
make
sudo make install
```

## Running ttyd

### Basic Usage

```bash
# Simple: accessible on localhost:7681
ttyd bash

# With credentials
ttyd -c admin:password bash

# Custom port
ttyd -p 8080 bash

# Specific interface
ttyd -i 0.0.0.0 bash
```

### Production Configuration

```bash
# Full configuration
/usr/local/bin/ttyd \
  --port 7681 \
  --interface 0.0.0.0 \
  --writable \
  --cred admin:root2026 \
  --timeout 0 \
  --ping-interval 15 \
  --max-clients 10 \
  bash &
```

**Options:**
- `--port 7681` - Listen port
- `--interface 0.0.0.0` - Bind to all interfaces
- `--writable` - Allow terminal input
- `--cred admin:root2026` - Username:password
- `--timeout 0` - No idle timeout
- `--ping-interval 15` - Keep-alive interval (seconds)
- `--max-clients 10` - Max concurrent connections

### Running in Background

```bash
# Using nohup
nohup /usr/local/bin/ttyd -p 7681 -c admin:root2026 bash > /tmp/ttyd.log 2>&1 &

# Using systemd (recommended for production)
# Create /etc/systemd/system/ttyd.service
[Unit]
Description=ttyd Web Terminal
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ttyd -p 7681 -c admin:root2026 bash
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl enable ttyd
sudo systemctl start ttyd
```

## Exposing to Public Internet

### Using Manus expose Tool

```bash
# Expose port 7681 to public URL
expose --port 7681 --brief "Expose ttyd terminal"
# Returns: https://7681-xxxxx.manus.computer
```

### Using ngrok (Alternative)

```bash
# Download ngrok
curl https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip | unzip ngrok

# Expose terminal
./ngrok http 7681
# Public URL: https://xxxx-xx-xxx-xxx.ngrok.io
```

### Using Cloudflare Tunnel

```bash
# Install cloudflared
curl -L --output cloudflared.tgz https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.tgz
tar -xzf cloudflared.tgz

# Create tunnel
./cloudflared tunnel create my-terminal

# Route traffic
./cloudflared tunnel route dns my-terminal example.com
./cloudflared tunnel run my-terminal --url http://localhost:7681
```

## Monitoring

### Check Process Status

```bash
# Is ttyd running?
ps aux | grep ttyd

# Is port open?
netstat -tuln | grep 7681

# Check logs
tail -f /tmp/ttyd.log
```

### Health Check

```bash
# Test connectivity
curl -I http://localhost:7681

# Test with credentials
curl -I -u admin:root2026 http://localhost:7681
```

## Security Considerations

### Authentication

- Always use `--cred` to set username/password
- Change default credentials in production
- Consider using OAuth integration

### Network Security

- Restrict access to trusted networks
- Use HTTPS/TLS (via reverse proxy)
- Implement rate limiting
- Monitor for unauthorized access

### Firewall Rules

```bash
# Allow only specific IPs
sudo ufw allow from 192.168.1.0/24 to any port 7681

# Or use reverse proxy (nginx)
# See nginx-reverse-proxy.conf in templates/
```

## Troubleshooting

### Port Already in Use

```bash
# Find process using port 7681
lsof -i :7681

# Kill process
kill -9 <PID>

# Or use different port
ttyd -p 7682 bash
```

### Connection Refused

```bash
# Check if ttyd is running
ps aux | grep ttyd

# Restart ttyd
pkill ttyd
/usr/local/bin/ttyd -p 7681 -c admin:root2026 bash &
```

### High CPU Usage

```bash
# Reduce ping interval
ttyd --ping-interval 30 bash

# Limit concurrent clients
ttyd --max-clients 5 bash
```

### Browser Shows Blank Screen

- Check browser console for errors
- Verify credentials are correct
- Check firewall rules
- Try different browser
- Clear browser cache

## Performance Tuning

### For High Concurrency

```bash
# Increase file descriptors
ulimit -n 65536

# Run with more resources
/usr/local/bin/ttyd \
  --max-clients 100 \
  --ping-interval 30 \
  bash
```

### For Low Bandwidth

```bash
# Reduce update frequency
ttyd --ping-interval 60 bash

# Limit client connections
ttyd --max-clients 3 bash
```

## Integration with Manus Web Project

### Environment Variables

```bash
# In .env or server config
TERMINAL_URL=https://7681-xxxxx.manus.computer
TERMINAL_USER=admin
TERMINAL_PASS=root2026
TERMINAL_PORT=7681
```

### Docker Deployment

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl

RUN curl -L https://github.com/tsl0922/ttyd/releases/download/1.7.3/ttyd.x86_64 \
  -o /usr/local/bin/ttyd && chmod +x /usr/local/bin/ttyd

EXPOSE 7681

CMD ["/usr/local/bin/ttyd", "-p", "7681", "-c", "admin:root2026", "bash"]
```

## Advanced Features

### Custom Shell

```bash
# Use zsh instead of bash
ttyd -p 7681 zsh

# Use Python REPL
ttyd -p 7681 python3

# Use custom script
ttyd -p 7681 /path/to/script.sh
```

### Environment Variables

```bash
# Pass environment to terminal
ttyd bash -c 'export MY_VAR=value; bash'

# Or set before running
MY_VAR=value ttyd bash
```

### Command Execution

```bash
# Run specific command on connect
ttyd bash -c 'echo "Welcome"; bash'

# Restricted shell (limited commands)
ttyd bash -c 'restricted_bash'
```

## References

- [ttyd GitHub Repository](https://github.com/tsl0922/ttyd)
- [ttyd Documentation](https://github.com/tsl0922/ttyd/wiki)
- [ttyd API Reference](https://github.com/tsl0922/ttyd/blob/master/docs/API.md)
