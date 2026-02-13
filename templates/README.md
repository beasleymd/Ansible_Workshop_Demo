# Templates Directory

This directory is for Jinja2 template files that will be processed and deployed to target hosts.

## Purpose

Store template files that use Jinja2 syntax to create dynamic configurations based on variables, such as:
- Configuration files with host-specific settings
- Service configuration templates
- Application config files
- Dynamic scripts

## Usage

Reference templates in playbooks using the `template` module:

```yaml
- name: Deploy configuration from template
  template:
    src: myconfig.j2
    dest: /etc/myapp/config.conf
    owner: root
    group: root
    mode: '0644'
```

## Template Syntax

Templates use Jinja2 syntax for variable substitution:

```jinja
# Example template: nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ inventory_hostname }};
    
    location / {
        root {{ document_root }};
    }
}
```

## Organization

Consider organizing templates by:
- Role or service
- Application
- File type

Example structure:
```
templates/
├── nginx/
│   ├── nginx.conf.j2
│   └── site.conf.j2
├── apache/
│   └── httpd.conf.j2
└── app/
    └── config.ini.j2
```

## Best Practices

- Use `.j2` extension for template files
- Add comments to explain complex logic
- Use filters for data manipulation
- Keep templates readable and maintainable
- Test templates with different variable values
