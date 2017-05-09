andock-ci-build (A drupal docksal build script.)
=========

**andock-ci-build** is a Ansible role which:
* Checks out a php repository (e.g. from github)
* runs build tasks (like composer, npm etc.). We recomend docksal (http://docksal.io/).
* pushes all build artifacts to a target build repository (can be a different one like Acquia or the same. Andock CI pushes to {{ branch }}-build branch.)  
  

Requirements
------------

In order to build your apps with Andock CI, you will need:

* Ansible in your deploy machine
* docksal (http://docksal.io/) on your target machine
* git on both machines


Role Variables
--------------

```yaml
vars:
  git_source_repository_path: git@github.com:andock-ci/drupal-8-demo.git # The source repository 
  git_target_repository_path: git@github.com:andock-ci/drupal-8-demo-build.git # The target repository. Can be the same repository as the source repository 
  build_dir: ~/ansible # The path where the build happens
  branch: "master" # The source branch. The target branch would be master-build
```

Installation
------------

Andock-CI is an Ansible role distributed globally using [Ansible Galaxy](https://galaxy.ansible.com/). In order to install Andock-CI role you can use the following command.

```
$ ansible-galaxy install andock-ci.andock-ci-build
```

Update
------

If you want to update the role, you need to pass **--force** parameter when installing. Please, check the following command:

```
$ ansible-galaxy install --force andock-ci.andock-ci-build
```

Dependencies
------------

@TODO

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - name: Deploy repository to build repository
      hosts: localhost
      remote_user: cw
      roles:
        - role: andock-ci-build
          git_source_repository_path: git@github.com:andock-ci/drupal-8-demo.git
          git_target_repository_path: git@github.com:andock-ci/drupal-8-demo-build.git
          build_dir: ~/ansible
          branch: "master"

Example gitlab configuration with shell executor
----------------
.gitlab-ci.yml:

    stages:
      - build
    
    before_script:
      # Install ssh if not already installed (https://docs.gitlab.com/ee/ci/ssh_keys/README.html#ssh-keys-when-using-the-docker-executor)
      # Install ansible if it is not installed (pip install ansible) 
      # Install the tools you need to compile your project. Such as composer. I recommend to use docksal. 
    
    build:
      stage: build
      script:
        - ansible-playbook .ansible/build.yml --extra-vars "branch=$branch build_dir=$CI_PROJECT_DIR/build git_source_repository_path=$CI_REPOSITORY_URL" --connection=local

playbook under .ansible/build.yml

    - name: Deploy repository to build repository
      hosts: localhost
      remote_user: cw
      roles:
        - role: andock-ci-build
          git_source_repository_path: {{ git_source_repository_path }}
          git_target_repository_path: git@github.com:andock-ci/drupal-8-demo-build.git
          build_dir: "{{ build_dir }}"
          branch: "{{ branch }}"
          hook_build_tasks: "hooks/build_tasks.yml"

build hooks under .ansible/hooks/build.yml

    - name: composer install
      command: fin rc -T composer install
      args:
        chdir: "{{ build_dir }}"


License
-------

BSD

Author Information
------------------

Christian Wiedemann (christian.wiedemann@key-tec.de)
