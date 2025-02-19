# 4640-w7-lab-start-w25

# Lab 7: Ansible Web Server Automation

## **1. Cloning the Starter Code**
In the first step, I cloned the starter repository to get the required files.
```bash
git clone https://gitlab.com/cit_4640/4640-w7-lab-start-w25.git
cd 4640-w7-lab-start-w25
```

## **2. Creating a New SSH Key**
I generated a new SSH key named `aws` and stored it in `~/.ssh/`.
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws
```
I left the passphrase empty and verified that the key files exist:
```bash
ls ~/.ssh/aws*
```

## **3. Importing the SSH Key into AWS**
I used the provided script to import my public SSH key into AWS.
```bash
./import_lab_key ~/.ssh/aws.pub
```

## **4. Deploying EC2 Instances with Terraform**
I initialized Terraform and applied the configuration to deploy two EC2 instances.
```bash
terraform init
terraform apply -auto-approve
```
Terraform provided the **public IP addresses** and **DNS names** for both instances.

## **5. Updating the Ansible Inventory (`hosts.yml`)**
I created and updated `ansible/inventory/hosts.yml` with the EC2 instance IPs.
```yaml
---
all:
  children:
    web:
      hosts:
        server-one:
          ansible_host: 34.221.117.168
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
        server-two:
          ansible_host: 44.243.113.168
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
```
I verified the inventory using:
```bash
ansible-inventory --list -y
```

## **6. Configuring Ansible (`ansible.cfg`)**
I updated the Ansible configuration in `ansible.cfg` to avoid SSH key checking issues.
```ini
[defaults]
inventory = inventory/hosts.yml
remote_user = ubuntu
host_key_checking = False
private_key_file = ~/.ssh/aws
```

## **7. Setting Up Required Ansible Files**
### **`files/nginx.conf`**
I created the `nginx.conf` file to configure Nginx.
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### **`templates/index.html.j2`**
I created the Ansible template for `index.html`.
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to {{ ansible_hostname }}</title>
</head>
<body>
    <h1>Server: {{ ansible_hostname }}</h1>
    <p>Deployed using Ansible</p>
</body>
</html>
```

### **`playbook.yml`**
I created the playbook to automate the web server setup.
```yaml
---
- name: Configure web servers
  hosts: web
  become: yes
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create directory for web documents
      ansible.builtin.file:
        path: /var/www/html
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Copy nginx conf file to server
      ansible.builtin.copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'

    - name: Create symbolic link to enable nginx site
      ansible.builtin.file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link

    - name: Generate index.html from template
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Reload and enable nginx service
      ansible.builtin.service:
        name: nginx
        state: reloaded
        enabled: yes
```

## **8. Running the Ansible Playbook**
I executed the playbook to configure the servers.
```bash
ansible-playbook playbook.yml
```

I verified if **Nginx** is running.
```bash
ansible all -m shell -a "systemctl status nginx"
```

I tested the web page by running:
```bash
curl http://<public_ip_1>
curl http://<public_ip_2>
```

I also opened the IPs in my web browser to verify that Nginx was serving the page.

## **9. Cleaning Up AWS Resources**
After completing the lab, I destroyed the infrastructure to avoid charges.
```bash
terraform destroy -auto-approve
```
I also removed the SSH key from AWS:
```bash
./delete_lsb_key
```

## **10. Pushing Work to GitHub**
I initialized Git and pushed my work to GitHub.
```bash
git init
git add .
git commit -m "Completed Lab 7 - Ansible Automation"
git remote set-url origin https://github.com/sim4444/lab7_ansible.git
git push -u origin master
```

## **11. Deliverables**
I submitted my **GitHub repository link**, which includes:
- `README.md` (this file)
- `ansible/` directory with:
  - `ansible.cfg`
  - `inventory/hosts.yml`
  - `files/nginx.conf`
  - `templates/index.html.j2`
  - `playbook.yml`
- Screenshot of the rendered HTML page.

## **12. Conclusion**
In this lab, I successfully:
Used **Terraform** to deploy EC2 instances  
Configured **Ansible** to install and manage Nginx  
Used **Ansible templates** to generate an HTML file dynamically  
Automated web server setup and configuration  


