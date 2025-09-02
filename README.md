# Ansible Deploy

An Ansible automation project for deploying a Node.js quiz application with user management across multiple servers using GitHub Actions and Secrets.

## 📋 Overview

This project automates the deployment of:
- **Server1 (homework)**: User management server with SSH key deployment
- **Server2 (github)**: Node.js application server with Nginx reverse proxy

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
- Creates development users dynamically from GitHub secrets
- Deploys SSH public keys for passwordless authentication
- Configures bash shell and home directories
- Supports flexible user configuration via JSON

### Server2 (Application Server)
- Deploys Node.js quiz application from GitHub
- Sets up PM2 process management
- Configures Nginx reverse proxy
- Manages environment variables securely
- Automatic dependency installation

## 📁 Project Structure

```
ansible_Deploy/
├── README.md
├── inventory.ini           # Server inventory
├── site.yml               # Main playbook
└── roles/
    ├── homework/
    │   ├── tasks/main.yml  # User management tasks
    │   ├── vars/main.yml   # User definitions & SSH key
    │   └── handlers/main.yml # SSH service handlers
    └── github/
        ├── defaults/main.yml    # App configuration
        ├── tasks/main.yml       # App deployment tasks
        ├── handlers/main.yml    # Service handlers
        └── templates/
            ├── env.j2           # Environment template
            └── nginx.conf.j2    # Nginx config template
```

## 🔐 GitHub Secrets Configuration

### Required Secrets

Configure the following secrets in your GitHub repository (`Settings` → `Secrets and variables` → `Actions`):

#### Application Configuration Secrets
| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `ADMIN_PASSWORD` | Admin panel password | `your_secure_admin_password` |
| `SESSION_SECRET` | Express session secret | `your_super_secret_session_key` |
| `OPENROUTER_API_KEY` | AI API key for quiz generation | `sk-or-v1-xxxxxxxxxxxxx` |

#### User Management Secrets
| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `USERS_JSON` | JSON array of users to create | `[{"name": "dev1"}, {"name": "dev2"}, {"name": "dev3"}]` |
| `SSH_PUBLIC_KEY` | SSH public key for user authentication | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5...` |

### Setting Up Secrets

1. **Navigate to your repository** on GitHub
2. **Go to Settings** → **Secrets and variables** → **Actions**
3. **Click "New repository secret"**
4. **Add each secret** with the exact names listed above

#### Example USERS_JSON Format:
```json
[
  {"name": "dev1"},
  {"name": "dev2"},
  {"name": "dev3"},
  {"name": "staging"},
  {"name": "production"}
]
```

#### Getting Your SSH Public Key:
```bash
# Generate new SSH key (if needed)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Display your public key
cat ~/.ssh/id_ed25519.pub
```

## ⚙️ Configuration

### Inventory Setup
Edit `inventory.ini` to match your server details:
```ini
[homework]
192.168.11.86 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa

[github]
192.168.11.87 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Application Configuration
Key settings in `roles/github/defaults/main.yml`:
- **Repository**: `https://github.com/aacc1on/quiz_MVC.git`
- **Domain**: `quizzzzz.camdvr.org`
- **Port**: `3000`
- **Admin Credentials**: Retrieved from GitHub secrets

## 🔧 Prerequisites

### Control Machine
- Ansible installed
- SSH access to target servers
- Private key configured (`~/.ssh/id_rsa`)

### Target Servers
- Ubuntu/Debian-based systems
- Root SSH access
- Internet connectivity for package installation

### GitHub Repository
- All required secrets configured
- GitHub Actions enabled (if using CI/CD)

## 🚀 Deployment

### Local Deployment with Secrets
```bash
# Export secrets as environment variables
export ADMIN_PASSWORD="your_admin_password"
export SESSION_SECRET="your_session_secret"
export OPENROUTER_API_KEY="your_api_key"
export USERS_JSON='[{"name": "dev1"}, {"name": "dev2"}]'
export SSH_PUBLIC_KEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5..."

# Test connectivity
ansible all -i inventory.ini -m ping

# Run the full deployment
ansible-playbook -i inventory.ini site.yml
```

### GitHub Actions Deployment
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy with Ansible
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Ansible
        run: |
          pip install ansible
          
      - name: Deploy to servers
        env:
          ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
          SESSION_SECRET: ${{ secrets.SESSION_SECRET }}
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
          USERS_JSON: ${{ secrets.USERS_JSON }}
          SSH_PUBLIC_KEY: ${{ secrets.SSH_PUBLIC_KEY }}
        run: |
          ansible-playbook -i inventory.ini site.yml
```

### Individual Server Deployment
```bash
# Deploy only user management (Server1)
ansible-playbook -i inventory.ini site.yml --limit homework

# Deploy only application (Server2)
ansible-playbook -i inventory.ini site.yml --limit github
```

## 🔐 Security Best Practices

### Secret Management
- **Never commit secrets** to version control
- **Use different secrets** for different environments
- **Rotate secrets regularly**, especially API keys
- **Use strong passwords** with mixed characters
- **Limit secret access** to necessary team members only

### User Management (Server1)
- Users are created dynamically from the `USERS_JSON` secret
- SSH public key authentication is enabled for all users
- Users are added to the root group for administrative access
- Bash shell is configured for all users

### Application Security (Server2)
- Environment variables sourced from GitHub secrets
- Session management with configurable secrets
- Nginx proxy for additional security layer
- Process isolation with PM2

## 🌐 Application Access

After deployment, the quiz application will be available at:
- **URL**: `http://quizzzzz.camdvr.org`
- **Admin Panel**: Login with credentials from GitHub secrets
- **Port**: Application runs on port 3000, proxied through Nginx

## 📝 Environment Variables

The application uses these environment variables (sourced from GitHub secrets):
```bash
PORT=3000
NODE_ENV=development
ADMIN_USERNAME=admin                    # Default value
ADMIN_PASSWORD=${ADMIN_PASSWORD}        # From GitHub secret
SESSION_SECRET=${SESSION_SECRET}        # From GitHub secret
OPENROUTER_API_KEY=${OPENROUTER_API_KEY} # From GitHub secret
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

**Missing Environment Variables**
```bash
# Check if secrets are properly set
echo $ADMIN_PASSWORD
echo $SESSION_SECRET

# Verify users JSON format
echo $USERS_JSON | python -m json.tool
```

**SSH Connection Failed**
```bash
# Test SSH connectivity
ssh -i ~/.ssh/id_rsa root@192.168.11.86

# Verify SSH public key format
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

**Application Not Starting**
```bash
# Check PM2 status
pm2 list
pm2 logs myapp

# Check environment file
cat /var/www/myapp/.env
```

**Domain Not Resolving**
- Ensure DNS is configured for `quizzzzz.camdvr.org`
- Check Nginx configuration: `sudo nginx -t`

## 🔧 Customization

### Adding More Users
Update the `USERS_JSON` secret with additional users:
```json
[
  {"name": "dev1"},
  {"name": "dev2"},
  {"name": "dev3"},
  {"name": "newuser"},
  {"name": "staging"}
]
```

### Changing Application Settings
Edit `roles/github/defaults/main.yml` or override with additional secrets:
```yaml
app_domain: "your-domain.com"
app_env:
  PORT: "8080"
  # ... other settings
```

### Multiple Environment Support
Create different secret sets for different environments:
- `PROD_ADMIN_PASSWORD`, `STAGING_ADMIN_PASSWORD`
- `PROD_SESSION_SECRET`, `STAGING_SESSION_SECRET`
- etc.

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

## 🚨 Security Warnings

1. **Never expose secrets** in logs or console output
2. **Use HTTPS** in production environments
3. **Regularly update** dependencies and system packages
4. **Monitor access logs** for suspicious activity
5. **Backup configurations** before making changes
