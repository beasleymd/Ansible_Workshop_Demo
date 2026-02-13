# Ansible Workshop Demo

This repository contains Ansible playbooks and content for both **Linux server automation** and **Cisco network device automation**. It provides comprehensive examples and best practices for learning and demonstrating Ansible automation across multiple platforms.

## 🎯 Supported Platforms

### Cisco Network Devices
- **4x Cisco C8000V Routers** (rtr1, rtr2, rtr3, rtr4)
- Model: Cisco C8000V (VXE)
- IOS Version: Cisco IOS XE Software, Version 17.14.01a
- Capabilities: Configuration management, backups, validation, facts gathering

### Linux Servers
- Web servers, database servers, load balancers
- Debian/Ubuntu and RedHat/CentOS support
- System administration and application deployment

## Repository Structure

```
Ansible_Workshop_Demo/
├── ansible.cfg              # Ansible configuration
├── requirements.txt         # Python dependencies
├── requirements.yml         # Ansible Galaxy collections
├── inventory/              # Inventory files
│   └── hosts              # Main inventory (routers + servers)
├── configs/               # Network configuration files
│   ├── golden_configs/   # Golden/reference configs
│   └── backup_configs/   # Configuration backups
├── playbooks/             # Ansible playbooks
│   ├── cisco_backup_config.yml       # Backup router configs
│   ├── cisco_deploy_golden_config.yml # Deploy golden configs
│   ├── cisco_gather_facts.yml        # Gather router information
│   ├── cisco_validate_config.yml     # Validate configurations
│   ├── site.yml          # Master playbook (Linux)
│   ├── common.yml        # Common configuration (Linux)
│   └── webserver.yml     # Web server setup (Linux)
├── roles/                # Reusable roles
│   ├── cisco_backup/    # Cisco config backup
│   ├── cisco_facts/     # Cisco facts gathering
│   ├── common/          # Common Linux role
│   └── webserver/       # Web server role
├── group_vars/          # Group-specific variables
│   ├── all.yml         # Global variables
│   └── routers.yml     # Cisco router variables
├── host_vars/           # Host-specific variables
├── files/               # Static files
├── templates/           # Jinja2 templates
└── docs/                # Documentation
```

## Prerequisites

### For All Automation
- Ansible 2.9 or later
- Python 3.6 or later

### For Cisco Network Automation
- Ansible collections: `cisco.ios`, `ansible.netcommon`
- Network connectivity to Cisco routers
- Valid credentials for router access
- Enable mode password

### For Linux Server Automation
- SSH access to target hosts
- Sudo privileges on target hosts

## Quick Start

### 1. Install Dependencies

```bash
# Install Python dependencies
pip install -r requirements.txt

# Install Ansible Galaxy collections (for network automation)
ansible-galaxy collection install -r requirements.yml
```

### 2. Configure Inventory

#### For Cisco Routers
Edit `inventory/hosts` to set IP addresses for your routers:

```ini
[routers]
rtr1 ansible_host=192.168.1.101
rtr2 ansible_host=192.168.1.102
rtr3 ansible_host=192.168.1.103
rtr4 ansible_host=192.168.1.104
```

Update credentials in `group_vars/routers.yml` or use Ansible Vault for production.

#### For Linux Servers
Edit `inventory/hosts` to add your Linux hosts:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
```

### 3. Run Playbooks

#### Cisco Network Automation
```bash
# Gather facts from all routers
ansible-playbook playbooks/cisco_gather_facts.yml

# Backup router configurations
ansible-playbook playbooks/cisco_backup_config.yml

# Deploy golden configurations
ansible-playbook playbooks/cisco_deploy_golden_config.yml

# Validate configurations against golden configs
ansible-playbook playbooks/cisco_validate_config.yml

# Run on specific router only
ansible-playbook playbooks/cisco_backup_config.yml --limit rtr1
```

#### Linux Server Automation
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

### Cisco Network Automation Playbooks

#### cisco_gather_facts.yml
Connects to Cisco routers and gathers comprehensive system information:
- Hostname, model, serial number
- IOS version and image
- Interface details
- Generates facts reports

#### cisco_backup_config.yml
Creates timestamped backups of router configurations:
- Backs up running configuration
- Creates metadata files
- Stores in `configs/backup_configs/`
- Includes backup summary report

#### cisco_deploy_golden_config.yml
Deploys golden (reference) configurations to routers:
- Backs up current config before deployment
- Deploys from `configs/golden_configs/`
- Supports merge or replace mode
- Validates deployment success

#### cisco_validate_config.yml
Validates running configurations against golden configs:
- Compares current vs. reference configurations
- Generates validation reports
- Identifies configuration drift
- Provides remediation guidance

### Linux Server Automation Playbooks

#### site.yml
Master playbook that orchestrates all other playbooks.

#### common.yml
Applies baseline configuration to all hosts:
- Updates package cache
- Installs common utilities
- Configures timezone
- Sets up ansible user and sudo access

#### webserver.yml
Configures Apache web servers:
- Installs Apache (httpd/apache2)
- Deploys custom web page
- Configures firewall rules
- Manages service state

## Available Roles

### Cisco Network Roles

#### cisco_backup
Handles configuration backup operations for Cisco routers:
- Automated backup with timestamps
- Metadata generation
- Retention management

#### cisco_facts
Gathers and reports detailed information from Cisco routers:
- Hardware details
- Software versions
- Interface information
- Neighbor discovery

### Linux Server Roles

#### common
Provides common configuration tasks across all hosts.

#### webserver
Installs and configures Apache web server with OS-specific variables.

## Cisco C8000V Router Details

This repository is configured to manage **4 Cisco C8000V virtual routers**:

### Hardware Specifications
- **Model**: Cisco C8000V (VXE)
- **Processor**: cisco C8000V (VXE) processor (revision VXE)
- **Memory**: 1955292K/3075K bytes
- **Storage**: 11526144K bytes virtual hard disk at bootflash
- **Interfaces**: 1 Gigabit Ethernet interface

### Software Specifications
- **IOS Software**: Cisco IOS XE Software
- **Version**: 17.14.01a
- **Image**: Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M)
- **Release**: RELEASE SOFTWARE (fc1)
- **License**: Smart Licensing Using Policy

### Router Inventory
| Hostname | Management IP | Role | Config File |
|----------|--------------|------|-------------|
| rtr1 | 192.168.1.101 (example) | Router | rtr1_golden_config.txt |
| rtr2 | 192.168.1.102 (example) | Router | rtr2_golden_config.txt |
| rtr3 | 192.168.1.103 (example) | Router | rtr3_golden_config.txt |
| rtr4 | 192.168.1.104 (example) | Router | rtr4_golden_config.txt |

### Golden Configuration Files
Golden configuration files are stored in `configs/golden_configs/` and should be replaced with your actual router configurations from:
- `Orig_Golden_Configs_rtr1_02-12-2026.rtf`
- `Orig_Golden_Configs_rtr2_02-12-2026.rtf`
- `Orig_Golden_Configs_rtr3_02-12-2026.rtf`
- `Orig_Golden_Configs_rtr4_02-12-2026.rtf`

**Note**: Convert RTF files to plain text format before use.

## Workshop Content

See the `docs/` directory for:
- **SETUP.md** - Detailed setup instructions
- **EXERCISES.md** - Workshop exercises and labs

## Best Practices

This repository demonstrates Ansible best practices:

### General Best Practices
- ✅ Role-based organization
- ✅ Separate inventory and variables
- ✅ Idempotent tasks
- ✅ Group and host variables
- ✅ Clear naming conventions
- ✅ Comprehensive documentation

### Network Automation Best Practices
- ✅ Backup before configuration changes
- ✅ Golden configuration management
- ✅ Configuration validation and compliance
- ✅ Network-specific connection plugins (network_cli)
- ✅ Secure credential management (Ansible Vault ready)
- ✅ Timestamped backups and reports

### Linux Automation Best Practices
- ✅ OS-agnostic playbooks using conditionals
- ✅ Handlers for service management
- ✅ Jinja2 templates for configuration files

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