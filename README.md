# Caddy with Docker and Cloudflare DNS

This project provides a setup for running a web application with Caddy in Docker containers, using Cloudflare's DNS-01 challenge for automatic HTTPS. This method allows you to obtain SSL certificates, including wildcard certificates, without exposing your server to the public internet on port 80.

## Prerequisites

*   Docker
*   Docker Compose
*   A domain name managed by Cloudflare.
*   A Cloudflare API Token.
*   **For Local Testing:** Add an A DNS record in your Cloudflare DNS settings for your `APP_DOMAIN` (e.g., `test1.dev.tiatech.dev`) with the value `127.0.0.1`. This allows your local machine to resolve the domain locally.

## How to Use

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/farzad1120/local-ssl-caddy.git
    cd local-ssl-caddy
    ```

2.  **Create a Cloudflare API Token:**
    Navigate to your Cloudflare dashboard -> My Profile -> API Tokens. Click 'Create Token' and select 'Create Custom Token'. Grant it 'Zone:DNS:Edit' permissions for the specific zone(s) where your domain is managed.

3.  **Configure your environment:**
    Create a `.env` file and add the following variables:
    ```
    APP_DOMAIN=your-domain.com
    SSL_EMAIL=your-email@example.com
    CF_API_TOKEN=your-cloudflare-api-token
    ```
    For a wildcard certificate, you would set `APP_DOMAIN=*.your-domain.com`.

4.  **Start your application:**
    ```bash
    docker compose up -d
    ```

Your application should now be running and accessible at `https://your-domain.com`. Caddy will automatically provision and renew SSL certificates for your domain using Cloudflare's DNS.

## Configuration

*   **`docker-compose.yml`**: Defines the `caddy` and `app` services. It uses a custom Caddy image (`caddybuilds/caddy-cloudflare:latest`) that includes the Cloudflare DNS provider.
*   **`.env`**: Contains your domain, email address, and Cloudflare API token.
*   **`Caddyfile`**: The configuration file for Caddy. It's configured to use the Cloudflare DNS-01 challenge to obtain SSL certificates.

## Directory Structure

```
.
├── .env                    # Environment variables
├── Caddyfile               # Caddy configuration
├── docker-compose.yml      # Docker Compose file
└── app
    ├── Dockerfile          # Dockerfile for the app
    ├── index.js            # Simple Node.js app
    └── package.json
```

## How it Works

Caddy is configured to use the DNS-01 challenge with Cloudflare. Here's how it works:

1.  When Caddy needs to get a certificate for your domain, it tells Let's Encrypt that it wants to use the DNS-01 challenge.
2.  Let's Encrypt gives Caddy a unique token.
3.  Caddy uses your Cloudflare API token to create a new TXT DNS record for your domain with the value of the token.
4.  Once the DNS record has propagated, Let's Encrypt's servers check for the existence of this record.
5.  If the record exists and has the correct value, Let's Encrypt issues the SSL certificate to Caddy.
6.  Caddy then removes the TXT record from your Cloudflare DNS.

This process allows Caddy to prove ownership of the domain without needing to be accessible from the public internet, which is more secure and flexible.
