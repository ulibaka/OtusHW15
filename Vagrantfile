# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = 'centos/7'
#  config.vm.box_url = 'https://cloud.centos.org/centos/8/x86_64/images/CentOS-8-Vagrant-8.4.2105-20210603.0.x86_64.vagrant-virtualbox.box'
#  config.vm.box_download_checksum = 'dfe4a34e59eb3056a6fe67625454c3607cbc52ae941aeba0498c29ee7cb9ac22'
#  config.vm.box_download_checksum_type = 'sha256'

config.vm.define "nginx" do |server|
config.vm.synced_folder ".", "/vagrant", disabled: true
  server.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  server.vm.host_name = 'nginx'
  server.vm.network :private_network, ip: "192.168.56.150"

           server.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
           SHELL


end

$script = <<-"SCRIPT"
mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
yum install -y epel-release && yum install -y ansible
mkdir ansible && cd ansible
echo "StrictHostKeyChecking no" >> ~/.ssh/config
cat << _EOF_ > ansible.cfg
[defaults]
# some basic default values...
inventory      = hosts
remote_user    = vagrant
host_key_checking = False
retry_files_enabled = False
_EOF_

cat << _EOF_ > hosts
[my_hosts] 
192.168.56.150 ansible_user=vagrant ansible_ssh_pass=vagrant
_EOF_

ansible my_hosts -m ping

cat << _EOF_ > nginx.conf.j2
# {{ ansible_managed }}
events {
    worker_connections 1024;
}

http {
    server {
        listen       {{ nginx_listen_port }} default_server;
        server_name  default_server;
        root         /usr/share/nginx/html;

        location / {
        }
    }
}
_EOF_

cat << _EOF_ > nginx.yml
---
- name: NGINX | Install and configure NGINX
  hosts: my_hosts 
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: NGINX | Install EPEL Repo package from standart repo
      yum:
        name: epel-release
        state: present
      tags:
        - epel-package
        - packages

    - name: NGINX | Install NGINX package from EPEL Repo
      yum:
        name: nginx
        state: latest
      notify:
        - restart nginx
      tags:
        - nginx-package
        - packages

    - name: NGINX | Create NGINX config file from template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
      tags:
        - nginx-configuration

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes
    
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
_EOF_

ansible-playbook nginx.yml

SCRIPT

# Cent OS 8.2
# config used from this
# https://github.com/eoli3n/vagrant-pxe/blob/master/client/Vagrantfile
  config.vm.define "ansible" do |client|
    client.vm.host_name = 'ansible'
    client.vm.network :private_network, ip: "192.168.56.160"
    client.vm.provider :virtualbox do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
         client.vm.provision "shell", inline: $script
        end
end

