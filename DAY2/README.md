# Setting Up Ansible with One EC2 Ubuntu Control Node and Two Managed Nodes

This guide outlines the steps to configure one EC2 Ubuntu instance as an Ansible control node and two EC2 Ubuntu instances as managed nodes.

## Prerequisites
- Three EC2 instances running Ubuntu (e.g., Ubuntu 20.04 or later).
- SSH key pair for accessing all instances.
- Security groups allowing SSH (port 22) between the control node and managed nodes.
- Basic knowledge of SSH and Ubuntu terminal commands.

## Step 1: Set Up the Control Node

1. **Connect to the Control Node**
   - SSH into the control node EC2 instance:
     ```bash
     ssh -i <your-key.pem> ubuntu@<control-node-public-ip>
     ```

2. **Update the System**
   - Ensure the system is up-to-date:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

3. **Install Ansible**
   - Install Ansible on the control node:
     ```bash
     sudo apt install ansible -y
     ```
   - Verify the installation:
     ```bash
     ansible --version
     ```

4. **Generate SSH Key for Ansible Communication**
   - Create an SSH key pair on the control node:
     ```bash
     ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key -N ""
     ```
   - This creates a private key (`~/.ssh/ansible_key`) and a public key (`~/.ssh/ansible_key.pub`).

5. **Copy SSH Public Key to Managed Nodes**
   - Copy the public key to both managed nodes:
     ```bash
     ssh-copy-id -i ~/.ssh/ansible_key.pub ubuntu@<managed-node-1-public-ip>
     ssh-copy-id -i ~/.ssh/ansible_key.pub ubuntu@<managed-node-2-public-ip>
     ```
   - Alternatively, manually append the public key to `~/.ssh/authorized_keys` on each managed node.

## Step 2: Set Up the Managed Nodes

1. **Connect to Each Managed Node**
   - SSH into each managed node:
     ```bash
     ssh -i <your-key.pem> ubuntu@<managed-node-public-ip>
     ```

2. **Update the System**
   - Update each managed node:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

3. **Install Python (Required for Ansible)**
   - Ensure Python is installed, as Ansible requires it on managed nodes:
     ```bash
     sudo apt install python3 -y
     ```

4. **Verify SSH Access from Control Node**
   - From the control node, test SSH access to each managed node using the Ansible key:
     ```bash
     ssh -i ~/.ssh/ansible_key ubuntu@<managed-node-public-ip>
     ```
   - Ensure passwordless SSH works for both managed nodes.

## Step 3: Configure Ansible on the Control Node

1. **Edit the Ansible Hosts File**
   - Create or edit the Ansible inventory file (`/etc/ansible/hosts`):
     ```bash
     sudo nano /etc/ansible/hosts
     ```
   - Add the managed nodes under a group (e.g., `managed_nodes`):
     ```ini
     [managed_nodes]
     managed_node_1 ansible_host=<managed-node-1-public-ip> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible_key
     managed_node_2 ansible_host=<managed-node-2-public-ip> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible_key
     ```
   - Save and exit.

2. **Test Ansible Connectivity**
   - Verify that the control node can communicate with the managed nodes:
     ```bash
     ansible all -m ping
     ```
   - Expected output for each node:
     ```json
     managed_node_1 | SUCCESS => {
         "changed": false,
         "ping": "pong"
     }
     managed_node_2 | SUCCESS => {
         "changed": false,
         "ping": "pong"
     }
     ```

3. **Configure Ansible SSH Settings (Optional)**
   - To streamline SSH settings, edit the Ansible configuration file (`/etc/ansible/ansible.cfg`):
     ```bash
     sudo nano /etc/ansible/ansible.cfg
     ```
   - Add or modify the following under `[defaults]`:
     ```ini
     [defaults]
     inventory = /etc/ansible/hosts
     host_key_checking = False
     ```
   - This disables SSH host key checking for convenience (not recommended for production).

## Step 4: Test Ansible with a Sample Playbook

1. **Create a Playbook**
   - On the control node, create a simple playbook to test functionality:
     ```bash
     nano test_playbook.yml
     ```
   - Add the following content:
     ```yaml
     ---
     - name: Test Ansible Playbook
       hosts: managed_nodes
       tasks:
         - name: Ensure a test file exists
           file:
             path: /tmp/ansible_test.txt
             state: touch
             mode: '0644'
     ```
   - Save and exit.

2. **Run the Playbook**
   - Execute the playbook:
     ```bash
     ansible-playbook test_playbook.yml
     ```
   - Verify the file was created on each managed node:
     ```bash
     ssh -i ~/.ssh/ansible_key ubuntu@<managed-node-public-ip> ls /tmp
     ```
   - Look for `ansible_test.txt`.

## Step 5: Best Practices and Security
- **Secure SSH Keys**: Store the Ansible SSH private key securely and restrict access (`chmod 600 ~/.ssh/ansible_key`).
- **Use Ansible Vault**: For sensitive data (e.g., passwords), use `ansible-vault` to encrypt variables.
- **Limit Access**: Configure EC2 security groups to restrict SSH access to the control node’s IP.
- **Backup Inventory**: Keep a backup of `/etc/ansible/hosts` and playbooks.
- **Use Roles**: For complex tasks, organize playbooks into roles for reusability.

## Troubleshooting
- **Ping Fails**: Ensure SSH keys are correctly copied and security groups allow port 22.
- **Permission Denied**: Verify the `ansible_user` and `ansible_ssh_private_key_file` in the inventory file.
- **Module Errors**: Confirm Python is installed on managed nodes.

You’re now ready to manage your EC2 instances with Ansible! Let me know if you need help with specific playbooks or advanced configurations. 🚀
