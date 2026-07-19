# Ansible Installation Guide for Ubuntu 24.04 LTS (Noble Numbat)

This guide provides step-by-step instructions to install Ansible on **Ubuntu 24.04 LTS**. It covers installation via the official PPA (apt) and Python Virtual Environments.

---

## 📦 Method 1: System-wide Installation via PPA (Recommended)
This is the simplest and most integrated method to install Ansible system-wide on Ubuntu 24.04 LTS.

### 1. Update system package index
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install pre-requisites
```bash
sudo apt install -y software-properties-common
```

### 3. Add official Ansible PPA
The official PPA provides the latest stable releases of Ansible.
```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

### 4. Install Ansible
```bash
sudo apt install -y ansible
```

---

## 🐍 Method 2: Isolated Installation via Python Virtual Environment (venv)
Use this method if you need to test specific Ansible versions or want to avoid modifying system packages.

### 1. Install Python3-venv
```bash
sudo apt update
sudo apt install -y python3-pip python3-venv python3-dev build-essential
```

### 2. Create directory and virtual environment
```bash
mkdir -p ~/ansible-workspace
cd ~/ansible-workspace
python3 -m venv venv
```

### 3. Activation
```bash
source venv/bin/activate
```
*(Your terminal prompt will now start with `(venv)`)*

### 4. Install Ansible via pip
Within the active virtual environment, pip is permitted to install packages.
```bash
pip install --upgrade pip
pip install ansible
```

### 5. Exit Virtual Environment
```bash
deactivate
```

---

## 🔍 Verification
To verify the installation:
```bash
ansible --version
```
This should output the installed Ansible version (e.g., `ansible [core 2.16.x]` or higher) using **Python 3.12** as its underlying interpreter.
