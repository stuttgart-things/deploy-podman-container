deploy-podman-conatiner
=======================

deploys reboot resistant podman containers.

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
- src: git@codehub.sva.de:Lab/stuttgart-things/supporting-roles/generate-selfsigned-certs.git
  scm: git
collections:
- name: containers.podman
- name: community.general 
- name: ansible.posix

EOF
ansible-galaxy install -r ./requirements.yaml --force && ansible-galaxy collection install -r ./requirements.yaml -f
```

Example Playbook + vars profile for deploying a minio container 
---------------------------------------------------------------

```
cat <<EOF > ./deploy-podman-container.yaml
---
- hosts: "{{ target_host | default('all') }}"
  gather_facts: true
  become: true
  vars_files:
    - "./{{ container }}.yaml"

  roles:
    - role: deploy-podman-container
EOF

cat <<EOF > ./minio.yaml
---
container_name: minio
permanent: true
container_image: minio/minio:latest
container_run_args: >-
    -d
container_cmd_args: >-
    server /data
mounts:
  data:
    host: "/podman/pv/{{ container_name }}/data/"  
    container: /data:z
  config:
    host: "/podman/pv/{{ container_name }}/config/"
    container: /config:z
  certs:
    host: "/podman/pv/{{ container_name }}/certs/"
    container: /root/.minio/certs:z

env_vars:
  ACCESS_KEY:
    key: MINIO_ACCESS_KEY
    value: admin
  SECRET_KEY:
    key: MINIO_SECRET_KEY
    value: jackhammer
ports:
  minio:
    host: 9000
    container: 9000
EOF
```

Playbook execution
---------------------------------------------------------------
```
ansible-playbook -i ../my_inventory_file \
-e target_host=srv-minio
-e container=minio \
deploy-podman-container.yaml \
-vv
```

Role history
----------------
| date  | who | changelog |
|---|---|---|
|2020-07-08  | Patrick Hermann | intial commit for this role in codehub
|2020-10-12   | Patrick Hermann | Updated for using of ansible collections, fixed role structure, added ability of generation self signed certs

License
-------

BSD

Author Information
------------------

Patrick Hermann (patrick.hermann@sva.de), SVA GmbH, 07/2020.

this role is heavily influenced by [podman-container-systemd](https://github.com/ikke-t/podman-container-systemd) - we didnt like the style of the role and for a better understanding for podman from a technology side, we're still in the process to rewrite/implement a similar role. 
