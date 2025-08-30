Hello 


# Ansible Auto Update Playbook

This repository contains an Ansible playbook that automates system updates and upgrades by creating a **daily cron job**.  
The cron job runs every day at **18:00** and logs all output to `/var/log/auto-update.log`.

---

## 🔧 How It Works

The playbook does the following:

1. Ensures that **cron** is installed and running.
2. Optionally puts certain packages (e.g. Kubernetes components) on **hold** so they are not upgraded automatically.
3. Creates a shell script (`/usr/local/bin/auto-update.sh`) that runs:
   - `apt-get update -y`
   - `apt-get upgrade -y` or `apt-get dist-upgrade -y` (depending on variable)
   - `apt-get autoremove -y` and `apt-get autoclean -y`
   - Logs everything to `/var/log/auto-update.log`
4. Creates the log file if it does not exist.
5. Adds a **cron job** that runs the script daily at 18:00.

---

## 📝 Playbook (`site.yml`)

```yaml
---
- name: Automatic update/upgrade with daily cron job
  hosts: all
  become: true
  vars:
    update_hour: 18          # hour of the day for cron job
    update_minute: 0         # minute of the hour
    log_path: /var/log/auto-update.log
    script_path: /usr/local/bin/auto-update.sh

    # Upgrade mode: "safe" = apt upgrade, "dist" = apt dist-upgrade
    upgrade_mode: "safe"

    # Packages to put on hold (skip from upgrades)
    held_packages:
      - kubeadm
      - kubelet
      - kubectl

  tasks:
    - name: Ensure cron is installed
      ansible.builtin.package:
        name: cron
        state: present

    - name: Ensure cron service is running and enabled
      ansible.builtin.service:
        name: cron
        state: started
        enabled: true

    - name: Put selected packages on hold
      ansible.builtin.command: "apt-mark hold {{ item }}"
      register: hold_result
      changed_when: "'hold' in hold_result.stdout or hold_result.rc == 0"
      failed_when: false
      loop: "{{ held_packages }}"
      when: held_packages | length > 0

I tried to run with this command ansible-playbook -i hosts.ini site.yml
but it gave me an error like this : <img width="1153" height="97" alt="image" src="https://github.com/user-attachments/assets/65fb19d4-546d-4083-90fd-57bf5676c523" />

Then i try this command ansible-playbook -i "localhost," -c local site.yml because -i "localhost," -> inline inventory the comma is required and -c local,-> run tasks locally without SSH

 Future Improvements

Add automatic reboot if a kernel upgrade requires it.
Extend support for remote servers with a proper inventory file.
Make package hold/upgrade lists configurable per host/group.
<img width="1257" height="715" alt="image" src="https://github.com/user-attachments/assets/6119ad3c-cca3-46bf-88ed-599a5f733646" />
# Author # 
Slobodan Milojevic

