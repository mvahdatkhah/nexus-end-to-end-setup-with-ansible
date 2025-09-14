# 🚀 Nexus End-to-End Setup with Ansible 🌐

This Ansible playbook automates the **installation and configuration of Sonatype Nexus Repository Manager** on **Ubuntu 22.04**, including:

* 🏗️ Nexus installation and setup
* 💾 LVM-based blobstore with ext4 filesystem
* 🌐 Nginx reverse proxy with SSL (Certbot)
* 🔥 Firewall configuration (ports 80, 443, 8081, 8082)

---

## ✅ Prerequisites

* Ubuntu 22.04 target servers
* Ansible 2.14+ installed on control node
* Public DNS configured for `repo.example.com` and `registry.example.com` (if using Certbot)
* Sudo privileges on target servers
* Internet access for downloading Nexus and packages

---

## 📂 Directory Structure

```
nexus_end_to_end_setup/
├── nexus_end_to_end_setup.yml       # Main playbook
├── roles/
│   └── nexus_end_to_end_setup/
│       ├── vars/
│       │   └── main.yml             # Variables
│       ├── tasks/
│       │   ├── nexus.yml            # Nexus installation tasks
│       │   ├── blobstore.yml        # LVM + ext4 blobstore tasks
│       │   ├── nginx.yml            # Nginx reverse proxy + SSL tasks
│       │   └── firewall.yml         # Firewall tasks
│       └── handlers/
│           └── main.yml             # Service handlers
```

---

## 📝 Variables (`vars/main.yml`)

```yaml
nexus_version: "3.83.2-01"
nexus_arch: "linux-x86_64"
nexus_archive: "nexus-{{ nexus_version }}-{{ nexus_arch }}.tar.gz"
nexus_url: "https://download.sonatype.com/nexus/3/{{ nexus_archive }}"
nexus_install_dir: "/opt/sonatype"
nexus_symlink: "/opt/sonatype/nexus"
nexus_user: "nexus"
nexus_service_file: "/etc/systemd/system/nexus.service"

nginx_sites:
  - server_name: "repo.example.com"
    proxy_pass: "http://127.0.0.1:8081"
  - server_name: "registry.example.com"
    proxy_pass: "http://127.0.0.1:8082"

nexus_blobstore: "/opt/sonatype/sonatype-work/nexus3/{{ ansible_facts['distribution'] | lower }}-{{ ansible_facts['distribution_release'] }}-blobstore"

firewall_ports:
  - 80
  - 443
  - 8081
  - 8082
```

---

## ⚡ Usage

1. **Clone the repository** or copy the playbook files to your Ansible control node.

2. **Edit variables** as needed:

   * Nexus version or architecture
   * Nginx server names
   * Firewall ports
   * Blobstore path (dynamic per OS)

3. **Run the playbook**:

```bash
ansible-playbook -i inventory nexus_end_to_end_setup.yml
```

4. **Prompted for blobstore device**:

```
Enter the block device for blobstore (default: /dev/sdc)
```

5. The playbook will perform:

* 🛠️ Install required packages
* 🏗️ Download and configure Nexus
* 💾 Create LVM volume and ext4 filesystem for blobstore
* 🌐 Configure Nginx reverse proxy and obtain SSL certificates
* 🔥 Open firewall ports 80, 443, 8081, 8082

---

## 💡 Notes

* **DNS Requirement for Certbot:** Ensure your domains (`repo.example.com`, `registry.example.com`) point to the server IP for SSL to work.
* **Testing without public DNS:** You can modify `nginx.yml` to use **self-signed certificates** for local testing.
* **Firewall persistence:** iptables rules are saved to `/etc/iptables/rules.v4`.

---

## 🛠️ Handlers

* **Reload systemd** – reloads systemd configuration
* **Enable Nexus** – enables and starts Nexus at boot
* **Start Nexus** – starts Nexus without enabling at boot

---
