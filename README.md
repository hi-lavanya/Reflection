# 🛡️ Reflection Guide

This guide explains how to set up a **Reflector** and **Mirror** system using Docker. The reflector forwards traffic securely to a mirror server, creating a protected proxy layer.

---

## 🔧 Prerequisites

* Docker & Docker Compose installed
* Linux-based environment (e.g., Ubuntu)
* Access to the `reflection` and `mirror` source code repositories
* Matching cryptographic and heartbeat keys in both systems (`crypt.js`, `aes.json`)

---

## 🚀 Reflector Setup

The Reflector acts as the public-facing entry point, securely forwarding all incoming traffic to the Mirror while masking the true server location and preserving the integrity of the backend system.


### 1. Clone the repository

```bash
git clone <reflection-docker-repo-url>
cd reflection
```

### 2. Build Docker image

```bash
docker build -t <reflection-image-name> .
```

### 3. Configure `reflection.json`

Update the configuration as needed:

```json
{
  "reflected_host": "REFLECTOR_BIND_IP",
  "reflected_port": 12000,
  "mirror_host": "MIRROR_SERVER_IP",
  "mirror_hostudpv4": "0.0.0.0",
  "mirror_port": 13010,
  "mirror_heartbeat_tcp": 11001,
  "mirror_heartbeat_udp": 11002,
  "mirror_heartbeats_lost_for_exit": 2,
  "mirror_service_host": "::",
  "mirror_service_host_udpv4": "0.0.0.0",
  "mirror_service_host_udpv6": "::",
  "mirror_service_port": 14020,
  "mirror_force_listener": true,
  "mirror_connect_timeout": 20000
}
```

> ⚙️ **Explanation:**
>
> * `reflected_*`: where the reflector listens for incoming traffic
> * `mirror_*`: where the reflector forwards traffic
> * `heartbeat_*`: health-check mechanism between reflector and mirror

### 4. Key Configuration

Ensure the encryption and heartbeat keys in `crypt.js` and `aes.json` match those on the mirror server.

### 5. Optional: Multi-server mode

Use `massreflect.json` if deploying multiple mirrors.

### 6. Run the Docker container

```bash
sudo docker run -d \
  --name reflector \
  --restart unless-stopped \
  --network host \
  <reflection-image-name>
```

---


## ✅ Workflow Overview

1. Start the **mirror** container first.
2. Launch the **reflector**, which connects to the mirror.
3. Route traffic to the reflector’s public endpoint.
4. Reflector securely forwards requests to the mirror.

---

## ❓ FAQ

**Q:** What happens if the mirror is unreachable?
**A:** The reflector will retry connection until `mirror_connect_timeout`. After a set number of missed heartbeats, it shuts down gracefully.

**Q:** Why must the keys match?
**A:** Both systems need matching encryption and heartbeat secrets for secure communication.

**Q:** Can reflector and mirror run on the same host?
**A:** Yes, but only recommended for development/testing.

