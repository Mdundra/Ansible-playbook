# Ansible-playbook

- hosts: localhost
  become: true
  tasks:
  - debug:
      var: ansible_os_family
      
  - name: "create yum repository for docker"
    yum_repository:
      name: docker-repo
      description: "repo for docker"
      baseurl: "https://download.docker.com/linux/centos/7/x86_64/stable/"
      enabled: yes
      gpgcheck: no
    when: ansible_facts['os_family'] == 'RedHat'
  
  - name: "Install docker"
    package: 
      name: "docker-ce-18.09.1-3.el7.x86_64"
      state: present

  - name: "Install pip"
    package: 
      name: "python3-pip"
      state: present
      update_cache: true
    
  - name: "Install docker sdk"
    pip:
     name: "docker"

above code is for docker configuration



This is for launching docker container 
- hosts: localhost
  become: true  
  vars:
  - dockerfile_folder: "Dockerfile"
  - docker_image_name: "ssh_os:1"
  - docker_container_name: "test_os"
  - patting_ssh_port: "2222"
  - patting_http_port: "80"
  tasks:
  - name: "Copy Docker file"
    copy:
     src: "{{dockerfile_folder}}"
     dest: "/srv/"

  - name: "Build docker image from Dockerfile"
    docker_image:
       name:  "{{docker_image_name}}"
       build:
         pull: yes
         path: "/srv/{{dockerfile_folder}}/"
       state: present
       source: build
  
  - name: "launch docker container"
    docker_container:
       name: "{{docker_container_name}}"
       image: "{{docker_image_name}}"
       state: started
       ports:
       - "{{patting_ssh_port}}:22"
       -  "{{patting_http_port}}:8080"
    register: docker_info


  - name: "Update inventory file"
    lineinfile:
      path: hosts
      insertafter: '^\[containers]'
      firstmatch: yes
      line: "{{docker_info.ansible_facts.docker_container.NetworkSettings.IPAddress}}"
      state: present



We need to setup any tool which can be accessed or executed by service user instead of actual user on runners/servers.



