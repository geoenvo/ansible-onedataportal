# ansible-onedataportal
[Ansible](https://docs.ansible.com) playbooks for automatic deployment of the One Data Portal.

## Requirements
### Target server
* A fresh installed [Ubuntu Server 18.04](https://ubuntu.com/download/server) with a user that is part of the `sudo` group (IMPORTANT: using the root user is not recommended since by default the server provisioning playbook `0_provision_server.yml` will disable root user login for better security)
  ```
  adduser username
  usermod -aG sudo username
  ```
* The server is fully updated and configured with a static IP address (below is an example setting the server IP address to 192.168.1.200)
  ```
  sudo apt update && sudo apt upgrade
  sudo nano /etc/netplan/50-cloud-init.yaml
  ```
  ```
  network:
     ethernets:
         enp0s3:
             dhcp4: no
             addresses: [192.168.1.200/24]
             gateway4: 192.168.1.1
             nameservers:
                 addresses: [8.8.8.8,8.8.4.4]
     version: 2
  ```
  ```
  sudo reboot
  ```
* Ansible installed on the target server
  ```
  sudo apt update
  sudo apt install -y software-properties-common
  sudo apt-add-repository --yes --update ppa:ansible/ansible
  sudo apt install -y ansible
  ansible --version
    ansible 2.9.6
      config file = /etc/ansible/ansible.cfg
      configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python2.7/dist-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]
  ```
  
## How to run
SSH to the target server and perform the following steps:
* Clone this repo (install `git` package if necessary)
  ```
  cd ~
  git clone https://github.com/geoenvo/ansible-onedataportal.git
  cd ansible-onedataportal
  ```
* Configure the mandatory playbook variables
  * ```nano 0_provision_server.yml```
    ```
    server_timezone: set this to the server's timezone (default is Asia/Jakarta)
    ```
  * ```nano 1_install_postgresql.yml```
    ```
    postgres_user_password: the password for the default postgres role
    ```
  * ```nano vars_ckan.yml```
    ```
    ckan_site_url: set this to the IP address or domain name of the server (default is http://192.168.1.200)
    ckan_admin.username: set the username for the CKAN admin user (default is admin)
    ckan_admin.password: set the default password for the CKAN admin user
    ```
* Run the playbooks
    * One by one
      * Must be run in the order of the filename
        ```
        ansible-playbook -K -i inventory 0_provision_server.yml
          BECOME password: enter the user's sudo password
        ansible-playbook -K -i inventory 1_install_postgresql.yml
        ansible-playbook -K -i inventory 2_install_solr.yml
        ansible-playbook -K -i inventory 3_install_ckan.yml
        ansible-playbook -K -i inventory 4_install_ckan_extras.yml
        ansible-playbook -K -i inventory 5_install_oskari.yml
        ```
    * Or run the install all playbook
      * This will run the playbooks above in correct order
        ```
        ansible-playbook -K -i inventory install_all.yml
          BECOME password: enter the user's sudo password
        ```
* Some manual configuration might be needed to achieve the following (this is optional):
  * Configure firewall to further secure the server
  * Reverse proxy a (sub)domain with SSL certificate
  * Change the default passwords for CKAN/Oskari/GeoServer
  * Modify advanced CKAN and Oskari settings
      * Default CKAN configuration file path is `/etc/ckan/ckan/production.ini`
      * Default Oskari configuration file path is `/opt/oskari/oskari-server/resources/oskari-ext.properties`
  * Using and updating a non-english localization
  * Check this repo's wiki TODO
* Reboot the server
  ```
  sudo reboot
  ```
* Access the One Data Portal at the server's IP address or domain name on a web browser:
  * By default the CKAN Data Portal is accessible at http://the.server.ip.address (reverse proxied by NGINX from http://the.server.ip.address:8090)
  * CKAN DataPusher service at http://the.server.ip.address:8800/
  * pycsw service at http://the.server.ip.address:8000/?service=CSW&version=2.0.2&request=GetCapabilities
  * Solr admin at http://the.server.ip.address:8983/solr/
  * Oskari Geoportal at http://the.server.ip.address:8080
  * GeoServer admin at http://the.server.ip.address:8082/geoserver/web
* Done!
