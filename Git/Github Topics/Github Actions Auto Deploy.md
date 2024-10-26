### 1. Generate Deployment SSH Key
```bash
# Generate new SSH key without passphrase
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/github_actions_deploy -N ""

# Display public key to copy
cat ~/.ssh/github_actions_deploy.pub

# Display private key to copy
cat ~/.ssh/github_actions_deploy
```

### 2. Server Setup
```bash
# On your server, add to authorized_keys
nano ~/.ssh/authorized_keys
# Paste the public key
```

### 3. GitHub Repository Setup
- Go to repo → Settings → Secrets and Variables → Actions
- Add secrets:
  - `HOST`: Your server IP
  - `USERNAME`: Your server username
  - `SSH_KEY`: The private key content (entire thing including BEGIN/END lines)

### 4. Create GitHub Action
```bash
# Create directories
mkdir -p .github/workflows

# Create deploy.yml
nano .github/workflows/deploy.yml
```

### 5. Deploy.yml Content
```yaml
name: Deploy
on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          port: 2222    # Adjust if different
          script: |
            cd ~/path/to/your/project
            git pull
            docker-compose pull
            docker-compose build --no-cache
            docker-compose up -d
            docker image prune -f
```

### 6. Verify Setup
- Push deploy.yml to main branch
- Go to Actions tab
- Click "Deploy" workflow
- Click "Run workflow"
- Check logs for success

### Common Issues:
- SSH port not 22? Add `port: YOUR_PORT` in deploy.yml
- Permission issues? Check docker and git permissions
- Timeout? Check firewall/networking
- Key issues? Verify no passphrase and correct format

### Testing:
```bash
# Test SSH connection manually
ssh -i ~/.ssh/github_actions_deploy -p 2222 username@server

# Test deployment script manually on server
cd ~/path/to/your/project
git pull
docker-compose pull
docker-compose build --no-cache
docker-compose up -d
```

### Optional Additions:
```yaml
# Add to deploy.yml for more triggers
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * *'  # Daily deployment
```

Remember:
- Keep SSH key secure
- Use specific image tags in docker-compose
- Consider adding deployment notifications
- Add branch protections for main
- Keep secrets updated