# Traefik Reverse Proxy (Docker + Cloudflare DNS Challenge)

## Overview

This repository provides a production-ready reverse proxy stack built with Traefik v3 and Docker.

The purpose of this service is to:

- Terminate HTTPS on port 443
- Automatically issue and renew TLS certificates using Let's Encrypt
- Use Cloudflare DNS challenge for certificate validation
- Route traffic to multiple internal Docker services via hostname
- Expose only a single public port (443)
- Keep backend services private and isolated

This configuration is designed for home lab and small self-hosted environments where simplicity, clarity, and reproducibility are priorities.

---

## Architecture

Internet  
→ Router (Port Forward 443)  
→ Docker Host  
→ Traefik (TLS termination & routing)  
→ Internal Docker Services  

All backend services remain internal and are accessed only through Traefik.

---

## Prerequisites

Before deployment, ensure the following:

- Docker Engine installed (20.10+ recommended)
- Docker Compose plugin installed
- A Docker network named `reverse-proxy`
- Router port forwarding configured (443 → Docker host)
- Firewall allowing inbound 443/tcp
- A Cloudflare DNS zone for your domain
- A Cloudflare API token with:
  - Zone:DNS:Edit
  - Zone:Zone:Read

---

## Step 1: Create Required Docker Network

If not already created:

```bash
docker network create reverse-proxy

Step 2: Create Environment File

Create a .env file in the same directory as docker-compose.yml:

CF_DNS_API_TOKEN=your_cloudflare_api_token
ACME_EMAIL=you@example.com

Do not commit this file to public repositories.

Step 3: Deploy Traefik

Start the stack:

docker compose up -d

Verify container status:

docker ps

Check logs:

docker logs reverse-proxy-traefik

You should see:

Docker provider started

ACME registration

Certificate renewal checks

Exposing Services via Traefik

To expose a Docker service:

Attach it to the reverse-proxy network.

Add Traefik labels.

Do not expose service ports via ports:.

Example:

services:
  myapp:
    image: myapp:latest
    networks:
      - reverse-proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.example.com`)"
      - "traefik.http.routers.myapp.entrypoints=websecure"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"

Important notes:

The internal port must match the container's listening port.

Do not expose backend ports to the host.

Only Traefik should expose port 443 publicly.

Updating Traefik

To update the image:

docker compose pull
docker compose up -d

Certificates and configuration are stored in a named Docker volume and persist across redeployments.

Security Model

Only port 443 is publicly exposed.

Backend services are internal-only.

Certificates are automatically renewed.

Docker socket is mounted read-only.

DNS-based ACME challenge avoids exposing port 80.

Intended Use

This setup is suitable for:

Home lab environments

Small self-hosted deployments

Multi-service Docker hosts

Environments behind ISP routers and NAT

Further hardening can be added as needed (authentication, rate limiting, WAF, etc.).