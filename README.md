# Ansible Collection

This Ansible Collection provides a method to update Controller Job Template survey questions choices with using external lists via a URL, modifying a export Job Template survey_spec section, and other means to get list of choices into a survey question.

## REQUIREMENTS

* Exporting Survey Questions
  
  ```yaml
  - name: Project Job Template Survey - export
    hosts: localhost
    gather_facts: no
    collections:
      - gearboxscott.dynamic_job_template_surveys
  ```

* Importing Survey Questions
  
  ```yaml
  - name: Project dynamicsurveys - importing
    hosts: localhost
    gather_facts: no
    collections:
      - awx.awx
      - gearboxscott.dynamic_job_template_surveys
  ```

## Installing this collection

You can install the redhat_cop controller_configuration collection with the Ansible Galaxy CLI:

```console
ansible-galaxy collection install awx.awx
ansible-galaxy collection install gearboxscott.dynamic_job_template_surveys
```

You can also include it in a `requirements.yml` file and install it with `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: awx.awx
  - name: gearboxscott.dynamic_job_template_surveys
    # If you need a specific version of the collection, you can specify like this:
    # version: ...
```

## Using this collection

* Exporting Survey Questions

  The following command will invoke the playbook with the collection on the Ansible Controller:

  ```yaml
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

  To run the above playbook from the Ansible Controller command line:  

  ```console
  ansible-playbook export.yml -vvv
  ```

  This will create a file in the directory specified by the `survey_spec_file`, for example: `survey_questions_download.yml`. Edit the file to add new values to choices under subsections of the file to prepare for it to be imported into Ansible Controller.

* Exporting Survey Questions

  The following command will invoke the playbook with the collection on the Ansible Controller:

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

  To run the above playbook from Ansible Controller command line:

  ```console
  ansible-playbook import.yml -vvv
  ```

  This will update the job template name in variable `job_template_file` with update values in the file named in the `survey_spec_file` variable, for example: `survey_questions_upload.yml`. 

## Example Survey Spec File:

* First a downloaded Job Template Survey Spec for Job Template named `controller_survey`:

  ```yaml
  name: ""
  description: ""
  spec:
  -   choices: 'Tiger Woods

          Jack Nicklaus

          Sam Snead

          Arnold Palmer

          Francis Ouimet

          Walter Hogan'
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick Your Favorite Golfer
      required: true
      type: multiplechoice
      variable: survey_favorite_golfer
  -   choices: 'Dilbert

          Calvin & Hobbs

          Peanuts'
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick A Comic Strip
      required: true
      type: multiplechoice
      variable: survey_comic_strip
  -   choices: ''
      default: Tiger Woods
      formattedChoices:
      -   choice: ''
          id: 0
          isDefault: false
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Enter Name
      required: true
      type: text
      variable: survey_name
  -   choices: ''
      default: $encrypted$
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Enter Password
      required: true
      type: password
      variable: survey_password
  -   choices: 'dog

          cat

          fish

          bird

          turtle

          elephant

          ducks'
      default: ''
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Pick One
      required: true
      type: multiplechoice
      variable: survey_choices
  -   choices: 'Formula 1

          NASCAR

          IndyCar

          CART

          Midgets

          IMSA'
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick Car Racing Series
      required: true
      type: multiselect
      variable: survey_multiple_choices
  -   choices: 'VLAN2: 10.10.2.0/24

          VLAN3: 10.10.3.0/24

          VLAN11: 10.10.11:0/24'
      default: ''
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Pick a VLAN
      required: true
      type: multiplechoice
      variable: survey_choice_indexed
    ```

* It was modified to look like this example:

  ```yaml
  name: ""
  description: ""
  spec:
  -   choices: "{{ lookup( 'file', 'files/pga.txt' ) }}"
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick Your Favorite Golfer
      required: true
      type: multiplechoice
      variable: survey_favorite_golfer
  -   choices: "{{ lookup( 'url', 'https://raw.githubusercontent.com/gearboxscott/lists_of_stuff/main/comic_strips.txt', split_lines=True ).split(',') | join('\n') }}"
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick A Comic Strip
      required: true
      type: multiplechoice
      variable: survey_comic_strip
  -   choices: ''
      default: Tiger Woods
      formattedChoices:
      -   choice: ''
          id: 0
          isDefault: false
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Enter Name
      required: true
      type: text
      variable: survey_name
  -   choices: ''
      default: $encrypted$
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Enter Password
      required: true
      type: password
      variable: survey_password
  -   choices: "{{ lookup( 'file', 'files/animals.txt' ) }}"
      default: ''
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Pick One
      required: true
      type: multiplechoice
      variable: survey_choices
  -   choices: "{{ lookup( 'file', 'files/racingseries.txt' ) }}"
      default: ''
      max: 1024
      min: 0
      new_question: true
      question_description: ''
      question_name: Pick Car Racing Series
      required: true
      type: multiselect
      variable: survey_multiple_choices
  -   choices: "{{ lookup( 'file', 'files/vlans.txt' ) }}"
      default: ''
      max: 1024
      min: 0
      new_question: false
      question_description: ''
      question_name: Pick a VLAN
      required: true
      type: multiplechoice
      variable: survey_choice_indexed
  ```

* Individual file examples for each survey question above to be imported back into Ansible Controller:

  * Listing of `animals.txt`:

    ```text
    dog
    cat
    fish
    bird
    turtle
    elephant
    zebra
    gorillas
    giraffes
    ducks
    goose
    dolphins
    ```

  * Listing of `pga.txt`:
  
    ```text
    Tiger Woods
    Jack Nicklaus
    Sam Snead
    Arnold Palmer
    Francis Ouimet
    Walter Hogan
    Ben Hogan
    Tom Watson
    Bobby Jones
    Lee Trevino
    ```

  * Listing of `racingseries.txt`:
  
    ```text
    Formula 1
    NASCAR
    IndyCar
    CART
    Midgets
    Sprint
    NHRA
    IMSA
    ```

  * Listing of `vlans.txt`:
   
    ```text
    VLAN2: 10.10.2.0/24
    VLAN3: 10.10.3.0/24
    VLAN10: 10.10.10.0/24
    VLAN11: 10.10.11:0/24
    ```

  * Lastly a URL lookup of the contents of a file on github for `comic_strips.txt` found at https://github.com/gearboxscott/lists_of_stuff/blob/main/comic_strips.txt:

    ```text
    B.C.
    Beetle Bailey
    Bloom County
    Calvin & Hobbs
    Dilbert
    Garfield
    Peanuts
    The Far Side
    ```

    This was added so show that other source to list could be used to update choices in a survey question. Reference the lookup command for examples: [Lookup plugins](https://docs.ansible.com/ansible/latest/plugins/lookup.html). There are many types found here: [Ansible Builtin](https://docs.ansible.com/ansible/latest/collections/index_lookup.html#ansible-builtin).


## See Also

- [Ansible Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) for more details.

## Release and Upgrade Notes

For details on changes between versions, please see [the changelog for this collection](CHANGELOG.rst).

## Roadmap

Adding more examples and more ability to modify other parts of the Job Template Survey Questions in programmatic and dynamic content way.

## Contributing to this collection

We welcome community contributions to this collection. If you find problems, please open an issue or create a PR against the [Dynamic Job Template Surveys repository](https://github.com/gearboxscott/dynamic_job_template_surveys).

## Licensing

GNU General Public License v3.0 or later.

See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) to see the full text.