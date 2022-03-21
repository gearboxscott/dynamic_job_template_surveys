role_import_survey Name
=========

`role_import_survey` will take a survey spec and modify the current survey question in a given job template in Ansible Controller. It can add, remove and modify questions.  

Requirements
------------

* Collections Required:

| Collection | Description |
| :--------- | :---------- |
| `awx.awx` | provides access to `awx.awx.job_template` which performs the import of the survey spec into Ansible Controller. |
| `gearboxscott.dynamic_job_template_surveys` | provides access to `gearboxscott.dynamic_job_template_surveys` which modifies job template survey questions |


Role Variables
--------------

| Variable | Type | Required | Default | Description |
| :-------- | :---- | :---------| :------- | :----------- |
| `survey_spec_in_file` | boolean | Yes | `True` | Will direct the role to read the survey spec from a file. |
| `survey_spec_file` | string | No | Null | If `survey_spec_in_file` is enabled, then set `survey_spec_file` to the name of the file to be read into the variable `survey_spec`. |
| `survey_spec` | dictionary | Yes | Null | YAML dictionary formatted survey definition. |
| `job_template_name` | string | Yes | Null | Name of the Job Template. |
| `controller_host` | string | Yes | Null | Ansible Controller Hostname (without `https://`). If value not set, will try environment variable `CONTROLLER_HOST` and then config files |
| `controller_username` | string | Yes | Null | Ansible Controller Username (with role to modify job templates). If value not set, will try environment variable `CONTROLLER_USERNAME` and then config files. |
| `controller_password` | string | Yes | Null | Password for Ansible Controller Username. If value not set, will try environment variable `CONTROLLER_PASSWORD` and then config files. |
| `controller_validate_certs` | string | False | Whether to allow insecure connections to Ansible Controller. If value not set, will try environment variable CONTROLLER_VERIFY_SSL and then config files. |
| `survey_enabled` | boolean | True | Enable a survey on the job template. |

Dependencies
------------

See Requirements Above.

Example Playbook
----------------

Example Playbook `import_survey_questions.yml`:

```yaml
---
- name: Project Job Template Survey - import
  hosts: localhost
  gather_facts: no
  collections:
    - awx.awx
    - gearboxscott.dynamic_job_template_surveys

  tasks: 

  - name: import survey questions
    include_role:
      name: gearboxscott.dynamic_job_template_surveys.role_import_survey
      apply:
        tags:
          - survey_import
    vars:
      controller_host: ansible controller hostname
      controller_username: ansible controller username
      controller_password: ansible controller username's password
      job_template_name: name of job template or job template workflow
      survey_spec_file: filename of survey question spec or dictionary
      survey_spec_in_file: optional, true or false
    tags:
      - always
    register: results_out

...

```

License
-------

GPL-3.0-or-later

Author Information
------------------

Scott Parker <gearboxscott@gmail.com>
