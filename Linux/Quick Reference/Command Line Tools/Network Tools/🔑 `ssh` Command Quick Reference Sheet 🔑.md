## Installation 📥
```bash
# Linux
sudo apt install openssh-client    # Debian/Ubuntu
sudo yum install openssh-clients   # CentOS/RHEL
sudo pacman -S openssh             # Arch Linux

# macOS
# Pre-installed on macOS

# Windows
choco install openssh              # Using Chocolatey
winget install Microsoft.OpenSSH   # Using winget
# Or use PuTTY or Windows Subsystem for Linux
```

## Basic Usage
```bash
ssh username@hostname              # Connect to remote host
ssh username@192.168.1.100         # Connect using IP address
ssh -p 2222 username@hostname      # Connect to specific port
```

## Common Options 🔍
```bash
ssh -i key.pem username@hostname   # Use specific identity file
ssh -v username@hostname           # Verbose mode for debugging
ssh -X username@hostname           # Enable X11 forwarding
ssh -L 8080:localhost:80 user@host # Local port forwarding
ssh -C username@hostname           # Enable compression
```

## SSH Keys 🔐
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id username@hostname

# Specify identity file location
ssh -i ~/.ssh/id_rsa username@hostname
```

## SSH Config File ⚙️
```bash
# Create/edit ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User username
    Port 2222
    IdentityFile ~/.ssh/id_rsa

# Then simply use:
ssh myserver
```

## Common Examples 🚀
```bash
# Copy files using SCP (SSH Copy)
scp file.txt username@hostname:/path/to/destination

# Run a command on remote server without login shell
ssh username@hostname "ls -la /var/log"

# Create a tunnel for MySQL
ssh -L 3306:localhost:3306 username@hostname

# Jump through a bastion host
ssh -J bastion_user@bastion_host end_user@end_host
```

## SSH Agent 🤖
```bash
# Start SSH agent
eval $(ssh-agent)

# Add key to agent
ssh-add ~/.ssh/id_rsa

# List keys in agent
ssh-add -l
```

## Troubleshooting 🔧
```bash
# Debug connection issues
ssh -vvv username@hostname

# Check SSH server status
systemctl status sshd

# Test connection without logging in
ssh -T username@hostname

# Check permissions on key files
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

## Further Reading 📚
- Official Documentation: [OpenSSH Manual](https://www.openssh.com/manual.html)
- Man Pages: `man ssh`, `man ssh_config`, `man scp`
- Additional Resources:
  - [SSH Academy](https://www.ssh.com/academy/ssh)
  - [SSH.com Documentation](https://www.ssh.com/ssh/)
  - [Digital Ocean SSH Tutorial](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
  - [Arch Linux SSH Wiki](https://wiki.archlinux.org/index.php/OpenSSH)

---