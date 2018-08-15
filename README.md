# NGINX

## Overview
It is very hard to write a generic role for the ansible configuration.

This role suppose to use includes so you can write your own piece ot the configuration files
and include them to the main config.

It's ment to be formore experienced users, sinde of theshell itgenerates non vlaid configuration.


## Requirements

- ansible: 2.1
- role: hudecof.nginx-repo

## Role Variables

### Distribution specific

OS specific variables are defined in `vars/os-<distribution>.yml`, but could be still overwritten.

### Global

These variables are in `defaults/main.yml`. I will list only subset of them.

`nginx_packages` list of packages to install/remove and is derived from OS specific variables

`nginx_service` is name o the service and is derived from the OS specific variables.

`nginx_user` and `nginx_group` are user and group name to run service under

`nginx_config_backup` is boolean. If set, backup file is created for each changed configuration file.

`nginx_dir_prefix` is directory where the confuguration file templaes will be searched for. I do not provide any sample config file. I used to set it `{{ playbook_dir}}/files/nginx`


### Enable modules

`nginx_server_http`, `nginx_server_mail` and `nginx_server_stream` are booleans to be set when want to run the modele. In most cases you set the `nginx_server_http`

### Modules configuration files

`nginx_core_conf`, `nginx_http_conf`, `nginx_http_sites`, `nginx_mail_conf`, `nginx_mail_sites`, `nginx_stream_conf`, `nginx_stream_sites`, `nginx_main_conf` are list of configuraion files, where each item looks like

    - {'file': '<filename>', 'enabled': True/False }
  
The **file** will be copies in coresponding ``<module>.d/<conf/site>-available`` directory. If config file is enabled, the symlink to  `<module>.d/<conf/site>-enabled` is created.


## Example configuration

You have to create all configuration from scratch. None of the distribution files are used by default.

### Ansible configuration variables
```
nginx_server_http: True
nginx_dir_prefix: "{{ playbook_dir }}/files/nginx"

nginx_main_conf_base:
  - { 'name': 'events.conf', 'enabled': True }
nginx_main_conf_host: []
nginx_main_conf: "{{ nginx_main_conf_base|union(nginx_main_conf_host) }}"

nginx_http_conf_base:
  - { 'name': 'basic.conf', 'enabled': True }
  - { 'name': 'gzip.conf', 'enabled': True }
  - { 'name': 'mime-types.conf', 'enabled': True }
  - { 'name': 'ssl.conf', 'enabled': True }
  - { 'name': 'log.conf', 'enabled': True }
  - { 'name': 'map.conf', 'enabled': True }
nginx_http_conf_host: []
nginx_http_conf: "{{ nginx_http_conf_base|union(nginx_http_conf_host) }}"

nginx_http_sites_base:
  - { 'name': 'nginx_status', 'enabled': True }
  - { 'name': 'ssl_redirect', 'enabled': True }
nginx_http_sites_host: []
nginx_http_sites: "{{ nginx_http_sites_base|union(nginx_http_sites_host) }}"
```

### Directory structure
```
{{ playbook_dir }}/files/nginx/http-conf
{{ playbook_dir }}/files/nginx/http-site
{{ playbook_dir }}/files/nginx/main-conf
```

### Configuration file contect

For example the file `{{ playbook_dir }}/files/nginx/http-site/ssl_redirect` looks like

```
server {
	listen 80 default_server;
	listen [::]:80 default_server;

        location / {
                return 301 https://$host$request_uri;
        }
}
```