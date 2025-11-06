# 🧠 Ansible Setup with Passwordless SSH (Private Key Method)

This guide walks you through a **clean Ansible setup** using a dedicated `ansible` user, passwordless sudo, and private key authentication (no password prompt).

This setup allows your control node to manage multiple EC2 instances without entering passwords every time, making automation smooth and efficient. Perfect for beginners wanting hands-on practice with Ansible and AWS

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

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/eysdy5hk7ubdbtiwhg30.png)
### Add `ansible` to sudoers:

```bash
sudo visudo
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vzc952loa7b7x5ctlzyp.png)


```
ansible ALL=(ALL) NOPASSWD:ALL
```
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n0smk8z4k4tjm12ml7vx.png)
Add this line at the end: After adding press ctrl+o then enter then ctrl+x

💡 This gives passwordless sudo access to the `ansible` user.

---

## 🔐 Step 2 — Configure SSH on (Managed+ controle) Nodes

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

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o53ec3rk8uta27jkoedg.png)

Then restart SSH:

* On **Amazon Linux**:

  ```bash
  sudo systemctl restart sshd
  ```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g9co2glgqphs4gnzi79y.png)
* On **Ubuntu**:

  ```bash
  sudo systemctl restart ssh
  ```

---

## 🧰 Step 4 — Install Ansible (Control Node Only)

On **control node** (Amazon Linux):
```
sudo yum install python3-pip -y
```
```
sudo pip3 install ansible
```

```
ansible --version
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4c06s8kbwj35uty7q70c.png)

Verify:

```bash
ansible --version
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qt291jq8rd5kuxh36k07.png)

## 🔑 Step 5 — Generate SSH Key Pair on Control Node

Switch to `ansible` user on **control node**:

```bash
sudo su - ansible
```

Generate SSH keys:

```bash
ssh-keygen -t rsa -b 2048
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i11fjf1c15bbrcc8lkmp.png)
Press **Enter** for all prompts to accept defaults (no passphrase).

You’ll get:

```
/home/ansible/.ssh/id_rsa      (private key)
 /home/ansible/.ssh/id_rsa.pub  (public key)
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ptuvzggkw8mjj2jyl5r3.png)
---

## 📤 Step 6 — Copy SSH Key to Managed Nodes (Passwordless Setup)

Use this command on the **control node**:

```bash
ssh-copy-id ansible@<managed_node_private_ip>
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9f9crc9bu88hx4uvyw69.png)
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

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y84le98bz4pk0yewg3a5.png)
If it logs in **without asking password**, passwordless SSH is working perfectly.

---

## 🧭 Step 7 — Verify Setup with Ansible Ping

Create an inventory file `/home/ansible/hosts`:

use this command under **/home/ansible** directory

```
sudo vi hosts
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p7bbtzhces4lkskgx6tz.png)

```
[web]
172.31.29.148 

[dev]
172.31.18.225 
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kja926uvecq28sd8w3rj.png)

Now test connection:

```bash
ansible all -i hosts -m ping
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i0p0j54x4gvcv2llopes.png)
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
ansible all -i hosts -m shell -a "hostname"
ansible all -i hosts -m shell -a "uptime"
```

If you see hostnames and uptime output — congratulations 🎉
Your **Ansible setup with private key passwordless access** is ready!

---

## 🧾 Notes

* Private key (`id_rsa`) always stays on the **control node**
* Public key (`id_rsa.pub`) is copied to managed nodes’ `~/.ssh/authorized_keys`
* Never share or upload your private key to any other system...

## ❤️ Follow My DevOps Journey
**Ritesh Singh**  
🌐 [LinkedIn](https://www.linkedin.com/in/ritesh-singh-092b84340/) | 📝 [Hashnode](https://ritesh-devops.hashnode.dev/) | [GITHUB](https://github.com/ritesh355/Devops-journal)
