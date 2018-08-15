# test_Ansible
# Ansible
# Hi Coders
# Richard here.I love DevOps and the beach life!

apache.yaml

---

- hosts: apache
  sudo: yes
  vars:
    http_port: 80
    domain: example.com
  tasks:
    - name: install apache2
      apt: name=apache2 update_cache=yes state=latest

    - name: enabled mod_rewrite
      apache2_module: name=rewrite state=present
      notify:
        - restart apache2

    - name: apache2 listen on port {{ http_port }}
      lineinfile: dest=/etc/apache2/ports.conf regexp="^Listen " line="Listen {{ http_port }}" state=present
      notify:
        - restart apache2

    - name: apache2 virtualhost on port {{ http_port }}
      lineinfile: dest=/etc/apache2/sites-available/000-default.conf regexp="^<VirtualHost \*:" line="<VirtualHost *:{{ http_port }}>"
      notify:
        - restart apache2

    - name: create virtual host file
      template: src=virtualhost.conf dest=/etc/apache2/sites-available/{{ domain }}.conf

    - name: a2ensite {{ domain }}
      command: a2ensite {{ domain }}
      args:
        creates: /etc/apache2/sites-enabled/{{ domain }}.conf
      notify:
        - restart apache2

  handlers:
    - name: restart apache2
      service: name=apache2 state=restarted





virtualhost.conf


<VirtualHost *:{{ http_port }}>
    ServerAdmin webmaster@{{ domain }}
    ServerName {{ domain }}
    ServerAlias www.{{ domain }}
    DocumentRoot /var/www/{{ domain }}
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
