# Nginx Configuration and Tutorial

This repository contains a comprehensive tutorial on Nginx configuration, covering everything from basic installation to advanced features like load balancing, SSL/TLS, caching, and security optimizations.

## Available Languages

- [English](docs/nginx-guide-en.md)
- [Persian (فارسی)](docs/nginx-guide-fa.md)

## What's Included

This tutorial covers:

- Basic Nginx installation and setup
- Service management with systemd
- Configuration file structure
- Location blocks and URL matching
- Redirects and rewrites
- Logging and monitoring
- Caching strategies
- HTTP/2 implementation
- SSL/TLS security best practices
- Reverse proxy and load balancing
- Rate limiting and security hardening

## Repository Structure

```
.
├── README.md (This file)
├── .gitignore
└── docs/
    ├── nginx-guide-en.md (English tutorial)
    ├── nginx-guide-fa.md (Persian tutorial)
    └── config-examples/
        ├── basic-server.conf (Basic configuration example)
        ├── ssl-config.conf (HTTPS/SSL setup)
        └── load-balancing.conf (Load balancing example)
```

## Configuration Examples

The repository includes several configuration examples to help you get started:

1. [Basic Server Configuration](docs/config-examples/basic-server.conf) - A simple web server setup
2. [SSL/HTTPS Configuration](docs/config-examples/ssl-config.conf) - Secure your site with SSL
3. [Load Balancing Configuration](docs/config-examples/load-balancing.conf) - Distribute traffic across multiple servers

## Purpose

The goal of this repository is to provide a structured reference for both beginners and advanced users looking to optimize their Nginx deployments. Each topic is explained with practical examples and configuration snippets. 