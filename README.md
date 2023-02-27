# Filebeat

An Ansible Role that installs [Filebeat](https://www.elastic.co/products/beats/filebeat) on RedHat/CentOS
## Requirements

### Ansible User Prerequisites

Some tasks of this role need to be run as root(thanks to condition 'when ansible_user_id == root')
So, a 'non-root' user just can deploy configuration files under conf.d directory.
Java and Logstash installations and admin settings (conf.d directory path) are reserved by root user.

### Operating Systems support

* Centos 7
* Redhat 7,8


## Dependencies

None.

## Role Variables

| Variables | Required | Default value | Description |
|-----------|----------|---------------|-------------|
| filebeat_version | false | *7.x* | Major version of filebeat package from elasticsearch repository (https://www.elastic.co/downloads/beats/filebeat) (as root user) |
| filebeat_disable_install | false | *false* | Will skip packages installation, service managed and directories creation(as root user) |
| filebeat_enabled_on_boot | false | *true* | enabled systemd service on boot(as root user) |
| filebeat_create_config | false | *true* | Whether to create the Filebeat configuration file and handle the copying of SSL key and cert for filebeat. |
| filebeat_confd_file_user | false | *root* | Owner of Filebeat configuration files on server |
| filebeat_confd_file_group | false | *root* | Group owning Filebaet configuration files on server |
| filebeat_inputs | false | - type: log paths: - "/var/log/*.log" | Inputs that will be listed in the inputs section of the Filebeat configuration. Read through the Filebeat Inputs configuration guide(https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html) for more options. |
| filebeat_output_elasticsearch_enabled | false | *false* | Enables Elasticsearch output |
| filebeat_output_elasticsearch_hosts | false | *"localhost:9200"* | List of Elasticsearch hosts to send output to. |
| filebeat_output_logstash_enabled | false | *true* | Enables logstash output |
| filebeat_output_logstash_hosts | false | *"localhost:5044"* | List of logstash hosts to send output to. |
| filebeat_enable_logging | false | *false* | Enables filebeat Logging |
| filebeat_log_level | false | *warning* | Logging severity |
| filebeat_log_dir | false | */var/log/filebeat* | log directory path on server(as root user) |
| filebeat_log_filename | false | *filebeat.log* | log filename |
| filebeat_ssl_dir | false | */etc/pki/logstash* | ssl directory path on server |
| filebeat_ssl_certificate_file | "" | Local path to the SSL certificate file, which will be copied into the *filebeat_ssl_dir* |
| filebeat_ssl_key_file | false | "" | Local path to the SSL kry file, which will be copied into the *filebeat_ssl_dir* |
| filebeat_ssl_insecure | false | *false* | Set this to *true* to allow the use of self-signed certificates (when a CA isn't available). |

```yaml
    filebeat_inputs:
      - type: log
        paths:
          - "/var/log/*.log"
        tags: ["json"]
        fields:
          app_id: query_engine_12
          apache: true
          type: varlogs
        fields_under_root: true
      # pipeline:
        keep_null: true
        index: "%{[agent.name]}-myindex-%{+yyyy.MM.dd}"
        publisher_pipeline.disable_host: true
        multilines:
          type: pattern
          pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
          negate: 'true'
          match: after
```
Inputs that will be listed in the `inputs` section of the Filebeat configuration.
Note that we use a particular keyname `multilines` for multiline.<all keys> definition, like an example below :

```yaml
        multilines:
          type: pattern
```
became this line in final configuration file :
```yaml
        multiline.type: pattern
```

Read through the [Filebeat Inputs configuration guide](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html) for more options.

### Role Vars file

By example, I would like to install(as root user) filebeat and set configuration files, place following content into vars yaml file

```yaml
---

# Installation
filebeat_version: '7.x'
filebeat_disable_install: false
filebeat_enabled_on_boot: true
filebeat_create_config: true

# configuration
filebeat_confd_file_user: root
filebeat_confd_file_group: root

filebeat_inputs:
  - type: log
    paths:
      - "/var/log/*.log"

filebeat_output_elasticsearch_enabled: false
filebeat_output_elasticsearch_hosts:
  - "localhost:9200"

filebeat_output_logstash_enabled: true
filebeat_output_logstash_hosts:
  - "localhost:5044"

filebeat_enable_logging: false
filebeat_log_level: warning
filebeat_log_dir: /var/log/filebeat
filebeat_log_filename: filebeat.log

filebeat_ssl_dir: /etc/pki/logstash
filebeat_ssl_certificate_file: ""
filebeat_ssl_key_file: ""
filebeat_ssl_insecure: "false"

Example Playbook
----------------

Including an example of how to use your role :

```yaml
---
    - hosts: servers
      vars_files:
        - vars_file.yml
      roles:
        - role: filebeat
```

Tests
-----

The tests were done by using molecule integration test(inspec)

## License

MIT / BSD
