# Sanic App Deployment on AWS EC2 with Ansible

## Overview

This project deploys a **Sanic web application** on an **AWS EC2 instance** using **Ansible** for automation. The application is managed as a **systemd service** and runs on **port 8000**.

## Prerequisites

- An AWS EC2 instance running **Amazon Linux**
- **Python 3.9+** installed
- **Ansible** installed on your local machine
- **Sanic** framework installed in a virtual environment
- SSH access to the EC2 instance

## Folder Structure

```
/opt/sanicapp/
|-- app.py
|-- requirements.txt
|-- static/
|-- templates/
|-- env/ (Python Virtual Environment)
|-- sanicapp.service (Systemd service file)
```

## Installation & Setup

### Step 1: SSH into EC2 Instance

```sh
ssh -i your-key.pem ec2-user@your-ec2-ip
```

### Step 2: Install Dependencies

```sh
sudo yum update -y
sudo yum install python3.9 python3.9-venv git -y
```

### Step 3: Clone the Repository & Setup Virtual Environment

```sh
cd /opt/
sudo git clone https://github.com/your-repo/sanicapp.git
cd sanicapp
python3.9 -m venv env
source env/bin/activate
pip install -r requirements.txt
```

### Step 4: Create a Systemd Service

Create :

```ini
[Unit]
Description=Sanic App
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/opt/sanicapp
ExecStart=/opt/sanicapp/env/bin/python /opt/sanicapp/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload systemd and start the service:

```sh
sudo systemctl daemon-reload
sudo systemctl start sanicapp
sudo systemctl enable sanicapp
```

### Step 5: Check Service Logs

```sh
sudo journalctl -u sanicapp -n 50 --no-pager
```

### Step 6: Verify the Application

```sh
ss -tulnp | grep 8000
curl http://localhost:8000
```

## Debugging Issues

- Check logs:
  ```sh
  sudo journalctl -u sanicapp --no-pager | tail -n 50
  ```
- Run app manually:
  ```sh
  cd /opt/sanicapp
  source env/bin/activate
  python app.py
  ```
- Check system resources:
  ```sh
  free -h
  top -o %CPU
  ```

## Deployment with Ansible

Use the `deploy.yml` Ansible playbook:

```yaml
- name: Deploy Sanic App
  hosts: ec2
  become: yes
  tasks:
    - name: Install required system packages
      yum:
        name:
          - python3.9
          - python3.9-venv
          - git
        state: present

    - name: Pull latest code
      git:
        repo: 'https://github.com/your-repo/sanicapp.git'
        dest: /opt/sanicapp
        force: yes

    - name: Set up virtual environment
      command: python3.9 -m venv /opt/sanicapp/env
      args:
        creates: /opt/sanicapp/env

    - name: Install dependencies
      pip:
        requirements: /opt/sanicapp/requirements.txt
        virtualenv: /opt/sanicapp/env

    - name: Copy systemd service file
      copy:
        dest: /etc/systemd/system/sanicapp.service
        content: |
          [Unit]
          Description=Sanic App
          After=network.target
          
          [Service]
          User=ec2-user
          WorkingDirectory=/opt/sanicapp
          ExecStart=/opt/sanicapp/env/bin/python /opt/sanicapp/app.py
          Restart=always
          
          [Install]
          WantedBy=multi-user.target

    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Start and enable Sanic App
      systemd:
        name: sanicapp
        state: restarted
        enabled: yes
```

Run playbook:

```sh
ansible-playbook -i inventory deploy.yml
```

## Conclusion

Your Sanic app should now be running on **AWS EC2** and automatically restart on failures. If issues persist, review logs and adjust configurations accordingly. 🚀
