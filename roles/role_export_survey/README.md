role_export_survey Name
=========

`role_import_survey` will export a job template survey questions of a specific job template named.

Requirements
------------

* Collections Required:

| Collection | Description |
| :--------- | :---------- |
| `gearboxscott.dynamic_job_template_surveys` | provides access to `gearboxscott.dynamic_job_template_surveys` which modifies job template survey questions |


Role Variables
--------------

| Variable | Type | Required | Default | Description |
| :-------- | :---- | :---------| :------- | :----------- |
| `survey_spec_file` | string | No | Null | If `survey_spec_in_file` is enabled, then set `survey_spec_file` to the name of the file to be read into the variable `survey_spec`. |
| `job_template_name` | string | Yes | Null | Name of the Job Template. |
| `controller_host` | string | Yes | Null | Ansible Controller Hostname (without `https://`). If value not set, will try environment variable `CONTROLLER_HOST` and then config files |
| `controller_username` | string | Yes | Null | Ansible Controller Username (with role to modify job templates). If value not set, will try environment variable `CONTROLLER_USERNAME` and then config files. |
| `controller_password` | string | Yes | Null | Password for Ansible Controller Username. If value not set, will try environment variable `CONTROLLER_PASSWORD` and then config files. |
| `controller_validate_certs` | string | False | Whether to allow insecure connections to Ansible Controller. If value not set, will try environment variable CONTROLLER_VERIFY_SSL and then config files. |

Dependencies
------------

See Requirements Above.

Example Playbook
----------------

Example Playbook `export_survey_questions.yml`:

```yaml
---
- name: Project Job Template Survey - export
  hosts: localhost
  gather_facts: no
  collections:
    - gearboxscott.dynamic_job_template_surveys

  tasks: 
  - name: Pull in Credentials from environment
    set_fact:
      controller_host: "{{ lookup( 'env', 'CONTROLLER_HOST' ) }}"
      controller_username: "{{ lookup( 'env', 'CONTROLLER_USERNAME' ) }}"
      controller_password: "{{ lookup( 'env', 'CONTROLLER_PASSWORD' ) }}"
      controller_validate_certs: "{{ lookup( 'env', 'CONTROLLER_VERIFY_SSL' ) }}"
    when: >
      ( lookup( 'env', 'CONTROLLER_HOST' ) | length > 0 ) and
      ( lookup( 'env', 'CONTROLLER_USERNAME' ) | length > 0 ) and
      ( lookup( 'env', 'CONTROLLER_PASSWORD' ) | length > 0 ) and
      ( lookup( 'env', 'CONTROLLER_VERIFY_SSL' ) | length > 0 ) 
    no_log: false

  - name: export survey questions
    include_role:
      name: gearboxscott.dynamic_job_template_surveys.role_export_survey
      apply:
        tags:
          - survey_export
    vars:
      controller_host: ansible controller hostname
      controller_username: ansible controller username
      controller_password: ansible controller username's password
      job_template_name: name of job template or job template workflow
      survey_spec_file: filename of survey question spec or dictionary
      controller_validate_certs: Whether to allow insecure connections to Ansible Controller (true or false)
    tags:
      - always
    no_log: false

...

```

License
-------

GPL-3.0-or-later

Author Information
------------------

Scott Parker <gearboxscott@gmail.com>
