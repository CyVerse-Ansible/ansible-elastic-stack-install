ansible-elastic-stack-install
=============================
[![Build Status](https://travis-ci.org/CyVerse-Ansible/ansible-elastic-stack-install.svg?branch=master)](https://travis-ci.org/CyVerse-Ansible/ansible-elastic-stack-install)

[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-elastic--stack--install-blue.svg)](https://galaxy.ansible.com/CyVerse-Ansible/ansible-elastic-stack-install/)

Performs basic installation and configuration of Elastic-Stack components, in accordance with Elastic's established packaging conventions.
This role is a "clone-and-own" of files from the [elastic/ansible-elasticsearch][ansible-elasticsearch] GH repo.

Beginning with version `5.x`, all products/packages in  "Elastic-Stack" product line are now released "lock-step". 
Furthermore, there are many consistencies which make it possible to create a "general purpose" Ansible role which 
allows for the generic installation and configuration of "Elastic-Stack" products.

The intent of this role is to support both archive-based and repository-based installation of _Elastic-Stack_ products. 
Currently, only repository-based installations are tested.

Role Variables
==============

User-Required Variables
-----------------------

| Variable                |        Required            | Default        | Description    |
|-------------------------|----------------------------|----------------|----------------|
| `ES_PACKAGE`            | yes                        |      none      | The Elastic-Stack package to install | 
| `es_instance_name`      | no                         |      none      | See the `"Instances"` section. | 
| `ES_CONFIG_CONTENTS`    | yes (w/ `es_instance_name`)|      none      | Variable which contains the contents of the Elastic-Stack package's `*.yml` file. | 
| `INIT_FILE_TEMPLATE_SRC`| yes (w/ `es_instance_name`)|      none      | Path to instance-specific init file Jinja template. When `es_instance_name` is set, this variable is required on non-_systemd_ hosts.|
| `SYSD_FILE_TEMPLATE_SRC`| yes (w/ `es_instance_name`)|      none      | Path to instance-specifica systemd service file Jinja template. When `es_instance_name` is set, this variable is required on _systemd_-capable hosts. | 

Default Directory Layout
------------------------

| ES Package | `es_home` | Bin | `conf_dir` | `data_dir` | `log_dir` |  
|------------|------|-----|---------------|------|------|
| [elasticsearch][elasticsearch-deb-dir-layout] | `/usr/share/elasticsearch/` | `/usr/share/elasticsearch/bin/` | `/etc/elasticsearch/elasticsearch.yml` | `/var/lib/elasticsearch/` | `/var/log/elasticsearch/` |
| [kibana][kibana-deb-dir-layout] | `/usr/share/kibana/`    | `/usr/share/kibana/bin/`   | `/etc/kibana/kibana.yml`       | `/var/lib/kibana/`         | N/A |
| [beat][packetbeat-dir-layout]  | `/usr/share/packetbeat/`     | `/usr/share/packetbeat/bin/`    | `/etc/packetbeat/packetbeat.yml`         | `/var/lib/packetbeat/`          | `/var/log/packetbeat/` |
| [logstash][logstash-dir-layout] | `/usr/share/logstash/`  | `/usr/share/logstash/bin/` | `/etc/logstash/logstash.yml`   | N/A | `/var/log/logstash/` |

The table below shows the default folder/file naming patterns:

| Directory/File | Default Value |
|----------------|---------------|
| `{{es_home}}`  | `/usr/share/{{ES_PACKAGE}}/` | 
| `{{conf_dir}}` | `/etc/{{ES_PACKAGE}}/{{es_instance_name}}/` | 
| `{{data_dir}}` | `/var/lib/{{ES_PACKAGE}}/`      | 
| `{{log_dir}}`  | `/var/log/{{ES_PACKAGE}}/` |
| `{{pid_dir}}`  | `/var/run/{{ES_PACKAGE}}/` |
| `{{instance_default_file}}`  | `{{default_file}}` |
| `{{instance_init_script}}`  | `{{init_script}}` |
| `{{instance_sysd_script}}`  | `{{sysd_script}}` |


Configuration/Settings File
---------------------------
Each package has a corresponding `*.yml` file. This role supports the generation of the package's configuration file via the `ES_CONFIG_CONTENTS` variable.

"Instances"
-----------
This role is based on the [elastic/ansible-elasticsearch][ansible-elasticsearch] Ansible role, which requires that the `es_instance_name` variable be defined.
Within this role, the `es_instance_name` variable is optional. 
The default behavior of this role, when `es_instance_name` is undefined, is to install the given `ES_PACKAGE` with the default directory layout.
However, when `es_instance_name` is specified the directory layout changes.

Instance-specific folder and filenames associated with this role are primarily controlled with the following variables:

| Variable | Default Value |
|----------|---------------|
|`instance_dir_suffix`|`{{inventory_hostname}}-{{es_instance_name}}`|
|`instance_sys_file_name`|`{{ES_PACKAGE}}_{{es_instance_name}}`|

The following table illustrates how these variables are applied:

| Directory/File              | Default Value |
|-----------------------------|---------------|
| `{{es_user}}`               | `{{ES_PACKAGE}}`| 
| `{{es_group}}`              | `{{ES_PACKAGE}}`| 
| `{{es_home}}`               | `/usr/share/{{ES_PACKAGE}}/` | 
| `{{conf_dir}}`              | `/etc/{{ES_PACKAGE}}/{{es_instance_name}}/` | 
| `{{data_dir}}`              | `/var/lib/{{ES_PACKAGE}}/{{instance_dir_suffix}}/`      | 
| `{{log_dir}}`               | `/var/log/{{ES_PACKAGE}}/{{instance_dir_suffix}}/` |
| `{{pid_dir}}`               | `/var/run/{{ES_PACKAGE}}/{{instance_dir_suffix}}/` |
| `{{instance_default_file}}` | `{{default_file | dirname}}/{{instance_sys_file_name}}` |
| `{{instance_init_script}}`  | `{{init_script | dirname}}/{{instance_sys_file_name}}` |
| `{{instance_sysd_script}}`  | `{{sysd_script | dirname}}/{{instance_sys_file_name}}.service` |

SysV INIT vs Systemd
--------------------
This role also allows for the templating of the `SysV INIT` or `Systemd` service files. This is a point where the Elastic-Stack packages are not consistent.
* Systemd
  * On *all* `Systemd` systems, _Elasticsearch_ places its service file is `/usr/lib/systemd/system`.
  * _Beats_ will respect the host OS's default directory (i.e. `/lib/systemd/system` on Debian, `/usr/lib/systemd/system` on CentOS). 
  * On *all* `Systemd` systems, _Kibana_ places the service file in `/etc/systemd/system`.

Due to this inconsistency, this role will remove the default service file from *all* of the following locations when templating the _systemd_ service file:
* `/usr/lib/systemd/system/{{ES_PACKAGE}}.service`
* `/lib/systemd/system/{{ES_PACKAGE}}.service`
* `/etc/systemd/system/{{ES_PACKAGE}}.service`

When a valid `SYSD_FILE_TEMPLATE_SRC` is given, it will be installed in the following location: `/usr/lib/systemd/system/`


Optional Variables
------------------

| Variable                |        Required            | Default        | Description    |
|-------------------------|----------------------------|----------------|----------------|
| `es_major_version`      | no                         | "5.x"          | | 
| `es_version`            | no                         | "5.2.0"        | | 
| `es_version_lock`       | no                         | false          | | 
| `es_use_repository`     | no                         | true           | | 
| `es_start_service`      | no                         | true           | | 
| `es_restart_on_change`  | no                         | true           | | 
| `update_java`           | no                         | false          | | 
| `es_update_package`     | no                         | true           | | 

Dependencies
============

None.

Example Playbooks
=================

Refer to [.travis.yml](.travis.yml) and the [test/](tests/) directory for examples.

License
=======

See [LICENSE.md](LICENSE.md)

Author Information
==================

https://cyverse.org

[packetbeat-dir-layout]: https://www.elastic.co/guide/en/beats/packetbeat/current/directory-layout.html
[elasticsearch-deb-dir-layout]: https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#deb-layout
[kibana-deb-dir-layout]: https://www.elastic.co/guide/en/kibana/current/deb.html#deb-layout
[logstash-dir-layout]: https://www.elastic.co/guide/en/logstash/current/dir-layout.html
[ansible-elasticsearch]: https://github.com/elastic/ansible-elasticsearch
[elastic-elasticsearch]: https://galaxy.ansible.com/elastic/elasticsearch/
