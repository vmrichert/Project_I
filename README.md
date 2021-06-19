## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

Diagrams/Network diagram.PNG

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the **yml** and **cfg** file may be used to install only certain pieces of it, such as Filebeat.

  - C:\Users\dakot\OneDrive\Desktop\Project_I

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.
- What aspect of security do load balancers protect? The load balancers ensures that incoming traffic will be shared by both web servers. Sharing the workload between the three web servers will increase availability.

- What is the advantage of a jump box? The jump box serves as a gateway and will ensure that only authorized users will be able to connect. It is the only one exposed to the public network so it restricts the public access to only one system.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, and system metrics.

- What does Filebeat watch for? Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.

-  What does Metricbeat record? Metric beat detects changes in system metrics, such as CPU usage, attempted SSH logins, sudo escaltion failures, etc.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name        | Function    | IP Address   | Operating System |
|-------------|-------------|--------------|------------------|
| Jump Box    | Gateway     | 10.0.0.4     | Linux            |
| Web-1       |Web Server   | 10.0.0.5     | Linux            |
| Web-2       |Web Server   | 10.0.0.6     | Linux            |
| Web-3       |Web Server   | 10.0.0.7     | Linux            |
|ElkServerA   |Monitoring   | 10.1.0.4     | Linux            |
|LoadBalancerA|Load Balancer| 20.85.218.199| Linux            |

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:
- Availability Zone 1: Web-1, Web-2 and Web-3
- Availability Zone 2: ElkServerA

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 107.2.68.95

Machines within the network can only be accessed by each other.

- Which machine did you allow to access your ELK VM? Web-1, Web-2 and Web-3 send traffic to the Elk server. It also allows TCP traffic from port 5601.

What was its IP address?
	Web-1 10.0.0.5
	Web-2 10.0.0.6
	Web-3 10.0.0.7
	Local Workstation via port 5601 107.2.68.95
	

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 107.2.68.95          |
| ELK      | Yes                 | 107.2.68.95 10.0.0.4 |
|          |                     | 10.0.0.5 10.0.0.6    |
|          |                     | 10.0.0.7	        |
| Web-1    | No                  | 10.1.0.4 10.0.0.4    |
| Web-2    | No                  | 10.1.0.4 10.0.0.4    |
| Web-3    | No                  | 10.1.0.4 10.0.0.4    |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because you can quickly configure and install multiple apps. You simply list tasks in a playbook, and Ansible will go through those tasks for you so you won't have to go through it one by one. This will cut on time and will allow you to focus on other more important tasks.

The playbook implements the following tasks:
- Install Docker (docker.io)
- Installs PIP (python3-pip)
- Increases the virtual memory
- Downloads and launches a docker ELK container

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

C:\Users\dakot\OneDrive\Desktop\Project_I\Diagrams

The playbook is duplicated below.

---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
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

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1  10.0.0.5
- Web-2  10.0.0.6
- Web-3  10.0.0.7

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat detects changes to the filesystem. We use it to collect logs generated by our ELK server container. It collects Apache logs
- Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed 'sudo' escalations, and CPU/RAM statistics.

The playbook below installs Metricbeat on the target hosts. The playbook for installing Filebeat is not included, but looks essential identical - simply replace 'metricbeat' with 'filebeat,' and it will work as expected.

---
- name: Install metric beat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download metricbeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
    delegate_to: localhost
  - name: copy artifact to remote host
    copy:
      src: /etc/ansible/metricbeat-7.4.0-amd64.deb
      dest: /etc/metricbeat-7.4.0-amd64.deb
    # Use command module
  - name: Install metricbeat .deb
    command: dpkg -i /etc/metricbeat-7.4.0-amd64.deb
    # Use copy module
  - name: Drop in metricbeat.yml
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml
    # Use command module
  - name: Enable and Configure docker Module
    command: metricbeat modules enable docker
    # Use command module
  - name: Setup metric beat
    command: metricbeat setup
    # Use command module
  - name: Start metric beat
    command: service metricbeat start
    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

For Elk:
- Update the 'hosts' file to add ELK VM
- Copy the 'install-elk.yml' file to Ansible Control Node
- Run 'ansible-playbook install-elk.yml'

For Filebeat:
- Download Filebeat in /etc/ansible directory by running 'curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > filebeat-config.yml' command
- Copy /etc/ansible/filebeat-config.yml file to file-beat.yml.
- Update filebeat-config.yml to include your ELK machine IP address on line #1106 like so:
 	output.elasticsearch:	
       	 hosts: ["10.1.0.4:9200"]
         username: "elastic"
         password: "changeme"
- Do the same thing to line #1806:
	setup.kibana:
         host: "10.1.0.4:5601"
- Update the filebeat-playbook.yml file to include installer: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
- Run the playbook using 'ansible-playbook filebeat-playbook.yml' command
- Navigate to Kibana using the public IP of your ELK server http://40.122.54.97:5601/app/kibana > Logs:Add log data > System logs > 5:Module Status > Check data to check that the installation worked as expected.

For Metricbeat:
- Download Metricbeat in /etc/ansible directory by running 'curl https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb'
- Copy /etc/ansible/metricbeat-config.yml file to metricbeat.yml
- Update the metricbeat-playbook.yml to include your ELK machine IP address:
	output.elasticsearch:
  	  # Array of hosts to connect to.
  	  hosts: ["10.1.0.4:9200"]
  	  username: "elastic"
  	  password: "changeme"

	setup.kibana:
         host: "10.1.0.4:5601"
- Update the metricbeat-playbook.yml file to include installer: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
- Run the playbook using the command 'ansible-playbook metricbeat-playbook.yml
- Navigate to Kibana using the public IP of your ELK server http://40.122.54.97:5601/app/kibana > Add Metric Data > Docker Metrics > Module Status to check that the installation worked as expected.


