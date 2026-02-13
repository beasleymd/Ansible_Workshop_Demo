# Ansible Workshop Demo

This repository contains Ansible playbooks and content from the Red Hat Automation Workshops. It provides a comprehensive set of examples and best practices for learning and demonstrating Ansible automation.

## Repository Structure

```
Ansible_Workshop_Demo/
├── ansible.cfg              # Ansible configuration
├── requirements.txt         # Python/Ansible dependencies
├── inventory/              # Inventory files
│   ├── hosts              # Main inventory file
│   └── group_vars/        # Group variables
├── playbooks/             # Ansible playbooks
│   ├── site.yml          # Master playbook
│   ├── common.yml        # Common configuration
│   └── webserver.yml     # Web server setup
├── roles/                # Reusable roles
│   ├── common/          # Common role
│   └── webserver/       # Web server role
├── group_vars/          # Group-specific variables
├── host_vars/           # Host-specific variables
├── files/               # Static files
├── templates/           # Jinja2 templates
└── docs/                # Documentation
```

## Prerequisites

- Ansible 2.9 or later
- Python 3.6 or later
- SSH access to target hosts
- Sudo privileges on target hosts

## Quick Start

### 1. Install Dependencies

```bash
# Install Python dependencies
pip install -r requirements.txt

# Or install Ansible directly
pip install ansible
```

### 2. Configure Inventory

Edit `inventory/hosts` to add your target hosts:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
```

### 3. Run Playbooks

```bash
# Run the master playbook
ansible-playbook playbooks/site.yml

# Run a specific playbook
ansible-playbook playbooks/webserver.yml

# Run with check mode (dry-run)
ansible-playbook playbooks/site.yml --check

# Run with verbose output
ansible-playbook playbooks/site.yml -v
```

## Available Playbooks

### site.yml
Master playbook that orchestrates all other playbooks.

### common.yml
Applies baseline configuration to all hosts:
- Updates package cache
- Installs common utilities
- Configures timezone
- Sets up ansible user and sudo access

### webserver.yml
Configures Apache web servers:
- Installs Apache (httpd/apache2)
- Deploys custom web page
- Configures firewall rules
- Manages service state

## Available Roles

### common
Provides common configuration tasks across all hosts.

### webserver
Installs and configures Apache web server with OS-specific variables.

## Workshop Content

See the `docs/` directory for:
- **SETUP.md** - Detailed setup instructions
- **EXERCISES.md** - Workshop exercises and labs

## Best Practices

This repository demonstrates Ansible best practices:
- ✅ Role-based organization
- ✅ Separate inventory and variables
- ✅ OS-agnostic playbooks using conditionals
- ✅ Idempotent tasks
- ✅ Handlers for service management
- ✅ Jinja2 templates for configuration files
- ✅ Group and host variables
- ✅ Clear naming conventions

## Customization

### Adding New Hosts
Edit `inventory/hosts` to add your servers.

### Modifying Variables
- Global variables: `group_vars/all.yml`
- Group-specific: `group_vars/<group_name>.yml`
- Host-specific: `host_vars/<hostname>.yml`

### Creating New Roles
```bash
# Create a new role structure
mkdir -p roles/myrole/{tasks,handlers,defaults,vars,templates,files}
```

## Testing

### Syntax Check
```bash
ansible-playbook playbooks/site.yml --syntax-check
```

### Lint Playbooks
```bash
ansible-lint playbooks/*.yml
```

### Check Mode (Dry Run)
```bash
ansible-playbook playbooks/site.yml --check
```

## Troubleshooting

### Connection Issues
- Verify SSH connectivity: `ssh ansible@<host>`
- Check inventory file syntax
- Verify ansible user has sudo privileges

### Playbook Failures
- Run with verbose output: `-v`, `-vv`, or `-vvv`
- Check ansible.log for details
- Verify variables are set correctly

## Contributing

This is a workshop demo repository. Feel free to:
- Add new playbooks
- Create additional roles
- Improve documentation
- Add workshop exercises

## Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## License

This repository is for educational and demonstration purposes.

## Contact

For questions or feedback about this workshop demo, please open an issue in the repository.