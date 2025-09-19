# Project 1 – EC2 REST Service: Pounds → Kilograms

This repository contains the source code for **Project 1** of the **CS 554 Cloud Computing** course at UAH. The project involves provisioning an AWS EC2 instance and deploying a small REST web service that converts **pounds (lbs)** to **kilograms (kg)**.

The architectural design for hosting this service is explained [here](Design.md).

## 1. Prerequisites

* **Ubuntu Linux AMI** on AWS EC2
* **Node.js** (v18+) and **npm**
* SSH access using a key pair

## 2. Setup and Deployment

### 2.1 EC2 Instance

* An EC2 instance was provisioned using an Ubuntu AMI.
* A key pair was created for SSH access.
* The Security Group was configured with the following rules:

  * **SSH (TCP 22)**: Allowed only from the owner’s IP address.
  * **HTTP (TCP 80)**: Allowed from anywhere (`0.0.0.0/0`).
  * **HTTPS (TCP 443)**: Allowed from anywhere (`0.0.0.0/0`) (reserved for future SSL/TLS).

**Public IP:** `3.144.168.176`

### 2.2 Service Installation

After SSHing into the instance:

```bash
cd ~/p1
npm install express morgan
```

The application source code was placed in the `~/p1` directory.

### 2.3 Running the Service with systemd

A systemd service runs the Node.js application under the non-privileged `ubuntu` user.

**Unit file:** `/etc/systemd/system/p1.service`

```ini
[Unit]
Description=CS554 Project 1 service

[Service]
ExecStart=/usr/bin/node /home/ubuntu/p1/server.js
WorkingDirectory=/home/ubuntu/p1
Restart=on-failure
User=ubuntu
Group=ubuntu

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl enable p1
sudo systemctl start p1
sudo systemctl status p1 --no-pager
```

**Service output example:**
![Systemd](images/systemd_output.png)

### 2.4 Reverse Proxy (NGINX)

NGINX was configured to forward requests from port 80 to the Node.js application running on port 8080:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## 3. API Test Cases

The REST API endpoint is:

```
http://3.144.168.176/convert?lbs=<value>
```

Responses are in JSON format. Example:

```json
{
    "lbs":91,
    "kg":41.277,
    "formula":"kg = lbs * 0.45359237"
}
```

### 3.1 Happy Path

**Request:**

```bash
curl 'http://3.144.168.176/convert?lbs=0'
```

**Response Screenshot:**
![0](images/0.png)

### 3.2 Typical Case

**Request:**

```bash
curl 'http://3.144.168.176/convert?lbs=150'
```

**Response Screenshot:**
![150](images/150.png)

### 3.3 Edge Case

**Request:**

```bash
curl 'http://3.144.168.176/convert?lbs=0.1'
```

**Response Screenshot:**
![0.1](images/0.1.png)


### 3.4 Error Cases

**Missing Parameter**

```bash
curl 'http://3.144.168.176/convert'
```

![error](images/error_convert.png)

**Negative Value**

```bash
curl 'http://3.144.168.176/convert?lbs=-5'
```

![negative](images/error_negative.png)

**Non-numeric Value**

```bash
curl 'http://3.144.168.176/convert?lbs=NAN'
```

![NAN](images/error_400.png)

