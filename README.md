elasticsearch_curator
=====================

Role which installs and configures Elasticsearch Curator.

The configuration of the role is done in such way that it should not be necessary
to change the role for any kind of configuration. All can be done either by
changing role parameters or by declaring completely new configuration as a
variable. That makes this role absolutely universal. See the examples below for
more details.

Please report any issues or send PR.


Example
-------

```yaml
---

# Example of how to use the role with default configuration
- hosts: myhost1
  roles:
    - elasticsearch_curator

# Example of how to customize the configuration
- hosts: myhost2
  vars:
    # Change elasticsearch target hosts
    elasticsearch_curator_config_client_hosts:
      - es1.prd.example.com
      - es1.stg.example.com
    # Add SSL parameters into the client configuration
    elasticsearch_curator_config_client__custom:
      client_cert: /path/to/my/file.crt
      client_key: /path/to/my/file.key
    # Adding logging configuration into the config file
    elasticsearch_curator_config__custom:
      blacklist:
        - elasticsearch
        - urllib3
      logfile: /var/log/currator.log
      logformat: default
      loglevel: INFO
    # Define action files
    elasticsearch_curator_actions:
      # This is the actual filename
      myactions1:
        # Here starts the content of the actual actions file
        actions:
          1:
            action: delete_indices
            description: >-
              Delete indices older than 45 days (based on index name), for
              logstash-prefixed indices. Ignore the error if the filter does not
              result in an actionable list of indices (ignore_empty_list) and
              exit cleanly.
            options:
              ignore_empty_list: yes
              disable_action: yes
            filters:
              - filtertype: pattern
                kind: prefix
                value: logstash-
              - filtertype: age
                source: name
                direction: older
                timestring: '%Y.%m.%d'
                unit: days
                unit_count: 45
    # Configure cron jobs
    elasticsearch_curator_crons:
      - name: Run myaction1
        minute: 0
        hour: 0
        job: >-
          curator --config {{ elasticsearch_curator_config_file }}
          {{ elasticsearch_curator_actions_path }}/myactions1.yaml
  roles:
    - elasticsearch_curator
```


Role variables
--------------

List of variables used by the role:

```yaml
# YUM repo URL
elasticsearch_curator_yum_repo_url: https://packages.elastic.co/curator/5/centos/{{ ansible_facts.distribution_major_version }}

# YUM repo GPG key
elasticsearch_curator_yum_repo_key: "{{ elastic_yum_repo_key | default('https://packages.elastic.co/GPG-KEY-elasticsearch') }}"

# Additional YUM repo parameters
elasticsearch_curator_yum_repo_params: "{{ elastic_yum_repo_params | default({}) }}"

# GPG key for the APT repo
elasticsearch_curator_apt_repo_key: "{{ elastic_apt_repo_key | default('https://artifacts.elastic.co/GPG-KEY-elasticsearch') }}"

# APT repo string
elasticsearch_curator_apt_repo_string: "{{ elastic_apt_repo_string | default('deb [arch=amd64] https://packages.elastic.co/curator/5/debian stable main') }}"

# Additional APT repo parameters
elasticsearch_curator_apt_repo_params: "{{ elastic_apt_repo_params | default({}) }}"

# Package to be installed (explicit version can be specified here)
elasticsearch_curator_pkg: elasticsearch-curator


# Path to the config file
elasticsearch_curator_config_file: /opt/elasticsearch-curator/curator.yaml

# Values of the default options of the client section
elasticsearch_curator_config_client_hosts:
  - 127.0.0.1
elasticsearch_curator_config_client_port: 9200

# Default options of the client section
elasticsearch_curator_config_client__default:
  hosts: "{{ elasticsearch_curator_config_client_hosts }}"
  port: "{{ elasticsearch_curator_config_client_port }}"

# Custom options of the client section
elasticsearch_curator_config_client__custom: {}

# Final client section of the config file
elasticsearch_curator_config_client: "{{
  elasticsearch_curator_config_client__default | combine(
  elasticsearch_curator_config_client__custom) }}"

# Default options of the config
elasticsearch_curator_config__default:
  client: "{{ elasticsearch_curator_config_client }}"

# Custom options of the config
elasticsearch_curator_config__custom: {}

# Final config
elasticsearch_curator_config: "{{
  elasticsearch_curator_config__default | combine(
  elasticsearch_curator_config__custom) }}"


# Default path for the action files
elasticsearch_curator_actions_path: /opt/elasticsearch-curator

# Action files definitions (see README.md for examples)
elasticsearch_curator_actions: {}


# List of cronjobs to run Curator (see README.md for examples)
elasticsearch_curator_crons: []
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`elasticsearch`](https://github.com/jtyr/ansible-elasticsearch) (optional)
- [`filebeat`](https://github.com/jtyr/ansible-filebeat) (optional)
- [`logstash`](https://github.com/jtyr/ansible-logstash) (optional)
- [`kibana`](https://github.com/jtyr/ansible-kibana) (optional)


License
-------

MIT


Author
------

Jiri Tyr
