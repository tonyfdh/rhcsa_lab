# RHCSA Ansible Lab Environment Builder

This Ansible project automatically provisions a self-contained lab environment suitable for practicing the Red Hat Certified System Administrator (RHCSA) exam objectives. All virtual machines are built on a single KVM host.

## Prerequisites

1. **Ansible Control Node:** Ansible must be installed on the machine you run the playbook from (e.g., your local machine or a dedicated control host). I use my VM Host as the control node..

2. **VM Host (Server):** One physical server running CentOS Stream 9 (or similar RHEL derivative) that will host all VMs. KVM/Libvirt will be installed on this server.

3. **SSH Access:** Your Ansible control node must be able to connect to `Server` via SSH using key-based authentication (recommended).

4. **IP Configuration:** Ensure the IP addresses you choose for the VMs are available/reserved in your network.

## ⚙️ Initial Configuration Steps

Before executing the main playbook, you **must** update the placeholders in the configuration files after cloning this repository.

### 1. Update `inventory.ini`

You need to define the connection details for your physical VM host, `Server2`, and the static IP addresses for the VMs.

| Placeholder | Location | Description |
| :--- | :--- | :--- |
| `<SERVER2_IP_ADDRESS>` | `[vm_host]` | The public or internal IP address of your physical CentOS Stream 9 server. |
| `<SSH_USER>` | `[vm_host]` | The username you use to SSH into `Server2`. This user must have `sudo` privileges. |
| `<REPO_VM_IP>` | `[rhcsa_repo]` | The chosen static IP for the Repository VM (e.g., `192.168.1.240`). |
| `<VM1_IP>` | `[rhcsa_vms]` | The chosen static IP for Exam VM 1 (e.g., `192.168.1.245`). |
| `<VM2_IP>` | `[rhcsa_vms]` | The chosen static IP for Exam VM 2 (e.g., `192.168.1.246`). |

### 2. Update `ansible.cfg`

| Placeholder | Location | Description |
| :--- | :--- | :--- |
| `<SSH_USER>` | `[defaults]` | The same SSH username used in `inventory.ini`. This user will be used to connect to all hosts, including the new VMs after they are built. |

### 3. Execution

Once configured, clone the repository onto your Ansible control node and run the playbook:

```bash
# Clone the repository (if not already done)
git clone [https://github.com/tonyfdh/rhcsa_lab.git](https://github.com/tonyfdh/rhcsa_lab.git)
cd rhcsa_lab

# Execute the main playbook
ansible-playbook rhcsa_lab_setup.yml