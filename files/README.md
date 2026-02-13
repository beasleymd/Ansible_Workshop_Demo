# Files Directory

This directory is for static files that will be copied to target hosts.

## Purpose

Store files that need to be distributed to managed hosts without modification, such as:
- Configuration files
- Scripts
- Certificates
- Static content
- Binary files

## Usage

Reference files in playbooks using the `copy` module:

```yaml
- name: Copy file to remote host
  copy:
    src: myfile.txt
    dest: /path/to/destination/myfile.txt
    owner: root
    group: root
    mode: '0644'
```

## Organization

Consider organizing files by:
- Role or purpose
- Environment (dev, staging, prod)
- Application or service

Example structure:
```
files/
├── scripts/
│   └── backup.sh
├── configs/
│   └── app.conf
└── ssl/
    ├── cert.pem
    └── key.pem
```
