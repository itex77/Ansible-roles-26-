Role Name
=========

playbook_z.ym

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).


Sequence of steps for installing and configuring zabbix-agent on multiple servers
---------------------------------------------------------------------------------

sudo apt update
sudo apt install ansible
sudo vim /etc/ansible/hosts - #список серверов и групп серверов для развёртывания клиента zabbix
  
sudo vim ansible.cfg - #отключение проверки ключей при подключении по SSH
[defaults]
host_key_checking = false
inventory         = /etc/ansible/hosts

ansible --version
sudo ssh-copy-id -i /home/itex/.ssh/id_rsa.pub itex@10.129.0.35
sudo ssh-copy-id -i /home/itex/.ssh/id_rsa.pub itex@10.129.0.5
ansible -i hosts servers -m ping
sudo ansible-galaxy init deploy_zabbix-agent_on_hosts

# configure roles.
+-- ansible.cfg
+-- hosts
+-- playbook_z.yml
L-- roles
    L-- deploy_zabbix-agent_on_hosts
        +-- defaults
        ¦   L-- main.yml
        +-- files
        +-- handlers
        ¦   L-- main.yml
        +-- meta
        ¦   L-- main.yml
        +-- README.md
        +-- tasks
        ¦   +-- main.yml
        ¦   +-- main.yml.copy
        ¦   L-- main.yml.only_start_zbbx
        +-- templates
        ¦   +-- zabbix-agentd.conf.j2
        ¦   L-- zabbix-agentd.conf.j2.copy
        +-- tests
        ¦   +-- inventory
        ¦   L-- test.yml
        L-- vars
            L-- main.yml
  
#handlers
- name: zabbix-agent systemd
  systemd:
          name: zabbix-agent.service
          enabled: yes
          state: started

#tasks
---
# tasks file for deploy_zabbix-agent_on_hosts
- block: # ========Block for RedHat======
  - name: Install zabbix-agent on RedHat Family
    ansible.builtin.yum:
            name: zabbix-agent
            state: latest

  - name: Stop service zabbix-agent
    ansible.builtin.service:
            name: zabbix-agent
            state: stopped

  - name: Remove zabbix config file
    ansible.builtin.file:
            path: /etc/zabbix_agentd.conf
            state: absent

  - name: Create new zabbix config file from templates
    ansible.builtin.template:
            src: zabbix_agentd.conf.j2
            dest: /etc/zabbix_agentd.conf

  - name: Start service zabbix-agent
    ansible.builtin.service:
            name: zabbix-agent
            state: started
            enabled: yes

  when: ansible_os_family == "RedHat"

- block: #=======Block for Debian======
  - name: Install zabbix-agent on Debian Family
    ansible.builtin.apt:
            name: zabbix-agent
            state: present

  - name: Stop service zabbix-agent
    ansible.builtin.service:
            name: zabbix-agent
            state: stopped

  - name: Remove zabbix config file
    ansible.builtin.file:
            path: /etc/zabbix/zabbix_agentd.conf
            state: absent

  - name: Create new zabbix config file from templates
    ansible.builtin.template:
            src: zabbix_agentd.conf.j2
            dest: /etc/zabbix/zabbix_agentd.conf

  - name: Start service zabbix-agent
    ansible.builtin.service:
            name: zabbix-agent
            state: started
            enabled: yes

  when: ansible_os_family == "Debian"
  
#templates: zabbix-agentd.conf.j2
  PidFile=/run/zabbix/zabbix_agentd.pid
  LogFile=/var/log/zabbix/zabbix_agentd.log
  LogFileSize=0
  Server=10.129.0.32
  ServerActive=127.0.0.1
  Hostname={{ ansible_hostname }}
  Include=/etc/zabbix/zabbix_agentd.d/*.conf
