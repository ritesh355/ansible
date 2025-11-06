# 🧠 Ansible Setup with Passwordless SSH (Private Key Method)

This guide walks you through a **clean Ansible setup** using a dedicated `ansible` user, passwordless sudo, and private key authentication (no password prompt).

---

## 🚀 Architecture Overview

| Role              | OS                    | Description                          |
| ----------------- | --------------------- | ------------------------------------ |
| **Control Node**  | Amazon Linux          | Runs Ansible and manages other nodes |
| **Managed Nodes** | Amazon Linux / Ubuntu | Machines managed by Ansible          |

---

## 🧩 Step 1 — Create Ansible User on All Nodes

### On **each** node (Control + Managed):

```bash
sudo adduser ansible
sudo passwd ansible
```

### Add `ansible` to sudoers:

```bash
sudo visudo
```

Add this line at the end:

```
ansible ALL=(ALL) NOPASSWD:ALL
```

💡 This gives passwordless sudo access to the `ansible` user.

---

## 🔐 Step 2 — Configure SSH on Managed Nodes

Edit `/etc/ssh/sshd_config` on **each managed node**:

```bash
sudo vi /etc/ssh/sshd_config
```

Uncomment or add the following lines:

```
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```

Then restart SSH:

* On **Amazon Linux**:

  ```bash
  sudo systemctl restart sshd
  ```
* On **Ubuntu**:

  ```bash
  sudo systemctl restart ssh
  ```

---

## 🔑 Step 3 — Generate SSH Key Pair on Control Node

Switch to `ansible` user on **control node**:

```bash
sudo su - ansible
```

Generate SSH keys:

```bash
ssh-keygen -t rsa -b 2048
```

Press **Enter** for all prompts to accept defaults (no passphrase).

You’ll get:

```
/home/ansible/.ssh/id_rsa      (private key)
 /home/ansible/.ssh/id_rsa.pub  (public key)
```

---

## 📤 Step 4 — Copy SSH Key to Managed Nodes (Passwordless Setup)

Use this command on the **control node**:

```bash
ssh-copy-id ansible@<managed_node_private_ip>
```

You’ll enter the password of the `ansible` user **only once**.

Repeat for each managed node.

Example:

```bash
ssh-copy-id ansible@172.31.29.148
ssh-copy-id ansible@172.31.18.225
```

✅ Now test:

```bash
ssh ansible@172.31.29.148
```

If it logs in **without asking password**, passwordless SSH is working perfectly.

---

## 🧭 Step 5 — Verify Setup with Ansible Ping

Create an inventory file `/home/ansible/inventory.ini`:

```ini
[web]
172.31.29.148 ansible_user=ansible ansible_ssh_private_key_file=/home/ansible/.ssh/id_rsa

[db]
172.31.18.225 ansible_user=ansible ansible_ssh_private_key_file=/home/ansible/.ssh/id_rsa
```

Now test connection:

```bash
ansible all -i inventory.ini -m ping
```

Expected output:

```bash
172.31.29.148 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
172.31.18.225 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 🧰 Step 6 — Install Ansible (Control Node Only)

On **control node** (Amazon Linux):

```bash
sudo yum install python3-pip -y
pip3 install ansible
```

Verify:

```bash
ansible --version
```

---

## ✅ Verification Checklist

| Step                                             | Check |
| ------------------------------------------------ | ----- |
| `ansible` user exists on all nodes               | ✅     |
| Passwordless sudo enabled                        | ✅     |
| SSH passwordless login (private key) works       | ✅     |
| `/etc/ssh/sshd_config` updated and SSH restarted | ✅     |
| Ansible ping successful                          | ✅     |

---

## 🧩 Bonus Tip — Test with Ad Hoc Command

```bash
ansible all -i inventory.ini -m shell -a "hostname"
ansible all -i inventory.ini -m shell -a "uptime"
```

If you see hostnames and uptime output — congratulations 🎉
Your **Ansible setup with private key passwordless access** is ready!

---

## 🧾 Notes

* Private key (`id_rsa`) always stays on the **control node**
* Public key (`id_rsa.pub`) is copied to managed nodes’ `~/.ssh/authorized_keys`
* Never share or upload your private key to any other system

