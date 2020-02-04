## Step 1 - Questions 1 to 7 
## Step 2 - Questions 8 to 11
## Step 3 - Questions 12 to 17  

### Step 1  
**Question 1**  
Create inventory file with below content  

[dev]  
node1.realmX.example.com  
 
[test]  
node2.realmX.example.com  

[prod]  
node3.realmX.example.com  
node4.realmX.example.com  
 
[balancer]  
node5.realmX.example.com  

[webservers:children]  
prod  

Create ansible.cfg file  
<pre>
# vim ansible.cfg  
[defaults]  
inventory = /home/devops/ansible/inventory  
remote_user = devops  
ask_pass = false  
#roles_path = /home/devops/ansible/roles:/usr/share/ansible/roles  
#vault_password_file = /home/devops/ansible/secret.txt  
 
[privilege_escalation]  
become = true  
become_method = sudo  
become_user = root  
become_ask_pass = false  
</pre>

**Question 2**  
Ansible Install (on Master node)  

<pre>
$ sudo yum install ansible -y  
</pre>

**Question 3**  
Adhoc script  
Create an adhoc.sh script to configure yum repository on all the nodes.  
URL is available in http://content.example.com/yum/repository  

<pre>
# cat /adhoc.sh  
ansible all -m yum_repository -a 'name=ex407-yum \  
description="test description" \  
baseurl=http://content.example.com/rhel7.6/x86_64/dvd/ \  
gpgcheck=yes \  
enabled=yes \  
state=present \  
gpgkey=http://content.example.com/rhel7.6/x86_64/dvd/RPM-GPG-KEYredhat-release'  

ansible all -m rpm_key -a "state=present key=http://content.example.com/rhel7.6/x86_64/dvd/RPM-GPG-KEY-redhatrelease"  
# chmod 755 adhoc.sh  
# ./adhoc.sh  
</pre>

Verification:  
<pre>
ansible all –m command –a ‘ls –l /etc/yum.repos.d'  
</pre>

**Question 4**  
Download roles using galaxy. Two links will be provided.  
Create a folder 'roles' in current path.  
Create an extraction file named 'requirements.yml' inside roles folder.  

<pre>
#  cat requirements.yml  
- src: link  
   name: balancer  
- src: link  
   name: phpinfo  
# ansible-galaxy install -r roles/requirements.yml  
</pre>

**Question 5**  
A play should be created using roles balancer and php.  
The load should be balanced across two servers.  
Each time when you reload the web page, the display should be  
"Welcome \<hostname\> on \<IP\>"


<pre>
$ cat roles.yml  
- name: user role1 
  hosts: webservers   
  roles:     
  - roles/phpinfo 
 
- name: user role2   
  hosts: balancer   
  roles:     
  - roles/balancer
$
</pre>
 
**Question 6**.  
Create an encrypted file named vault.yml.  
Password to encrypt the file should be as \"P@ssw0rd\".  
Save the \"P@ssw0rd\" in the file named "secret.txt" in current path.  
Inside vault.yml, store the variable names like below:  
pw_manager: Iammgr  
pw_developer: Iamdev  
Note: This yaml file should be accessible from anywhere in the system.  

<pre>
$ cat secret.txt 
P@ssw0rd 
$ 

$ ansible-vault create --vault-password-file=secret.txt vault.yml 
Enter the below content  
pw_manager: Iammgr  
pw_developer: Iamdev 
$ 

$ vim /home/devops/ansible/ansible.cfg 
vault_password_file = /home/devops/ansible/secret.txt 
$ 
</pre>

Verificaton:  
<pre>
$ ansible-vault view vault.yml 
</pre>

**Question 7**.  
Download a file from the link and save as local.yml. 
Change the encryption password of the file from \<password_1\> to \<password_2\>.  

<pre>
$ wget http://content.example.com./  -O local.yml
$ ansible-vault rekey local.yml 
Vault password: 
New vault password: 
Confirm new vault password:
Rekey successful 
$ 
</pre>

### Step 2 
**Question 8**  
Download the hostfile template from the link and complete the template so that it creates and entry in dev group.  
The contents on /etc/hosts should copied to /etc/myhosts.  

<pre>
$ cat hosts.j2  
{% for host in groups[‘all’] %}  
{{hostvars[host][‘ansible_facts’][‘default_ipv4’][‘address’]}} {{hostvars[host][‘ansible_facts’][‘fqdn’]}} {{hostvars[host][‘ansible_facts’][‘hostname’]}}  
{% endfor %}  
$  

$  cat myhosts.yml  
--- 
- name: copy content using template   
  hosts:  all   
  tasks:     
  - name: copy using template       
    template:         
      src:  hosts.j2         
      dest: /etc/myhost       
    when: inventory_hostname in groups['dev']
 $ 
</pre>  

**Question 9**  
Install httpd, firewalld, start services, enable firewall service for httpd.  
Create a playbook to install httpd and firewall.  
Download the template to /home/greg/ansible folder.  

Download template
<pre>
$ pwd 
/home/greg/ansible 
$ wget http://example.com/template.j2 -O template.j2 
</pre> 

Create apache role in roles directory 
<pre>
$ mkdir /home/greg/ansible/roles 
$ cd /home/greg/ansible/roles 
$ pwd 
/home/greg/ansible/roles 
$ ansible-galaxy init apache
</pre> 

Update template and move into roles' template directory  
<pre>
$ pwd 
/home/greg/ansible 
$ mv template.j2 apache/templates
$ echo '{{ ansible_hostname }} {{ ansible_default_ipv4['address'] }}' >> apache/templates/template/j2 
</pre>

Create apache deployment yaml file  
<pre>
$ pwd
/home/greg/ansible 
$ cat apache_role.yml 
---
- name: apache_role.yml
  hosts: dev, test 
  roles:
  - apache 
$ 
</pre> 

Create tasks in apache role  
<pre>
$ pwd 
/home/greg/ansible 
$ cd roles/apache/tasks
$ cat main.yml
# tasks for apache 
- name: install httpd and firewalld
  yum: 
    name: "{{ item }}" 
    state: latest 
  loop:
  - httpd
  - firewalld 
    
- name: Enable services 
  service:
    name: "{{ item }}" 
    state: started 
    enabled: yes 
  loop:
  - httpd 
  - firewalld 
  
- name: Enable firewall rule 
  firewalld:
    immediate: yes 
    permanent: yes 
    service: http
    state: enabled 
    
- name: create index.html 
  copy: 
    src: template.j2 
    dest: /var/www/html/index.html 
$
</pre>





