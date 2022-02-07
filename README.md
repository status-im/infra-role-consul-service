# Consul Service

Create the configuration file for a [Consul](https://www.consul.io/) service definition.

# Usage

## In Role

```yaml
- name: Create Consul service definition
  include_role: name=consul-service
  vars:
    consul_config_name: 'app-abc'
    consul_services:
      - name: 'app-abc'
        tags: ['abc', 'app']
        address: '12.34.56.78'
        port: 1234
        checks:
          - id: 'app-abc-status'
            name: 'ABC Healthcheck'
            type: http
            http: 'http://localhost:1234/health'
            interval: '60s'
            timeout: '2s'
            success_before_passing: 0
            failures_before_warning: 2
            failures_before_critical: 3
```

## Checks

Most fields are optional and have sane defaults, though you do want to define a type or your check won't do a thing.
Each check has one or more mandatory fields (usually named like the check type), see the examples below:

```yaml
checks:
  - id: 'web-http-check'
    name: 'Checks for /_ping on port'
    type: http
    # required, the URL to ping, checks the return code (2xx for pass, 429 warning, anything else error)
    http: 'http://{{ consul_service_address }}:{{ consul_service_port }}/_ping'

  - id: 'web-tcp-check'
    name: 'Checks for TCP connection on port'
    type: tcp
    # optional, defaults to localhost:{{ consul_service_port }}
    tcp: '{{ consul_service_address }}:{{ consul_service_port }}'

  - id: 'web-script-check'
    name: 'Runs script and checks the exit code'
    type: script
    # required, cannot take arguments
    script: '/usr/local/bin/myscript'

  - id: 'web-ttl-check'
    name: 'Stays up for as long as the TTL value, needs to be pinged through the HTTP api'
    type: ttl
    # required, takes a human duration
    ttl: '5m'

  - id: 'web-docker-check'
    name: 'Does a docker exec in a container and checks the exit code'
    type: docker
    # required, the ID of the container to run the check in
    docker_container_id: '1d88ad189f4'
    shell: '/bin/bash'
    script: '/app/local-check.py'
```

The following variables are also available:

* `interval` - The interval between each check (N/A for TTL).
* `timeout` - How much time the check has to answer (N/A for TTL).
* `deregister_critical_service_after` - Deregister service after it has been marked as critical for the given amount of time.
