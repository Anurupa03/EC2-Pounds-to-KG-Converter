# Design Document: Weight Conversion API

## Introduction

This document outlines the architecture and technology stack for the **Weight Conversion API**. The service is hosted on AWS and exposes a REST API that allows clients to convert weights via a simple HTTP request.

## Architecture Overview

1. A client (e.g., `curl` or a web browser) makes an HTTP request to a public-facing **EC2 instance**.
2. The EC2 instance hosts a **Node.js/Express REST API** that performs the weight conversion.
3. The API returns a **JSON response** containing the converted weight.
4. **NGINX** acts as a reverse proxy, handling public traffic on ports 80 (HTTP) and 443 (HTTPS) and forwarding requests to the application on port 8080.

**High-Level Architecture Diagram:**

```text
+---------+         +-----------------------+         +--------------------+
| Client  |  --->   | EC2 Instance          |  --->   | JSON Response      |
| (curl,  |         |  - NGINX (443/80)     |         | { "kg": 68.04 }    |
| browser)|         |  - Node.js.           |         +--------------------+
+---------+         +-----------------------+
```

## Technology Stack

* **Cloud Provider**: AWS
* **Compute Resource**: EC2 (t2.micro, AWS Free Tier)
* **Operating System**: Ubuntu Linux
* **Application Framework**: Node.js
* **Process Management**: systemd
* **Reverse Proxy**: NGINX
* **SSL Certificates**: Managed via **Certbot**


## Reliability & Service Management

* **systemd Service**:

  * Ensures the application starts on boot.
  * Automatically restarts the service if it crashes.
  * Logs output to the systemd journal for monitoring and debugging.

* **Non-root Execution**:

  * The application runs under the `ubuntu` user instead of `root`.
  * Minimizes security risks in case of compromise.


## Security Configuration

### Security Group Rules

* **SSH (TCP 22)**: Allowed only from the owner’s IP address.
* **HTTP (TCP 80)**: Allowed from anywhere (`0.0.0.0/0`) for public access.
* **HTTPS (TCP 443)**: Allowed from anywhere (`0.0.0.0/0`) for public access.

### Additional Security Practices

* Application runs on a non-privileged port (8080).
* NGINX handles incoming traffic and forwards it internally to Node.js.
* Configured SSL certificates using **Certbot** and **NGINX**, ensuring all client-server communication is encrypted.
