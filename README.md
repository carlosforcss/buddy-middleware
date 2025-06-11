# Buddy Middleware

A Docker-based nginx proxy middleware that sits between your frontend and backend applications, providing SSL termination, load balancing, and routing.

## Features

- **SSL/TLS Termination**: Supports Cloudflare certificates
- **Reverse Proxy**: Routes `/api` requests to backend, everything else to frontend
- **WebSocket Support**: Full WebSocket proxy support for real-time applications
- **Static Asset Optimization**: Caching and compression for JS/CSS/images
- **Security Headers**: Enhanced security with proper HTTP headers
- **Docker Compose**: Easy setup and deployment

## Architecture

```
Internet → Nginx Proxy (SSL) → Frontend (Port 3000) / Backend (Port 8000)
```

- **Frontend**: Served on all routes except `/api` and `/ws`
- **Backend**: Served on `/api/*` and `/ws/*` routes
- **SSL**: Terminated at nginx level using Cloudflare certificates

## Quick Start

### 1. SSL Certificates Setup

Create the SSL directory and add your Cloudflare certificates:

```bash
mkdir ssl
# Copy your Cloudflare certificates to the ssl directory
cp /path/to/your/cloudflare.key ssl/
cp /path/to/your/cloudflare.pem ssl/
```

### 2. Update Docker Compose

Edit `docker-compose.yml` to configure your actual frontend and backend services:

```yaml
frontend:
  # Replace with your frontend configuration
  build: ./frontend  # or use your frontend image
  environment:
    - NODE_ENV=production
  # Add your frontend volumes and commands

backend:
  # Replace with your backend configuration  
  build: ./backend   # or use your backend image
  environment:
    - NODE_ENV=production
  # Add your backend volumes and commands
```

### 3. Start Services

```bash
docker-compose up -d
```

### 4. Access Your Application

- **HTTPS**: https://localhost (redirects from HTTP automatically)
- **HTTP**: http://localhost (redirects to HTTPS)

## Configuration

### Nginx Configuration

The nginx configuration includes:

- **SSL Settings**: TLS 1.2/1.3 with secure ciphers
- **Security Headers**: HSTS, XSS protection, content type sniffing protection
- **Gzip Compression**: For text-based assets
- **WebSocket Support**: Full duplex communication support
- **Static Asset Caching**: 1-year cache for static resources

### Routing Rules

| Route Pattern | Destination | Description |
|---------------|-------------|-------------|
| `/api/*` | Backend | API endpoints |
| `/ws/*` | Backend | WebSocket connections |
| `*.js, *.css, *.png, etc.` | Frontend | Static assets |
| `/*` | Frontend | All other routes (SPA support) |

### Environment Variables

You can customize the configuration by modifying:

- `nginx.conf`: Nginx server configuration
- `docker-compose.yml`: Service definitions and ports
- SSL certificate paths in the nginx configuration

## SSL Certificate Management

### Cloudflare Certificates

1. Generate certificates in Cloudflare dashboard
2. Download `cloudflare.pem` (certificate) and `cloudflare.key` (private key)
3. Place them in the `ssl/` directory
4. Ensure proper file permissions: `chmod 600 ssl/*`

### Custom Certificates

To use different SSL certificates, update the paths in `nginx.conf`:

```nginx
ssl_certificate /etc/nginx/ssl/your-cert.pem;
ssl_certificate_key /etc/nginx/ssl/your-key.key;
```

## Development

### Local Development

For local development without SSL:

1. Comment out the SSL redirect in `nginx.conf`
2. Update the HTTPS server block to listen on port 80
3. Remove SSL certificate directives

### Debugging

View nginx logs:

```bash
docker-compose logs nginx
```

Test configuration:

```bash
docker-compose exec nginx nginx -t
```

Reload configuration:

```bash
docker-compose exec nginx nginx -s reload
```

## Production Deployment

### Security Considerations

- Ensure SSL certificates are properly secured
- Update security headers as needed for your domain
- Configure proper firewall rules
- Monitor logs for security issues
- Keep nginx and Docker images updated

### Performance Tuning

- Adjust worker processes in `nginx.conf`
- Configure appropriate timeout values
- Optimize gzip compression settings
- Set up proper caching strategies

## Troubleshooting

### Common Issues

1. **SSL Certificate Errors**: Ensure certificates are in the correct format and path
2. **WebSocket Connection Issues**: Check upstream server configuration
3. **Static Assets Not Loading**: Verify frontend service is running on port 3000
4. **API Routes Not Working**: Verify backend service is running on port 8000

### Health Check

Access the health endpoint:

```bash
curl -k https://localhost/health
```

Should return: `healthy`

## License

This project is open source. Feel free to modify and use according to your needs. 