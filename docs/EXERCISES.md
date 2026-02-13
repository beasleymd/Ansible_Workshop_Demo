# Ansible Workshop Exercises

This document contains hands-on exercises for learning Ansible automation. Each exercise builds on previous concepts and demonstrates key Ansible features.

## Prerequisites

- Complete the [SETUP.md](SETUP.md) guide
- Have at least one target host configured
- Basic understanding of YAML syntax
- Familiarity with command line

---

## Exercise 1: Ad-Hoc Commands

Learn to use Ansible ad-hoc commands for quick tasks.

### 1.1 Ping All Hosts
```bash
ansible all -m ping
```

**Expected Result**: All hosts should respond with "pong"

### 1.2 Check Disk Space
```bash
ansible all -m shell -a "df -h"
```

### 1.3 Install a Package
```bash
ansible webservers -m package -a "name=vim state=present" --become
```

### 1.4 Gather Facts
```bash
ansible all -m setup
```

**Challenge**: Filter facts to show only IP addresses
```bash
ansible all -m setup -a "filter=ansible_default_ipv4"
```

---

## Exercise 2: Your First Playbook

Create a simple playbook from scratch.

### 2.1 Create a Playbook

Create `playbooks/hello.yml`:

```yaml
---
- name: Hello World Playbook
  hosts: all
  tasks:
    - name: Print message
      debug:
        msg: "Hello from {{ inventory_hostname }}!"
    
    - name: Create a file
      file:
        path: /tmp/hello.txt
        state: touch
        mode: '0644'
      become: true
    
    - name: Display date
      command: date
      register: current_date
    
    - name: Show date
      debug:
        var: current_date.stdout
```

### 2.2 Run the Playbook

```bash
# Check syntax
ansible-playbook playbooks/hello.yml --syntax-check

# Dry run
ansible-playbook playbooks/hello.yml --check

# Execute
ansible-playbook playbooks/hello.yml
```

---

## Exercise 3: Working with Variables

Learn how to use variables in playbooks.

### 3.1 Playbook with Variables

Create `playbooks/variables.yml`:

```yaml
---
- name: Variables Demo
  hosts: all
  vars:
    package_name: htop
    service_name: sshd
  
  tasks:
    - name: Install package
      package:
        name: "{{ package_name }}"
        state: present
      become: true
    
    - name: Check service status
      service:
        name: "{{ service_name }}"
        state: started
      become: true
```

### 3.2 Using Group Variables

Edit `group_vars/webservers.yml`:

```yaml
---
http_port: 8080
max_clients: 200
```

Create a playbook that uses these variables.

---

## Exercise 4: Handlers and Notifications

Learn to use handlers for service management.

### 4.1 Create Playbook with Handler

Create `playbooks/handlers.yml`:

```yaml
---
- name: Handlers Demo
  hosts: webservers
  become: true
  
  tasks:
    - name: Install Apache
      package:
        name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"
        state: present
      notify: restart web server
    
    - name: Copy configuration file
      copy:
        content: "# Apache Configuration\nServerName localhost\n"
        dest: /tmp/apache.conf
      notify: restart web server
  
  handlers:
    - name: restart web server
      service:
        name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"
        state: restarted
```

### 4.2 Run and Observe

```bash
ansible-playbook playbooks/handlers.yml
```

**Note**: Handlers only run when notified and only once at the end.

---

## Exercise 5: Using Templates

Learn to create dynamic configuration files with Jinja2 templates.

### 5.1 Create a Template

Create `templates/motd.j2`:

```jinja
Welcome to {{ inventory_hostname }}!

System Information:
- OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
- Kernel: {{ ansible_kernel }}
- Architecture: {{ ansible_architecture }}
- IP Address: {{ ansible_default_ipv4.address | default('N/A') }}

Managed by Ansible
Last updated: {{ ansible_date_time.date }}
```

### 5.2 Deploy Template

Create `playbooks/template.yml`:

```yaml
---
- name: Deploy MOTD
  hosts: all
  become: true
  
  tasks:
    - name: Deploy message of the day
      template:
        src: motd.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'
```

### 5.3 Verify

```bash
ansible-playbook playbooks/template.yml
ssh ansible@<host> "cat /etc/motd"
```

---

## Exercise 6: Using Roles

Learn to organize playbooks using roles.

### 6.1 Create a New Role

```bash
mkdir -p roles/nginx/{tasks,handlers,templates,defaults}
```

### 6.2 Define Role Tasks

Create `roles/nginx/tasks/main.yml`:

```yaml
---
- name: Install Nginx
  package:
    name: nginx
    state: present

- name: Start Nginx service
  service:
    name: nginx
    state: started
    enabled: yes

- name: Deploy index page
  template:
    src: index.html.j2
    dest: /usr/share/nginx/html/index.html
  notify: reload nginx
```

### 6.3 Create Handler

Create `roles/nginx/handlers/main.yml`:

```yaml
---
- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

### 6.4 Use the Role

Create `playbooks/nginx.yml`:

```yaml
---
- name: Setup Nginx
  hosts: webservers
  become: true
  
  roles:
    - nginx
```

---

## Exercise 7: Conditionals and Loops

Learn to use when statements and loops.

### 7.1 Conditionals

Create `playbooks/conditionals.yml`:

```yaml
---
- name: Conditional Tasks
  hosts: all
  become: true
  
  tasks:
    - name: Install Apache on Debian systems
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install Apache on RedHat systems
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Only run on production servers
      debug:
        msg: "This is a production server"
      when: inventory_hostname in groups['production']
```

### 7.2 Loops

Create `playbooks/loops.yml`:

```yaml
---
- name: Loop Examples
  hosts: all
  become: true
  
  tasks:
    - name: Install multiple packages
      package:
        name: "{{ item }}"
        state: present
      loop:
        - vim
        - git
        - htop
    
    - name: Create multiple users
      user:
        name: "{{ item }}"
        state: present
      loop:
        - alice
        - bob
        - charlie
```

---

## Exercise 8: Error Handling

Learn to handle errors gracefully.

### 8.1 Ignore Errors

```yaml
---
- name: Error Handling
  hosts: all
  
  tasks:
    - name: This might fail
      command: /bin/false
      ignore_errors: yes
    
    - name: This will run anyway
      debug:
        msg: "Previous task failed but we continued"
```

### 8.2 Failed When

```yaml
---
- name: Custom Failure Conditions
  hosts: all
  
  tasks:
    - name: Check disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      failed_when: disk_usage.stdout|int > 80
```

---

## Exercise 9: Run the Workshop Playbooks

Now run the provided workshop playbooks.

### 9.1 Common Configuration

```bash
# Check mode first
ansible-playbook playbooks/common.yml --check

# Execute
ansible-playbook playbooks/common.yml
```

### 9.2 Web Server Setup

```bash
ansible-playbook playbooks/webserver.yml
```

### 9.3 Full Site Deployment

```bash
ansible-playbook playbooks/site.yml
```

### 9.4 Verify Web Server

```bash
# Check if web server is running
ansible webservers -m service -a "name=apache2 state=started" --become

# Test HTTP response
curl http://<webserver-ip>/
```

---

## Exercise 10: Advanced Topics

### 10.1 Vault for Secrets

Encrypt sensitive data:

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Use in playbook
ansible-playbook playbooks/site.yml --ask-vault-pass
```

### 10.2 Tags

Run specific parts of a playbook:

```yaml
---
- name: Tagged Tasks
  hosts: all
  
  tasks:
    - name: Install packages
      package:
        name: vim
        state: present
      tags: packages
    
    - name: Configure firewall
      firewalld:
        service: http
        state: enabled
      tags: firewall
```

Run specific tags:
```bash
ansible-playbook playbooks/site.yml --tags packages
ansible-playbook playbooks/site.yml --skip-tags firewall
```

---

## Challenge Exercises

### Challenge 1: Dynamic Inventory
Create a dynamic inventory script or use cloud provider inventory plugins.

### Challenge 2: Custom Module
Write a simple custom module in Python.

### Challenge 3: Role Dependencies
Create a role that depends on other roles.

### Challenge 4: CI/CD Pipeline
Integrate Ansible with a CI/CD tool (Jenkins, GitLab CI, etc.).

### Challenge 5: Ansible Tower/AWX
Install and configure Ansible Tower or AWX for GUI-based automation.

---

## Best Practices Checklist

- [ ] Use roles for organization
- [ ] Keep playbooks idempotent
- [ ] Use variables for flexibility
- [ ] Implement error handling
- [ ] Use handlers for service management
- [ ] Tag tasks appropriately
- [ ] Use version control (Git)
- [ ] Test with --check mode first
- [ ] Use ansible-lint for best practices
- [ ] Document your playbooks
- [ ] Use vault for sensitive data
- [ ] Keep inventory organized

---

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Pre-built roles
- [Ansible Lint](https://ansible-lint.readthedocs.io/) - Linting tool
- [Molecule](https://molecule.readthedocs.io/) - Testing framework
- [Red Hat Training](https://www.redhat.com/en/services/training/do007-ansible-essentials-simplicity-automation-technical-overview)

---

## Completion

Congratulations! You've completed the Ansible Workshop exercises. You should now have a solid foundation in:

- Running ad-hoc commands
- Writing playbooks
- Using variables and templates
- Creating and using roles
- Implementing handlers
- Working with conditionals and loops
- Error handling
- Best practices

Continue practicing and building more complex automation scenarios!
