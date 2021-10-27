# Cybersecurity-BC-Project1
1st Project
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

     /Onedrive/Pictures/homework12.png

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

---
- name: Config Web Server VM With Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
     force_apt_get: yes
     update_cache: yes
     name: docker.io
     state: present
  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present
  - name: Install Docker python module
    pip:
      name: docker
      state: present
  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80
  - name: Enable Docker service
    systemd:
      name: docker
      enabled: yes

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl  -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/beats/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

---
- name: installing and launching metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.6.1-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/beats/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure system module
    command: metricbeat modules enable system

  - name: setup metricbeat
    command: metricbeat setup

  - name: start metricbeat service
    command: service metricbeat start

  - name: enable service filebeat on boot
    systemd:
      name: metricbeat
      enabled: yes

---
  - hosts: elk
    become: true
    tasks:

    - name: docker.io
      apt:
         force_apt_get: yes
         update_cache: yes
         name: docker.io
         state: present

    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Docker python module
      pip:
        name: docker
        state: present

    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly efficent and avaliable, in addition to restricting unwanted access to the network.
- What aspect of security do load balancers protect? What is the advantage of a jump box?_
   Load Balancers protect the avaliability aspect of the CIA triad. The advantage of a jumpbox is it connects to the internet, and allows our other VM's to stay offline.
Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file system and system metrics.
- What does Filebeat watch for? Monitors the log files for wherever you tell it to, collects them, and then transfers them to be indexed.
- What does Metricbeat record? Collects system metrics and from the OS and services running on your server.

The configuration details of each machine may be found below.


| Name      | Function | IP Address | Operating System |
|-----------|----------|------------|------------------|
| Jump Box  | Gateway  | 10.1.0.4   | Linux            |
| Web-1     | Server   | 10.1.0.5   | Linux            |
| Web-2     | Server   | 10.1.0.6   | Linux            |
| Elkmachine| Server   | 10.0.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet.

Only the Home machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 23.121.228.177

Machines within the network can only be accessed by RedTeamNET.
- Which machine did you allow to access your ELK VM? What was its IP address?_
  I allowed my jumpbox to access my elk server. The IP address is  20.112.98.181

A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible | Allowed IP Addresses |
|-----------|---------------------|----------------------|
| Jump Box  | Yes                 | 23.121.228.177       |
| Web-1     | No                  | 10.1.0.4             |
| Web-2     | No                  | 10.1.0.4             |
| ElkMachine| No                  | 10.1.0.4             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- You can easily set it up on different machines by just running the automated script.

The playbook implements the following tasks:
- Install Docker.io
- Install Python.pip
- Install Docker
- Command- sysctl -w vm.max_map_count=262144
- Launch docker container elk


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

   /OneDrive/Pictures/sebpelkrunningPROJECT1

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- 10.1.0.5 and 10.1.0.6

We have installed the following Beats on these machines:
- Metricbeat and Filebeat

These Beats allow us to collect the following information from each machine:
- Metricbeat collects system metrics and filebeat collects and transfers data.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned:

SSH into the control node and follow the steps below:
- Copy the filebeat-playbook.yml file to the web VM's.
- Update the hosts file to include the correct IP address for web and elk VM's.
- Run the playbook, and navigate to Kibana to check that the installation worked as expected.

- _Which file is the playbook? Where do you copy it? /etc/ansible/beats/filebeat-playbook.yml
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on? You edit the ansible hosts file and set the host to whatever machine group you want to add it on._
- _Which URL do you navigate to in order to check that the ELK server is running?   http://[your.VM.IP]:5601/app/kibana
