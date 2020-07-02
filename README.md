deploy-podman-conatiner
=======================

deploys reboot resistant podamn containers on target systems. 

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
        ports:
          http:
            host: 80
            container: 80
          https:
            host: 443
            container: 443  
        #container_cmd_args: 
```

Role Requirements 7 Dependencies
--------------------------------

```
sudo cat <<EOF > ./requirements.yaml
- src: git@codehub.sva.de:Lab/stuttgart-things/ansible/base-os-setup.git
  scm: git
- src: git@codehub.sva.de:Lab/stuttgart-things/ansible/install-configure-podman.git
  scm: git
- src: git@codehub.sva.de:Lab/stuttgart-things/kubernetes/deploy-podman-container.git
  scm: git
EOF
ansible-galaxy install -r ./requirements.yaml --force
```

License
-------

BSD

Author Information
------------------

Patrick Hermann (patrick.hermann@sva.de), SVA GmbH, 07/2020
this role was heavily influenced by [podman-container-systemd](https://github.com/ikke-t/podman-container-systemd) - but we didnt like the style of the role and for a better understanding for podman from a technology side, we're still in the process to rewrite/implement a similar role. 
