<p class="has-line-data" data-line-start="0" data-line-end="3">Работаю на Windows 10 PRO<br>
Создал в Hyper-V 2 виртуальных машины CentOS 7.0<br>
Ansible уставлен в WSL 2.0 (Ubuntu)</p>
<p class="has-line-data" data-line-start="4" data-line-end="5">Изменил файл prod.yml</p>
<pre><code>---
  elasticsearch:
    hosts:
      localhost:
          ansible_host: 192.168.10.64
          ansible_connection: ssh
          ansible_user: root
  kibana:
    hosts:
      localhost:
          ansible_host: 192.168.10.68
          ansible_connection: ssh
          ansible_user: root
</code></pre>
<p class="has-line-data" data-line-start="19" data-line-end="20">Добавил фаил переменных /kibana/vars.yml</p>
<pre><code>---
kibana_version: &quot;7.10.1&quot;
kibana_home: &quot;/opt/kibana/{{ kibana_version }}&quot;
</code></pre>
<p class="has-line-data" data-line-start="25" data-line-end="28">Добавил файл в Temlpate kbn.sh.j2<br>
# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.<br>
#!/usr/bin/env bash</p>
<pre><code>export ES_HOME={{ kibana_home }}
export PATH=$PATH:$ES_HOME/bin
</code></pre>
<p class="has-line-data" data-line-start="32" data-line-end="68">Добавил блок в site.yml<br>
- name: Install Kibana<br>
hosts: kibana<br>
tasks:<br>
- name: Upload tar.gz Kibana from remote URL<br>
get_url:<br>
url: &quot;<a href="https://artifacts.elastic.co/downloads/kibana/kibana-">https://artifacts.elastic.co/downloads/kibana/kibana-</a>{{ kibana_version }}-linux-x86_64.tar.gz&quot;<br>
dest: “/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz”<br>
mode: 0755<br>
timeout: 60<br>
force: true<br>
validate_certs: false<br>
register: get_kibana<br>
until: get_kibana is succeeded<br>
tags: kibana<br>
- name: Create directrory for Kibana<br>
file:<br>
state: directory<br>
path: “{{ kibana_home }}”<br>
tags: kibana<br>
- name: Extract kibana in the installation directory<br>
become: true<br>
unarchive:<br>
copy: false<br>
src: “/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz”<br>
dest: “{{ kibana_home }}”<br>
extra_opts: [–strip-components=1]<br>
creates: “{{ kibana_home }}/bin/kibana”<br>
tags:<br>
- kibana<br>
- name: Set environment Kibana<br>
become: true<br>
template:<br>
src: templates/kbn.sh.j2<br>
dest: /etc/profile.d/kbn.sh<br>
tags: kibana</p>
<p class="has-line-data" data-line-start="70" data-line-end="71">Попробуйте запустить playbook на этом окружении с флагом --check</p>
<pre><code>mike@HOMEDX79SR:~/ans82/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check
PLAY [Install Java] ******
TASK [Gathering Facts] **************************
ok: [localhost]
TASK [Set facts for Java 11 vars] *************************************************
ok: [localhost]
TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************
ok: [localhost]
TASK [Ensure installation dir exists] *******************************************************************
ok: [localhost]
TASK [Extract java in the installation directory] **********************************************************************
skipping: [localhost]
TASK [Export environment variables] ********************************************
ok: [localhost]
PLAY [Install Elasticsearch] ********************************************
TASK [Gathering Facts] *****************************************************
ok: [localhost]
TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************
changed: [localhost]
TASK [Create directrory for Elasticsearch] **********************************************
ok: [localhost]
TASK [Extract Elasticsearch in the installation directory] *************************************************************
skipping: [localhost]
TASK [Set environment Elastic] ************************************************
ok: [localhost]
PLAY [Install Kibana] *********************************************************
TASK [Gathering Facts] **********************************************************
ok: [localhost]
TASK [Upload tar.gz Kibana from remote URL] ***********************************************************************
changed: [localhost]
TASK [Create directrory for Kibana] *************************************************************************
ok: [localhost]
TASK [Extract kibana in the installation directory] ********************************************************************
skipping: [localhost]
TASK [Set environment Kibana] ***************************************************************************
ok: [localhost]
PLAY RECAP ***************************************************************************
localhost                  : ok=13   changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
</code></pre>
<p class="has-line-data" data-line-start="111" data-line-end="112">Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.</p>
<pre><code>mike@HOMEDX79SR:~/ans82/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] ************************************************
TASK [Gathering Facts] ************************************************************************
ok: [localhost]
TASK [Set facts for Java 11 vars] ****************************************************************************
ok: [localhost]
TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************
ok: [localhost]
TASK [Ensure installation dir exists] *****************************************************************
ok: [localhost]
TASK [Extract java in the installation directory] **********************************************************************
skipping: [localhost]
TASK [Export environment variables] ************************************************************************************
ok: [localhost]
PLAY [Install Elasticsearch] ***************************************************************
TASK [Gathering Facts] ***********************************************************
ok: [localhost]
TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************
ok: [localhost]
TASK [Create directrory for Elasticsearch] *****************************************************************************
ok: [localhost]
TASK [Extract Elasticsearch in the installation directory] *************************************************************
skipping: [localhost]
TASK [Set environment Elastic] ********************************************
ok: [localhost]
PLAY [Install Kibana] ************************************************************************
TASK [Gathering Facts] **************************************************************************
ok: [localhost]
TASK [Upload tar.gz Kibana from remote URL] ****************************************************************************
ok: [localhost]
TASK [Create directrory for Kibana] ************************************************************************************
ok: [localhost]
TASK [Extract kibana in the installation directory] ********************************************************************
skipping: [localhost]
TASK [Set environment Kibana] *********************************************************************
ok: [localhost]
PLAY RECAP *******************************************************************
localhost                  : ok=13   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
</code></pre>
