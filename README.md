# **RHCSA Ansible Lab Environment Builder**

This Ansible project automatically provisions a self-contained lab environment suitable for practicing the Red Hat Certified System Administrator (RHCSA) exam objectives. All virtual machines are built on a **single KVM host**, which also serves as the Ansible control node.

## **Prerequisites**

1. **VM Host / Control Node:** One server running CentOS Stream 9 (or similar RHEL derivative) which has:  
   * **Git** installed.  
   * The user running the playbook must have sudo privileges.  
   * **RAM Requirements (CRITICAL):** The playbook provisions **8 GB RAM** for each of the three VMs, requiring a minimum of **24 GB free RAM** during execution. Ran into an issue where CentOS would freeze during installing at 3GB of RAM.
   * **Storage Requirement (CRITICAL):** The playbook defines a new Libvirt storage pool (rhcsa\_storage) that uses the **/vmfs** mount point. This mount point must exist and have **at least 80 GB of free space** available for all VMs and the mirrored repository content.

## **⚙️ Initial Configuration and Software Requirements**

To avoid module compatibility errors (like the ones encountered during development), you must ensure your Ansible environment meets these minimum requirements:

| Component | Minimum Required Version | Installation Command |
| :---- | :---- | :---- |
| **Ansible Core** | 2.14.x | (Usually installed via DNF) |
| **ansible.posix** | 2.1.0+ | ansible-galaxy collection install ansible.posix |
| **community.general** | 6.6.0+ | ansible-galaxy collection install community.general |
| **community.libvirt** | **2.0.0+** | ansible-galaxy collection install community.libvirt |

**Crucial Step:** Run the ansible-galaxy collection install command for community.libvirt to ensure the virt\_pool module supports the path and type parameters used in this playbook.

### **1\. Update inventory.ini**

You need to define the static IP addresses for the VMs. The vm\_host group automatically uses localhost.

| Placeholder | Location | Description |
| :---- | :---- | :---- |
| \<REPO\_VM\_IP\> | \[rhcsa\_repo\] | The chosen static IP for the Repository VM (e.g., 192.168.1.240). |
| \<VM1\_IP\> | \[rhcsa\_vms\] | The chosen static IP for Exam VM 1 (e.g., 192.168.1.245). |
| \<VM2\_IP\> | \[rhcsa\_vms\] | The chosen static IP for Exam VM 2 (e.g., 192.168.1.246). |

### **2\. Execution**

Run these steps *on your VM Host / Control Node*:

\# 1\. Clone the repository  
git clone \[https://github.com/tonyfdh/rhcsa\_lab.git\](https://github.com/tonyfdh/rhcsa\_lab.git)  
cd rhcsa\_lab

\# 2\. Update required collections to minimum versions  
\# ansible-galaxy collection install community.libvirt  
\# ansible-galaxy collection install community.general

\# 3\. Update inventory.ini with your chosen IPs (required)  
\# nano inventory.ini

\# 4\. Execute the main playbook  
ansible-playbook rhcsa\_lab\_setup.yml \-K

**Note:** The playbook will install KVM, build the VMs, and mirror CentOS content. This process takes significant time (especially mirroring the repository).