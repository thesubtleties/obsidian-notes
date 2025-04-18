
## Installation 📥
```bash
# Linux
sudo apt install curl wget      # Debian/Ubuntu
sudo yum install curl wget      # CentOS/RHEL
sudo pacman -S curl wget        # Arch Linux

# macOS
brew install curl wget          # Using Homebrew

# Windows
choco install curl wget         # Using Chocolatey
winget install curl wget        # Using winget
```

## Basic Usage
```bash
curl [options] URL              # Make requests to URLs
wget [options] URL              # Download files from URLs
```

## Common Options - curl 🔍
```bash
curl -o filename.html URL       # Save output to file
curl -O URL                     # Save with remote filename
curl -L URL                     # Follow redirects
curl -I URL                     # Headers only
curl -X POST URL                # Specify HTTP method
curl -d "data" URL              # Send POST data
curl -H "Header: Value" URL     # Add custom header
curl -u username:password URL   # Basic authentication
```

## Common Options - wget 🔍
```bash
wget URL                        # Download file
wget -O filename.html URL       # Save to specific filename
wget -c URL                     # Resume interrupted download
wget -b URL                     # Download in background
wget -r URL                     # Recursive download
wget --limit-rate=100k URL      # Limit download speed
wget -P /path/to/dir URL        # Save to specific directory
```

## HTTP Methods with curl 🔄
```bash
curl -X GET URL                 # GET request (default)
curl -X POST -d "data" URL      # POST request with data
curl -X PUT -d "data" URL       # PUT request
curl -X DELETE URL              # DELETE request
```

## Common Examples 🚀
```bash
# Download a file with curl
curl -O https://example.com/file.zip

# Download with progress bar
curl -# -O https://example.com/file.zip

# POST JSON data
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com

# Download a website recursively with wget
wget -r -np -k https://example.com/

# Download multiple files with wget
wget -i urls.txt

# Resume a large download
wget -c https://example.com/large-file.iso
```

## Authentication 🔐
```bash
# Basic auth with curl
curl -u username:password https://example.com

# OAuth token with curl
curl -H "Authorization: Bearer token123" https://api.example.com

# Save cookies
curl -c cookies.txt https://example.com
curl -b cookies.txt https://example.com/login
```

## Troubleshooting 🔧
```bash
curl -v URL                     # Verbose output
wget -d URL                     # Debug output
curl --connect-timeout 10 URL   # Set connection timeout
wget --tries=10 URL             # Set number of retries
```

## Further Reading 📚
- Official Documentation:
  - [curl Documentation](https://curl.se/docs/)
  - [wget Documentation](https://www.gnu.org/software/wget/manual/wget.html)
- Man Pages: 
  - `man curl` and `man wget` in your terminal
- Additional Resources:
  - [curl Cookbook](https://catonmat.net/cookbooks/curl)
  - [wget Cheat Sheet](https://gist.github.com/Dammmien/4af98e05f9c51c2da007cc70d62bf562)
  - [HTTP API Testing with curl](https://curl.se/docs/httpscripting.html)

---