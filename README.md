# Ansible Deploy

An Ansible automation project for deploying a Node.js quiz application with user management across multiple servers.

## 📋 Overview

This project automates the deployment of:
- **Server1**: User management server with SSH key deployment
- **Server2**: Node.js application server with Nginx reverse proxy

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐
│     Server1     │    │     Server2     │
│  192.168.11.86  │    │  192.168.11.87  │
│                 │    │                 │
│ •User Management│    │ • Node.js App   │
│ •SSH Keys       │    │ • PM2 Process   │
│                 │    │ • Nginx Proxy   │
└─────────────────┘    └─────────────────┘
```

## 🚀 Features

### Server1 (Management Server)
- Creates development users (dev1, dev2, dev3)
- Sets up user passwords with secure SHA512 hashing
- Deploys SSH public keys for passwordless authentication
- Configures bash shell and home directories

### Server2 (Application Server)
- Deploys Node.js quiz application from GitHub
- Sets up PM2 process management
- Configures Nginx reverse proxy
- Manages environment variables
- Automatic dependency installation

## 📁 Project Structure

```
ansible_Deploy/
├── README.md
├── inventory.ini           # Server inventory
├── site.yml               # Main playbook
└── roles/
    ├── server1/
    │   ├── tasks/main.yml  # User management tasks
    │   └── vars/main.yml   # User definitions & SSH key
    └── server2/
        ├── defaults/main.yml    # App configuration
        ├── tasks/main.yml       # App deployment tasks
        └── templates/
            ├── env.j2           # Environment template
            └── nginx.conf.j2    # Nginx config template
```

## ⚙️ Configuration

### Inventory Setup
Edit `inventory.ini` to match your server details:
```ini
[server1]
192.168.11.86 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa

[server2]
192.168.11.87 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Application Configuration
Key settings in `roles/server2/defaults/main.yml`:
- **Repository**: `https://github.com/aacc1on/quiz_MVC.git`
- **Domain**: `quizzzzz.camdvr.org`
- **Port**: `3000`
- **Admin Credentials**: admin/admin123

## 🔧 Prerequisites

### Control Machine
- Ansible installed
- SSH access to target servers
- Private key configured (`~/.ssh/id_rsa`)

### Target Servers
- Ubuntu/Debian-based systems
- Root SSH access
- Internet connectivity for package installation

## 🚀 Deployment

### Quick Start
```bash
# Clone the repository
git clone <your-repo-url>
cd ansible_Deploy

# Test connectivity
ansible all -i inventory.ini -m ping

# Run the full deployment
ansible-playbook -i inventory.ini site.yml
```

### Individual Server Deployment
```bash
# Deploy only user management (Server1)
ansible-playbook -i inventory.ini site.yml --limit server1

# Deploy only application (Server2)
ansible-playbook -i inventory.ini site.yml --limit server2
```

## 🔐 Security Configuration

### User Management (Server1)
The playbook creates three development users:
- `dev1` / `pass1`
- `dev2` / `pass2` 
- `dev3` / `pass3`

**Security Notes:**
- Passwords are hashed using SHA512
- SSH public key authentication is enabled
- Users are added to the root group

### Application Security (Server2)
- Environment variables for sensitive data
- Session management with configurable secrets
- Nginx proxy for additional security layer

## 🌐 Application Access

After deployment, the quiz application will be available at:
- **URL**: `http://quizzzzz.camdvr.org`
- **Admin Panel**: Login with admin/admin123
- **Port**: Application runs on port 3000, proxied through Nginx

## 📝 Environment Variables

The application uses these environment variables:
```bash
PORT=3000
NODE_ENV=development
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin123
SESSION_SECRET=supersecret
OPENROUTER_API_KEY=<ai_api_key>
```

## 🔄 Process Management

The Node.js application is managed by PM2:
```bash
# Check application status
pm2 list

# View logs
pm2 logs myapp

# Restart application
pm2 restart myapp
```

## 🐛 Troubleshooting

### Common Issues

**SSH Connection Failed**
```bash
# Test SSH connectivity
ssh -i ~/.ssh/id_rsa root@192.168.11.86
```

**Application Not Starting**
```bash
# Check PM2 status
pm2 list
pm2 logs myapp

# Check Nginx status
sudo systemctl status nginx
```

**Domain Not Resolving**
- Ensure DNS is configured for `quizzzzz.camdvr.org`
- Check Nginx configuration: `sudo nginx -t`

## 🔧 Customization

### Adding Users
Edit `roles/server1/vars/main.yml`:
```yaml
users:
  - { name: newuser, password: "newpass" }
```

### Changing Application Settings
Edit `roles/server2/defaults/main.yml`:
```yaml
app_domain: "your-domain.com"
app_env:
  PORT: "8080"
  # ... other settings
```

### Updating SSH Key
Replace the public key in `roles/server1/vars/main.yml`:
```yaml
my_pubkey: "ssh-ed25519 YOUR_NEW_PUBLIC_KEY"
```

## 📚 Dependencies

### Server Packages
- curl
- git  
- nginx
- Node.js 20.x
- npm
- pm2

### Ansible Collections
- `ansible.builtin`
- `ansible.posix`

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the deployment
5. Submit a pull request

---

**Note**: Remember to update sensitive information like passwords, SSH keys, and API keys before deploying to production environments.