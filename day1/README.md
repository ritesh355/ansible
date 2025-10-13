## How to install ansible into ubuntu ec2 

### **step1**Update your package index:

```
sudo apt update
```
This ensures you have the latest package lists from Ubuntu repositories

### **step2**Install software-properties-common (needed to manage PPAs):
```
sudo apt install software-properties-common -y
```
This package lets you easily add third-party PPAs for up-to-date software.​

### **step3**Add the Ansible official PPA:
```
sudo add-apt-repository --yes --update ppa:ansible/ansible
```
This step provides access to the latest stable Ansible version.​

### **step4**Install Ansible:
```
sudo apt install ansible -y
```
This installs Ansible and its required dependencies.​


### **step5**Verify the installation:

```
ansible --version
```
You should see the Ansible version information as confirmation.​


