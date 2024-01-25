# Ansible

## Install 
### pipx
    sudo dnf install pipx
    pipx install --include-deps ansible
    pipx inject --include-apps ansible argcomplete

### Upgrade
    pipx upgrade --include-injected ansible

### Test
    python3 -m pip -V
    ansible --version
    ansible-community --version

### Config
    ansible-config init --disabled -t all > ansible.cfg

## inventory
### setup
    # single host
    bpg-debian12 ansible_host=192.168.1.170 ansible_user=debian
### test
    ansible bpg-debian12 -m ping -i inventory
    ansible all -m ping -i inventory
### envvar
    export ANSIBLE_INVENTORY=inventory
    
## playbook
### create test
```
---
- name : test user
  hosts: bpg-debian12
  remote_user: debian
  become: true

  tasks:
  - name: get id
    shell: id
    register: results
  - name: Add the user 'clinux'
    ansible.builtin.user:
      name: clinux
      comment: created by ansible
  - name: break
    shell: exit 1
    ignore_errors: true
  - name: Remove the user 'clinux'
    ansible.builtin.user:
      name: clinux
      state: absent
      remove: yes

  - name: Install some packages
    package:
      name:
        - htop
        - curl
        - nmon
        - net-tools
        - nginx
      state: latest

  - name: Make sure a service unit is running
    ansible.builtin.systemd_service:
      state: started
      enabled: true
      name: nginx

  - debug:
      var: results.stdout 
```
### create post tofu root
```
---
- name: "tofu : setup root key"
  ansible.builtin.copy:
    remote_src: true
    src: /home/debian/.ssh/authorized_keys
    dest: /root/.ssh/authorized_keys
    owner: root
    group: root
    mode: '0600'

- name: "tofu : allow root sshkey"
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '(.*)PermitRootLogin(.*)$'
    line: 'PermitRootLogin without-password'
    owner: root

- name: "tofu : reload ssh"
  service:
    name: ssh
    state: restarted
```

### Execute
    ansible-playbook plb_get_user.yml -i inventory
    export ANSIBLE_INVENTORY=inventory
    ansible-playbook plb_get_user.yml


## Roles 
### Create 
#### galaxy
    ansible-galaxy init roles/post_tofu --offline
#### custom
##### structure
```
mkdir -p roles/post_tofu/tasks
```
##### tasks
###### main

    cat << EOF > roles/post_tofu/tasks/main.yml
    - include_tasks: copy_ssh_key_root.yml
    EOF

###### task sshkey
    cat << EOF > roles/post_tofu/tasks/copy_ssh_key_root.yml
    ---
    - name: copy ssh authozied_keys to root
      ansible.builtin.copy:
        src: /home/debian/.ssh/authorized_keys
        dest: /root/.ssh/authorized_keys
        remote_src: yes
        owner: root
        group: root

    - name: Update and upgrade apt packages
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400 #One day
    EOF


##### playbook

    cat << EOF > deploy_post_tofu.yml
    ---
    - name : post actions tofu
      hosts: debian12
      remote_user: debian
      become: true

      roles:
        - post_tofu
    EOF        

##### run
```
ansible-playbook deploy_post_tofu.yml
```
### Nginx 
#### structure
```
mkdir -p roles/nginx/{tasks,handlers,files}
```
#### tasks
```
cat << EOF > roles/nginx/tasks/main.yml
```
```
---
- include_tasks: install_packages.yml
- include_tasks: configuration.yml
EOF
```
```
cat << EOF > roles/nginx/tasks/install_packages.yml
```
```
---
- name: Install nginx
  package:
    name:
      - nginx
      - net-tools
    state: latest

- name: Make sure a service unit is running
  ansible.builtin.systemd_service:
    state: started
    enabled: true
    name: nginx
EOF
```
```
cat << EOF > roles/nginx/tasks/configuration.yml
- name: copy configuration
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
  notify: restart_nginx
  tags:
    - configuration_nginx
EOF
```
#### handlers
```
cat << EOF > roles/nginx/handlers/main.yml
- name: Restart nginx
  service:
    name: nginx
    state: restarted
  listen: restart_nginx
EOF
```
#### files
```
cat << EOF > roles/nginx/files/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access_default.log;

	##
	# Gzip Settings
	##

	gzip on;

	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_buffers 16 8k;
	gzip_http_version 1.1;
	gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
EOF
```
#### playbook
```
cat << EOF > deploy_webservers.yml
---
- name : post actions tofu
  hosts: webservers
  become: true
  roles:
    - nginx
EOF        
```
##### run
```
ansible-playbook deploy_webservers.yml
```