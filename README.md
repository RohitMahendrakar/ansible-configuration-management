# Ansible Configuration Management

This project demonstrates how to set up passwordless SSH authentication and use Ansible for configuring multiple target servers, such as CentOS or Ubuntu servers, in a simple and efficient way.

## Prerequisites

Before starting, ensure that you have the following:
- At least two EC2 instances: one as the **Ansible Server** and one as the **Target Server**.
- **SSH Keys** for passwordless authentication between the Ansible server and the target server.
- **Ansible** installed on the Ansible server.
- **Inventory File** to manage target server IP addresses.

## Step 1: SSH Key Generation for Passwordless Authentication

1. **On Ansible Server**:
   - Generate an SSH key pair using the command:
     ```bash
     ssh-keygen -t rsa
     ```
     This will create two files: `key` (private key) and `key.pub` (public key). These files will be in the `~/.ssh/` directory.

2. **On the Target Server**:
   - Run `ssh-keygen -t rsa` to generate a key pair on the target server.
   - Copy the public key from the Ansible server (located in `~/.ssh/key.pub`) to the target server’s `~/.ssh/authorized_keys` file:
     ```bash
     cat ~/.ssh/key.pub >> ~/.ssh/authorized_keys
     ```

3. **Verify Passwordless Authentication**:
   - From the Ansible server, try to SSH into the target server using its private IP:
     ```bash
     ssh <target-server-private-ip>
     ```
   - If successful, you've set up passwordless authentication.

## Step 2: Setting Up Ansible

1. **Create an Inventory File**:
   - The inventory file stores IP addresses of target servers. Example:
     ```ini
     [webservers]
     192.168.1.100
     192.168.1.101
     ```

2. **Running Ansible Commands**:
   - To run a shell command on all servers in the inventory:
     ```bash
     ansible -i inventory all -m shell -a "touch /tmp/ansible_testfile"
     ```
   - To target a specific server:
     ```bash
     ansible -i inventory <target-server-ip> -m shell -a "touch /tmp/ansible_testfile"
     ```

3. **Using Host Groups**:
   - You can group servers in your inventory. For example, to run a command on all web servers, use:
     ```bash
     ansible -i inventory webservers -m shell -a "touch /tmp/ansible_testfile"
     ```

## Step 3: Writing and Running Ansible Playbooks

Ansible playbooks are written in YAML format. Below is an example of a playbook to install Nginx on all web servers:

1. **Create the Playbook**:
   - Run the following command to create the playbook:
     ```bash
     vim first-playbook.yml
     ```
   - Add the following YAML content:
     ```yaml
     ---
     - name: Install Nginx on all servers
       hosts: all
       become: yes  # Use 'yes' to enable privilege escalation (sudo)

       tasks:
         - name: Install nginx
           shell: apt-get update && apt-get install -y nginx
           # apt-get update ensures the package list is updated
           # apt-get install -y installs nginx without asking for confirmation
     ```

2. **Run the Playbook**:
   - Use the following command to run the playbook:
     ```bash
     ansible-playbook -i inventory first-playbook.yml
     ```
   - To enable debugging and verbosity, use `-vvv`:
     ```bash
     ansible-playbook -vvv -i inventory first-playbook.yml
     ```

3. **Gathering Facts**:
   - The `gather_facts` directive is enabled by default and collects information about target servers. This information is useful for configuration management and automation.

## Step 4: Important Notes

- **Do not remove existing keys**: When adding new public keys to the `authorized_keys` file on the target server, ensure that the existing keys are not removed. Simply append the new key on a new line.
  
- **Naming Conventions**: Make sure that inventory file groups, such as `webservers` or `db_servers`, do not contain spaces. Use underscores if necessary (e.g., `web_servers`).

- **Modules and Tasks**:
   - Use the Ansible shell module to run shell commands. For example, in the playbook above, the `shell` module is used to install Nginx.
   - Ansible modules are continuously updated, so always refer to the latest documentation for available modules.

## Conclusion

This project demonstrates the power of Ansible in configuration management, making it easier to automate and manage multiple servers efficiently. By using passwordless SSH authentication and Ansible playbooks, you can manage server configurations at scale without needing to manually SSH into each server.

---

### Further Resources:
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Playbook Examples](https://github.com/ansible/ansible-examples)
