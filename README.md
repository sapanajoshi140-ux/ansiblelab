# ansiblelab

Ansible Labs for DevOps

## Assignment Solution

### 1. Create `requirements.yml`

```yaml
roles:
  - name: geerlingguy.docker
```

### 2. Install the Role

```bash
ansible-galaxy install -r requirements.yml
```

<img width="958" alt="Install Role" src="https://github.com/user-attachments/assets/23c3aab7-1780-4500-b66f-4e1979b604a2" />

<br>

<img width="866" alt="Role Installed" src="https://github.com/user-attachments/assets/088d312f-805a-402b-a731-deea35331a5d" />

---

### 3. Create Templates Directory

```bash
mkdir ~/.ansible/roles/geerlingguy.docker/templates
```

<img width="932" alt="Templates Directory" src="https://github.com/user-attachments/assets/5cfa75b5-2665-4ee4-b0d5-6b508cfac834" />

---

### 4. Create Homepage Template

```bash
vim ~/.ansible/roles/geerlingguy.docker/templates/index.html.j2
```

<img width="670" alt="Template File" src="https://github.com/user-attachments/assets/4c63114e-041f-48f9-86a8-68627b653b4d" />

---

### 5. Create Site Directory

```bash
mkdir /opt/site
```

---

### 6. Edit Handlers

```bash
vim ~/.ansible/roles/geerlingguy.docker/handlers/main.yml
```

<img width="725" alt="Handlers" src="https://github.com/user-attachments/assets/1a0a72ea-da8b-4b4a-90c1-091c5701bcad" />

---

### 7. Edit Tasks

```bash
vim ~/.ansible/roles/geerlingguy.docker/tasks/main.yml
```

```yaml
---
- name: Load OS-specific vars.

  include_vars: "{{ lookup('first_found', params) }}"
  vars:
    params:
      files:
        - '{{ansible_facts.distribution}}.yml'
        - '{{ansible_facts.os_family}}.yml'
        - main.yml
      paths:
        - 'vars'

- include_tasks: setup-RedHat.yml
  when: ansible_facts.os_family == 'RedHat'

- include_tasks: setup-Suse.yml
  when: ansible_facts.os_family == 'Suse'

- include_tasks: setup-Debian.yml
  when: ansible_facts.os_family == 'Debian'

- name: Install Docker packages.
  package:
    name: "{{ docker_packages }}"
    state: "{{ docker_packages_state }}"
  notify: restart docker
  ignore_errors: "{{ ansible_check_mode }}"
  when: "ansible_version.full is version_compare('2.12', '<') or ansible_facts.os_family not in ['RedHat', 'Debian']"

- name: Install Docker packages (with downgrade option).
  package:
    name: "{{ docker_packages }}"
    state: "{{ docker_packages_state }}"
    allow_downgrade: true
  notify: restart docker
  ignore_errors: "{{ ansible_check_mode }}"
  when: "ansible_version.full is version_compare('2.12', '>=') and ansible_facts.os_family in ['RedHat', 'Debian']"

- name: Install docker-compose plugin.
  package:
    name: "{{ docker_compose_package }}"
    state: "{{ docker_compose_package_state }}"
  notify: restart docker
  ignore_errors: "{{ ansible_check_mode }}"
  when: "docker_install_compose_plugin | bool == true and (ansible_version.full is version_compare('2.12', '<') or ansible_facts.os_family not in ['RedHat', 'Debian', 'Suse'])"

- name: Install docker-compose-plugin (with downgrade option).
  package:
    name: "{{ docker_compose_package }}"
    state: "{{ docker_compose_package_state }}"
    allow_downgrade: true
  notify: restart docker
  ignore_errors: "{{ ansible_check_mode }}"
  when: "docker_install_compose_plugin | bool == true and ansible_version.full is version_compare('2.12', '>=') and ansible_facts.os_family in ['RedHat', 'Debian']"

- name: Ensure /etc/docker/ directory exists.
  file:
    path: /etc/docker
    state: directory
    mode: 0755
  when: docker_daemon_options.keys() | length > 0

- name: Configure Docker daemon options.
  copy:
    content: "{{ docker_daemon_options | to_nice_json }}"
    dest: /etc/docker/daemon.json
    mode: 0644
  when: docker_daemon_options.keys() | length > 0
  notify: restart docker

- name: Replace Docker service ExecStart command if configured.
  when: docker_service_start_command != ""
  notify: restart docker
  block:
    - name: Get Docker service status
      ansible.builtin.systemd_service:
        name: docker
      register: docker_service_status

    - name: Patch docker.service
      ansible.builtin.replace:
        path: "{{ docker_service_status.status['FragmentPath'] }}"
        regexp: "^ExecStart=.*$"
        replace: "ExecStart={{ docker_service_start_command }}"
      register: docker_service_patch

    - name: Reload systemd services
      service:
        daemon_reload: true
      when: docker_service_patch is changed

- name: Ensure Docker is started and enabled at boot.
  service:
    name: docker
    state: "{{ docker_service_state }}"
    enabled: "{{ docker_service_enabled }}"
  ignore_errors: "{{ ansible_check_mode }}"
  when: docker_service_manage | bool

- name: Ensure handlers are notified now to avoid firewall conflicts.
  meta: flush_handlers

- include_tasks: docker-compose.yml
  when: docker_install_compose | bool

- name: Get docker group info using getent.
  getent:
    database: group
    key: docker
    split: ':'
  when: docker_users | length > 0

- name: Check if there are any users to add to the docker group.
  set_fact:
    at_least_one_user_to_modify: true
  when:
    - docker_users | length > 0
    - item not in ansible_facts.getent_group["docker"][2]
  with_items: "{{ docker_users }}"

- include_tasks: docker-users.yml
  when: at_least_one_user_to_modify is defined

- name: Run nginx in a container
  community.docker.docker_container:
    name: hello-nginx
    image: nginx:latest
    state: started
    ports:
      - "8080:80"
    volumes:
      - "/opt/site:/usr/share/nginx/html:ro"
- name: Render the homepage
  ansible.builtin.template:
    src: templates/index.html.j2
    dest: /opt/site/index.html
  notify: Restart nginx
~
```

---

### 8. Create Playbook

```bash
vim assign.yml
```

<img width="471" alt="assign.yml" src="https://github.com/user-attachments/assets/45dbe224-568e-4e9e-9bb4-ad87efa291d4" />

---

### 9. Run the Playbook

```bash
ansible-playbook assign.yml
```

<img width="1545" alt="Playbook Run" src="https://github.com/user-attachments/assets/1838af54-a4e0-4b3c-81cd-4273d65972d4" />

<br>

<img width="762" alt="Result 1" src="https://github.com/user-attachments/assets/52ea99a1-0d1b-4069-817d-47cc30d937cd" />

<br>

<img width="1912" alt="Result 2" src="https://github.com/user-attachments/assets/f27f2f2e-a496-4243-b5b0-d3b8161bf38e" />

<br>

<img width="1916" alt="Result 3" src="https://github.com/user-attachments/assets/bba55b43-c74f-4854-bcbd-9ea968fa64f6" />

---

