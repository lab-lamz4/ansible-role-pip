# Ansible Role: Pip (for Python)

An Ansible Role that installs [Pip](https://pip.pypa.io) on Linux.

## CI with Jenkins

This role contains Jenkinsfile for CI.

Install jenkins master from role https://github.com/lean-delivery/ansible-role-jenkins

Install jenkins slaves (if you have it) from role https://github.com/lean-delivery/ansible-role-jenkins-slave

Playbook for these actions like this:

```yaml

- name: Get facts from slaves
  hosts: j-nodes
  gather_facts: yes
  tags:
    - slave

- name: Install and Configure Jenkins
  hosts: j-master
  vars:
    # java settings
    transport: repositories
    java_major_version: 8
    # jenkins settings
    jenkins2_ssh_keys_inv_slave_groupname: 'j-nodes'
    # jenkins2_ssh_keys_private_keyname: jenkins_nodes
    jenkins2_ssh_keys_slave_hosts:
      - {host: 'j-node1.example.com', users: ['root', 'jenkins']}
      - {host: 'j-node2.example.com', users: ['root', 'jenkins']}
    jenkins2_credentials:
      slave:
        type: 'key'
        keySource: 0
        key: "{{ jenkins_private_key }}"
        id: 'slave'
        username: 'jenkins'
        passphrase: ''
        description: 'credentials for slave node'
  roles:
    - role: ansible-role-java
    - role: ansible-role-jenkins
    - role: ansible-role-docker
      become: True
  tasks:
    - name: install pip and virtualenv
      block:
      - name: install pip
        apt:
          name: python-pip
          state: present
          update_cache: yes
          cache_valid_time: 3600

      - name: install virtualenv
        pip:
          name: virtualenv

      - name: Add user(s) to "docker" group
        user:
          name: "jenkins"
          groups: "docker"
          append: true
      become: True

- name: Install Docker to node1
  hosts: j-node1
  vars:
    # java settings
    transport: repositories
    java_major_version: 8
    # jenkins
    master_host: "j-master.example.com"
    slave_linux_host: "j-node1.example.com"
    slave_agent_name: "{{ inventory_hostname }}"
    slave_linux_jenkins_cred_id: slave
    slave_linux_jenkins_username: jenkins
    slave_linux_user_group: jenkins
    slave_linux_jenkins_public_key: "{{ jenkins_slave_pub_key }}"
  tasks:
    - name: install pip and virtualenv
      block:
      - name: install pip
        apt:
          name: python-pip
          state: present
          update_cache: yes
          cache_valid_time: 3600
        tags:
          - virtualenv

      - name: install virtualenv
        pip:
          name: virtualenv
        tags:
          - virtualenv

      - name: Add user(s) to "docker" group
        user:
          name: "jenkins"
          groups: "docker"
          append: true
      become: True
  roles:
    - { role: ansible-role-java, tags: ["java"] }
    - { role: ansible-role-jenkins-slave, tags: ["slave"] }
    - { role: ansible-role-docker, tags: ["docker"] }
      become: True

- name: Install Docker to node2
  hosts: j-node2
  vars:
    # java settings
    transport: repositories
    java_major_version: 8
    # jenkins
    master_host: "j-master.example.com"
    slave_linux_host: "j-node2.example.com"
    slave_agent_name: "{{ inventory_hostname }}"
    slave_linux_jenkins_cred_id: slave
    slave_linux_jenkins_username: jenkins
    slave_linux_user_group: jenkins
    slave_linux_jenkins_public_key: "{{ jenkins_slave_key }}"

  tasks:
    - name: install pip and virtualenv
      block:
      - name: install pip
        apt:
          name: python-pip
          state: present
          update_cache: yes
          cache_valid_time: 3600
        tags:
          - virtualenv

      - name: install virtualenv
        pip:
          name: virtualenv
        tags:
          - virtualenv

      - name: Add user(s) to "docker" group
        user:
          name: "jenkins"
          groups: "docker"
          append: true
      become: True
  roles:
    - { role: ansible-role-java, tags: ["java"] }
    - { role: ansible-role-jenkins-slave, tags: ["slave"] }
    - { role: ansible-role-docker, tags: ["docker"] }
      become: True


```

1. Create credentials for github (generate ssh-key, put public key to deployment keys for your repository, configure jenkins with private key)

2. Create Multibranch Pipeline.

3. Set name and description

4. Configure Branch Sources with Git

4.1. Project Repository    -  https://github.com/lab-lamz4/ansible-role-pip

4.2. Credentials - early created in first step

4.3. Build Configuration - by Jenkinsfile

4.4. Scan Multibranch Pipeline Triggers - Periodically if not otherwise run 2minutes

4.5. Orphaned Item Strategy - checked

Save.

Pipeline:

In pipeline i used parallel stages for running in nodes.
Pipeline logic:
- Run every 2 minutes on master node, check git for changes.
- If new commits, git pull and stash files on master
- parallel stage configured by
def scenarios = [centos7 : 'node1', debian9 : 'node2', centos6 : 'node1']
- delete if exist virtualenv folder in workspace
- put to node fresh code from git
- setup virtualenv, name for folder configured by
setting environment  TESTDIR
- check versions of docker, python,ansible and molecule
- activate virtualenv and run molecule test with scenario name defined in scenarios variable

## Requirements

- **Supported OS**:
  - Ubuntu
    - bionic
  - Debian
    - stretch
  - EL
    - 6 (in this case python2.7 and pip2.7 will be installed)
    - 7

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    pip_package: python-pip

The name of the packge to install to get `pip` on the system. You can set to `python3-pip`, for example, when using Python 3 on Ubuntu.

    pip_executable: pip

The role will try to autodetect the pip executable based on the `pip_package` (e.g. `pip` for Python 2 and `pip3` for Python 3). You can also override this explicitly, e.g. `pip_executable: pip3.6`.

    pip_install_packages: []

A list of packages to install with pip. Examples below:

    pip_install_packages:
      # Specify names and versions.
      - name: docker
        version: "1.2.3"
      - name: awscli
        version: "1.11.91"

      # Or specify bare packages to get the latest release.
      - docker
      - awscli

      # Or uninstall a package.
      - name: docker
        state: absent

      # Or update a package ot the latest version.
      - name: docker
        state: latest

      # Or force a reinstall.
      - name: docker
        state: forcereinstall

      # Or install a package in a particular virtualenv.
      - name: docker
        virtualenv: /my_app/venv

## Dependencies

None.

## Example Playbook

    - hosts: all

      vars:
        pip_install_packages:
          - name: docker
          - name: awscli

      roles:
        - ansible-role-pip

## License

MIT / BSD

## Author Information

Original role was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/) .

Forked in 2019 by lamz4
