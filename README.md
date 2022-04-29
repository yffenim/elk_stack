## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram](https://raw.githubusercontent.com/yffenim/elk_stack/main/Images/NetworkDiagram.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the each Ansible configuration and playbook files may be used to install only certain pieces of it, such as Filebeat.

**Config files**

[Ansible](https://github.com/yffenim/elk_stack/blob/main/Files/ansible.yml 'Ansible Config')

[Hosts](https://github.com/yffenim/elk_stack/blob/main/Files/hosts.txt 'Hosts file') 

[Filebeat](https://github.com/yffenim/elk_stack/blob/main/Files/filebeat-config.yml 'Filebeat Config')

[Metricbeat](https://github.com/yffenim/elk_stack/blob/main/Files/metricbeat-config.yml 'Hosts file') 

**Playbooks**

[Pentesting](https://github.com/yffenim/elk_stack/blob/main/Files/pentesting.yml 'pentesting playbook')

[ELK](https://github.com/yffenim/elk_stack/blob/main/Files/install-elk.yml 'ELK playbook')

[Filebeat](https://github.com/yffenim/elk_stack/blob/main/Files/filebeat-playbook.yml 'filebeat playbook')

[Metricbeat](https://github.com/yffenim/elk_stack/blob/main/Files/metricbeat.yml 'metricbeat playbook')


This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available by distributing the incoming traffic to the network to multiple servers so that there is no single point of failure. Load balancers offer resiliency against DDoS attacks by ensuring that our resource is not dependent on a single server and can optimize for performance in terms of response times and reliability depending on the forwarding rules configuration. By configuring these rules, the load balancer can also restrict access to the network based on IP.

Currently, our load balancer is only accessble via `HTTP` through port 80, creating a segmentation of authentification by separating the `HTTP` requests to the servers from admin access to the same servers  which we have set up via `SSH`. (NOTE: The `SSH` keys are currently stored in the ansible container which is *not* secure practice as it is easily accessible. Instead, please remember to store `SSH` keys in an external repository.) 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network and system logs (Filebeat) and system metric (Metricbeat). Elasticsearch then indexes the raw logs then visalized with Kibana.

The configuration details of each machine may be found below.

| Name                  | Function           | IP Address (public) | IP Address (private) | Operating System |
|-----------------------|--------------------|---------------------|----------------------|------------------|
| Jump-Box-Provisioner  | Gateway            | 20.70.13.75         | 10.0.0.4             | Linux            |
| LB-RedTeam            | Load Balancer      | 20.37.4.117         | N/A                  | N/A              |
| Web-1                 | Server for DVWA    | 20.37.4.117         | 10.0.0.5             | Linux            |
| Web-2                 | Server for DVWA    | 20.37.4.117         | 10.0.0.6             | Linux            |
| ELK-VM                | Monitor            | 20.213.102.100      | 10.1.0.4             | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the load balancer can accept connections from the Internet.  Access to this machine is only allowed through port 80 from from my current home station's public IP address. Machine within the network can only bea accessed by the Jump Box (Jump-Box-Provisioner) which itself is only accessible via `SSH` from one IP address. We have used network peering to set up `SSH` access to the ELK VM from the Jump Box even though they are on different virtual networks (Vnet-ELK for the ELK VM and VirtualNetwork-A1 for the Jump Box and web servers). 

A summary of the access policies in place can be found in the table below.

| Name                               | Publicly Accessible     | Allowed IP addresses                     |
|------------------------------------|-------------------------|------------------------------------------|
| Jump Box                           | Yes, restricted by IP   | `<myworkstationip:22>`                   |
| Load Balancer                      | Yes, restrcited by IP   | `<myworkstationip:80>`                   |
| Web Servers (1 & 2)                | No                      | 10.0.0.4:22                              |
| ELK                                | Yes, restricted by IP   | 10:0.0.4:22 and `<myworkstationip:5601> `|           |---------------------------------------------------------------------------------------------------------|

*`<myworkstationip:22>` represents the public IP of my local workstation that I would prefer not to be listed in a public repo. Thank you.

### Elk Configuration

Ansible was used to automate configuration of all of the ELK machine meaning that no configuration was performed manually on the deployed webservers. The advantage of automating configuration is that we can deploy or update multiple machines from a single playbook which saves time, reduces errors, and minimalizes carpal tunnel. 

Having all config in one place also makes it easier to troubleshoot as complexity is reduced when backtracking;tThe playbook format is also very read-able and concise with only a few possible options for each command, thereby helping us prevent errors to begin with.

The ELK playbook implements the following tasks:
  - install docker.io
  - install python 3
  - pip install the docker module
  - use the sysctl to increase memory (and virtual memory) allocation
  - download and launch the elk container
  - enable the service docker on boot, ensuring persistance through reboots

The following screenshot displays the result of running `docker ps`: it lists the dockers currently started.:
_ADD SCREENSHOT_

Successfully configuring and running a playbook should look something like this:
_ADD SCREENSHOT_

### Target Machines & Beats

This ELK server is configured to monitor the web servers at 10.0.0.5 and 10.0.0.6.

We have installed the following Beats on these machines:

- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- **Filebeat** *monitors the log files or locations that you specify, collects log events*
- **Metricbeat** *collect metrics from the operating system and from services running on the server.*

Both the Filebeat and Metricbeat will send the collected data to Elasticsearch, which then passes data to Kibana for heasily uman-readable analysis.
(sources: [filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html) and [metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html))

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

`SSH` into the Jumpbox from your public IP address usng this syntax template: `ssh -i <ssh-key-location> <name>@<publicIPaddress>`

Start docker: `sudo docker start <docker_name>`

Attach docker: `sudo docker attach <docker_name>`

Copy or download the config and playbook files above to `/etc/ansible`.

Specify to include the webservers in the hosts file in the same directory.

Run the playbooks: `sudo ansible-playbook <playbookname>.yml` and navigate to the ELK IP from your browser to check that the installation worked as expected: `http://<ELK-public-ip>:5601/app/kibana` in your browser to check that the installation worked as expected.

