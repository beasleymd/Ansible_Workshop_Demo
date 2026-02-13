# Host Variables Directory

This directory contains host-specific variables.

## Purpose

Define variables that apply to individual hosts in your inventory.

## Usage

Create a YAML file named after each host:

```
host_vars/
├── web1.example.com.yml
├── web2.example.com.yml
└── db1.example.com.yml
```

## Example

File: `host_vars/web1.example.com.yml`

```yaml
---
# Host-specific variables for web1.example.com
http_port: 8080
max_connections: 100
custom_setting: value
```

## Precedence

Host variables have higher precedence than group variables, allowing you to override group settings for specific hosts.

## Best Practices

- Use host_vars sparingly; prefer group_vars when possible
- Only define host-specific overrides here
- Keep variable names consistent across hosts
- Document any unusual host-specific configurations
