# 🚀 WireGuard VDS Setup Guide

This guide will walk you through setting up a Virtual Dedicated Server (VDS) on Timeweb using Ubuntu 22.04, creating a user, installing Docker, and configuring WireGuard with a web UI.

## 🏢 Hosting Details

**Provider**: [Timeweb](https://timeweb.cloud/)
**Server Location**: Netherlands
**Server Specifications**:

- 2 x 3.3 GHz CPU
- 2 GB RAM
- 40 GB NVMe
  **Operating System**: Ubuntu 22.04

## 👤 Setting Up a New User

1. **Connect to SSH as Root**

Open your terminal (PowerShell, Bash, etc.) and connect via SSH:

```bash
ssh root@<host>
```

If prompted about authenticity, type `yes` and then enter the root password.

2. **Create a New User**

Create a new user with the following command:

```bash
adduser username
```

Follow the prompts to set a password and optional information (you can press ENTER to skip optional fields).

3. **Grant Sudo Privileges**

Add the new user to the `sudo` group:

```bash
usermod -aG sudo username
```

4. **Setup SSH Authorization for the New User**

Copy the SSH keys to the new user's directory:

```bash
cp -r ~/.ssh /home/username
```

Change the ownership of the `.ssh` directory to the new user:

```bash
chown -R username:username /home/username/.ssh
```

5. **Login as the New User**

Now, you can log in as the new user:

```bash
ssh username@server_ip
```

## 🐳 Installing Docker

Follow the [official Docker documentation](https://docs.docker.com/engine/install/ubuntu/) for detailed instructions.

1. **Remove Conflicting Packages**

Run the following command to uninstall any conflicting packages:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

2. **Set Up Docker's `apt` Repository**

Add Docker's official GPG key and repository:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

3. **Install Docker**

Install Docker and its components:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. **Add Docker Group**

Add your user to the Docker group:

```bash
sudo usermod -aG docker $(whoami)
```

5. **Reboot the Server**

Reboot the server to apply changes:

```bash
sudo reboot now
```

6. **Verify Docker Installation**

Run the following command to verify Docker is installed correctly:

```bash
sudo docker run hello-world
```

If the installation is successful, you'll see a message from Docker.

## 🔄 Uninstalling Docker

To remove Docker and its components, use the following commands:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

If you want to remove Docker data as well, run:

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

## 🔒 Setting Up WireGuard in a Docker Container

You can use the `wg-easy` Docker image to set up WireGuard with a web UI.

1. **Create the `.env` File**

Create a `.env` file with the following content:

```env
WG_HOST=<your_server_ip_or_domain>
WG_PASSWORD_HASH=<your_bcrypt_hashed_password>
```

To generate a bcrypt password hash, run:

```bash
docker run ghcr.io/wg-easy/wg-easy wgpw YOUR_PASSWORD

#PASSWORD_HASH='$2b$12$coPqCsPtcFO.Ab99xylBNOW4.Iu7OOA2/ZIboHN6/oyxca3MWo7fW' // literally YOUR_PASSWORD
```

Ensure the password is enclosed in single quotes.

2. **Create `docker-compose.yml`**

Create a `docker-compose.yml` file:

```yaml
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    env_file:
      - .env
    environment:
      - LANG=ru
      - WG_HOST=${WG_HOST}
      - PASSWORD_HASH=${WG_PASSWORD_HASH}
      - PORT=51821
      - WG_PORT=51820
    volumes:
      - ./wg-easy/:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```

3. **Manage Docker Containers**

Use the following Docker Compose commands:

```bash
docker compose ps              # List all services
docker compose up -d           # Build and start containers
docker compose stop            # Stop containers
docker compose rm <id/name>    # Remove container
docker compose up -d --force-recreate --no-deps --build  # Rebuild containers
```

4. **Access the Web UI in your browser**

```
http://<server_ip>:51821/
```

Log in with your password and configure clients.

## 📱 Setting Up WireGuard Client

For instructions on installing WireGuard on various platforms, visit the [official WireGuard website](https://www.wireguard.com/install/). You can also install it on Android via Google Play and add configurations using a QR code or file.
