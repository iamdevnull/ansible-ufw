## This ansible role is based on weareinteractive.ufw role
## additionally it integrates [ansible-ufw](https://github.com/chaifeng/ufw-docker) and [ufw-docker-automated](https://github.com/shinebayar-g/ufw-docker-automated)

### Usage

```shell
$ cd <roles-folder>
$ git clone https://github.com/iamdevnull/ansible-ufw.git
$ cd ansible-ufw
```

This is an example playbook:

```yaml
# @see https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html#examples
---

- hosts: all
  become: true

  vars:
    ufw_reset: false
    ufw_enabled: false
    ufw_manage_config: true
    ufw_config:
      IPV6: "no"
      DEFAULT_INPUT_POLICY: DROP
      DEFAULT_OUTPUT_POLICY: ACCEPT
      DEFAULT_FORWARD_POLICY: DROP
      DEFAULT_APPLICATION_POLICY: SKIP
      MANAGE_BUILTINS: "no"
      IPT_SYSCTL: /etc/ufw/sysctl.conf
      IPT_MODULES: ""
    ufw_rules:
      - logging: "full"
      - rule: allow
        name: OpenSSH
      - rule: allow
        interface: br-+
        direction: in
        proto: any
        to_port: '53'
      - rule: allow
        interface: docker0
        direction: in
        proto: any
        to_port: '53'

  pre_tasks:
    - name: reset ufw
      community.general.ufw:
        state: reset
      when: ufw_reset and not ufw_enabled | bool
      register: ufw_reset_exitcode

  roles:
    - ansible-ufw

  post_tasks:
    - name: enable ufw
      community.general.ufw:
        state: enabled
      when: ufw_reset_exitcode is succeeded

    - name: find ufw backup files
      find:
        paths: /etc/ufw
        patterns: "*.rules.*"
      register: wildcard_files_to_delete
      when: ufw_reset_exitcode is succeeded

    - name: delete ufw backup files
      file:
        path: "{{ item.path }}"
        state: absent
      with_items: "{{ wildcard_files_to_delete.files }}"
      when: ufw_reset_exitcode is succeeded
```

### Usage with docker or docker-compose

```yaml
version: "2.4"
services:
  web:
    image: nginx:alpine
    ports:
      - "HOSTPORT:CONTAINERPORT" # host-port-binding is mandatory for ufw autodiscovery
    labels:
    UFW_MANAGED: 'TRUE' # allow external/public access to the published port of the container

networks:
  webnet:
```

## License
Copyright (c) We Are Interactive under the MIT license.
