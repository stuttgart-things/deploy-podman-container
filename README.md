deploy-podman-conatiner
=======================

deploys reboot resistant podman containers.

Requirements
------------

or any other system w/ installed podman, systemd + firewalld. e.g. CentOs8.

Example Playbook for a root container 
-------------------------------------

```
---
- hosts: loadbalancer
  become: true
  roles:
    - base-os-setup # installs basic os setup + updates packages
    - install-configure-podman # installs podman, buildah + skopeo
    - role: deploy-podman-container
      vars: 
        container_name: rancher-loadbalancer
        container_image: nginx:latest
        container_run_args: >-
          -d
        mounts:
          webserverconfig:
            host: /tmp/test
            container: /test:Z
        env_vars:
          hostname:
            key: INGRESS_HTTP
            value: master0.openshift.example
        ports:
          http:
            host: 80
            container: 80
          https:
            host: 443
            container: 443  
        #container_cmd_args: 
```

Role Requirements / Dependencies
--------------------------------

```
sudo cat <<EOF > ./requirements.yaml
roles:
- src: git@codehub.sva.de:Lab/stuttgart-things/supporting-roles/install-configure-podman.git
  scm: git
- src: git@codehub.sva.de:Lab/stuttgart-things/supporting-roles/deploy-podman-container.git
  scm: git
- src: git@codehub.sva.de:Lab/stuttgart-things/supporting-roles/install-requirements.git
  scm: git
  
collections:
- name: containers.podman 
EOF
ansible-galaxy install -r ./requirements.yaml --force && ansible-galaxy collection install -r ./requirements.yaml -f
```

License
-------

BSD

Author Information
------------------

Patrick Hermann (patrick.hermann@sva.de), SVA GmbH, 07/2020.

this role is heavily influenced by [podman-container-systemd](https://github.com/ikke-t/podman-container-systemd) - we didnt like the style of the role and for a better understanding for podman from a technology side, we're still in the process to rewrite/implement a similar role. 
