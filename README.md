# Squid Proxy with TLS Termination via Nginx

Secure Squid proxy server with TLS termination through Nginx. This project uses Docker Compose to deploy two containers: Squid for proxying and Nginx for TLS connection handling.

## 📋 Overview

This project provides a ready-to-use configuration for deploying an HTTP(S) proxy server with basic authentication and TLS encryption. Nginx acts as a TLS terminator, accepting encrypted connections on port 8443 and proxying them to Squid on internal port 3128.

### Architecture

```
Client → Nginx (TLS:8443) → Squid (HTTP:3128) → Internet
```

## 🚀 Features

- **TLS Encryption**: Secure connections using Let's Encrypt certificates
- **Basic Authentication**: Access protection with username and password
- **Containerization**: Easy deployment with Docker Compose
- **Auto-restart**: Containers automatically restart on failure
- **Modern Protocols**: Support for TLS 1.2 and 1.3

## 📦 Requirements

- Docker
- Docker Compose
- Let's Encrypt certificates (located in `/etc/letsencrypt/`)
- `htpasswd` (from `apache2-utils` package) for user creation

## 📁 Project Structure

```
squid-nginx-tls/
├── docker-compose.yaml      # Docker Compose configuration
├── create-user.sh           # Script for creating users
├── nginx/
│   └── stream.conf          # Nginx stream configuration
├── squid/
│   └── squid.conf           # Squid configuration (included)
└── passwd                   # Password file (created by script)
```

## ⚙️ Configuration

### 1. Squid Configuration

The project includes a pre-configured `squid/squid.conf` file with:
- Basic authentication support
- Standard port access rules (80, 443, etc.)
- Secure defaults (denies unsafe ports and connections)
- Caching and refresh patterns

You can customize this configuration if needed. The main settings include:

```
# Authentication
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Squid Proxy Authentication

# Allow authenticated users
http_access allow authenticated_users
http_access deny all

# Listening port
http_port 3128
```

### 2. SSL Certificate Configuration

Update the certificate paths in [nginx/stream.conf](nginx/stream.conf) with your domain:

```conf
ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
```

### 3. Creating Users

Use the [create-user.sh](create-user.sh) script to create users:

```bash
./create-user.sh username
```

The script will:
- Install `htpasswd` if not available
- Create the `passwd` file (on first run)
- Add the user and prompt for a password

## 🔧 Installation and Setup

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd squid-nginx-tls
```

### Step 2: Prepare Configuration

1. Update SSL certificate paths in [nginx/stream.conf](nginx/stream.conf) with your domain
2. (Optional) Customize [squid/squid.conf](squid/squid.conf) if you need specific proxy rules
3. Create at least one user

```bash
./create-user.sh myuser
```

### Step 3: Start Services

```bash
docker-compose up -d
```

Check status:

```bash
docker-compose ps
```

View logs:

```bash
# All services
docker-compose logs -f

# Squid only
docker-compose logs -f squid

# Nginx only
docker-compose logs -f tls
```

## 🔌 Using the Proxy

### Client Configuration

Configure your browser or application to use the proxy:

- **Host**: your-domain.com (or server IP address)
- **Port**: 8443
- **Protocol**: HTTPS
- **Authentication**: Basic (username and password)

### Example with curl

```bash
curl -x https://username:password@your-domain.com:8443 https://ifconfig.me
```

### System Configuration (Linux/macOS)

Export environment variables:

```bash
export https_proxy="https://username:password@your-domain.com:8443"
export http_proxy="https://username:password@your-domain.com:8443"
```

## 🛡️ Security

### Recommendations

1. **Use Strong Passwords**: Choose complex passwords when creating users
2. **Restrict Access**: Configure firewall to limit access to port 8443
3. **Update Certificates**: Set up automatic Let's Encrypt renewal
4. **Monitor**: Regularly check logs for suspicious activity
5. **Squid ACLs**: Configure ACLs in Squid to restrict accessible resources

### Certificate Renewal

For automatic Let's Encrypt certificate renewal and Nginx reload:

```bash
# After certificate renewal
docker-compose restart tls
```

## 🔄 Management

### Stop Services

```bash
docker-compose down
```

### Restart

```bash
docker-compose restart
```

### Update Containers

```bash
docker-compose pull
docker-compose up -d
```

### Add New User

```bash
./create-user.sh newuser
docker-compose restart squid
```

### Remove User

Edit the `passwd` file, delete the user's line, and restart Squid:

```bash
nano passwd
docker-compose restart squid
```

## 📊 Monitoring

### Check Proxy Operation

```bash
# Check port availability
nc -zv proxyIP 8443

# Test through proxy
curl -x https://user:pass@proxyIP:8443 https://ifconfig.me
```


## 🐛 Troubleshooting

### Containers Won't Start

Check logs:
```bash
docker-compose logs
```

### SSL Errors

- Ensure certificate paths are correct
- Check certificate expiration dates
- Verify certificate files are readable

### Authentication Errors

- Check that `passwd` file exists and contains users
- Ensure the file is mounted in the Squid container
- Restart Squid after changing passwords

### Proxy Not Working

1. Check port availability: `docker-compose ps`
2. Verify Squid configuration: `docker exec -it squid-proxy squid -k parse`
3. Check Nginx logs: `docker-compose logs tls`

## 📝 License

This project uses open source software:
- Squid Cache - GPL
- Nginx - BSD-like license

## 🤝 Contributing

To contribute:
1. Fork the repository
2. Create a branch for your changes
3. Submit a pull request

## 📧 Contact

For questions or issues, please create an Issue in the project repository.
