# NGINX

## Overview
It is very hard to write a generic role for the ansible configuration.

This role suppose to use includes so you can write your own piece ot the configuration files
and include them to the main config.

The sites configuration files are just plain text files, but I'm using power of the jinja2 template engine
to keep the config files KISS.

## Requirements
This role requires Ansible 1.4 or higher and platform requirements are listed in the metadata file.

## Role Variables

### Distribution specific
These variables are in `var/Debian.yml` and `vars/RedHat.yml`.
They are included based on the distribution of the target system.

### Global
These variables are in `defaults/main.yml`.


`nginx_sites` is the list of the siets to be generated to the `{{ nging_dir_etc }}/sites-available`. 

    nginx_sites:
     - name: test
       state: enabled


`nginx_conf_d` is the list of the file to be copies and enabled

    nginx_conf_d:
     - name: basic.conf
       state: enabled 


`nginx_conf_main_include` is the list of the files to be included in the main configuration file. The **http** sesction
is also include file.

    nginx_conf_main_include:
     - modules/http.conf


`nginx_file_extra` is the list of the files to be copies to `{{ nginx_dir_etc }}` expept fthe main configurtion file.

    nginx_file_extra:
     - src: modules/http.conf
       dest: modules/http.conf
     - src: modules/mail.conf
       dest: modules/mail.conf


`nginx_dir_extra` list of the extra directoris to be created. For example for the mail module.

    nginx_dir_extra:
     - name: mail.d
       owner: root
       group: root
       mode: '0750'

## Site configuration
In the **templates/sites/tmpl** I have included some templates which could be helpfull to create site configs.

    upstream upstream_site_test {
      ip_hash;
      server 192.168.10.10:80;
      server 192.168.10.10:80;
    }

    server {
      listen *:443;
      server_name	test.test.sk;

      {% include 'tmpl/log_upstream.conf' %}
      {% include 'tmpl/ssl.conf' %}
      {% include 'tmpl/proxy_headers.conf' %}

      location / {
        proxy_redirect     off;
        proxy_set_header Host $host;
        proxy_pass http://upstream_site_test;
      }
    }

