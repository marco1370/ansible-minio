## Ansible role for install minio cluster/standalone server

### Parameters

| parameter | value |
|-----------|-------|
| minio_access_key  | random variable, generate with `python -c "import uuid; print str(uuid.uuid4().get_hex().upper()[0:20])"` |
| minio_secret_key | random variable, generate with `python -c "import uuid; print str(uuid.uuid4().get_hex()[0:40])"` |
| minio_volumes | list of nodex and it's export, like http://minio1/export http://minio2/export http://minio3/export http://minio4/export, for standalone server it is just a list of folders  |
| nginx_nodes | list of nginx nodes |


### Cluster install example

Define groups in inventory file:
```
[minio]
minio1
minio2
minio3
minio4

[nginx]
nginx1
```

And install minio cluster and nginx proxy:

```yaml
- hosts: minio
  become: yes
  become_user: root
  roles:
    - ansible-minio
  vars:
    minio_access_key: 66CCE2594A6E44FBAF3D
    minio_secret_key: 1fa894a644a846a1b8e64c17ff3bbee3
    minio_volumes: "{{ groups['minio']|map('extract', hostvars, 'ansible_default_ipv4') \
                          | map(attribute='address') \
                          | map('regex_replace', '^(.*)$', 'http://\\1/export') | list }}"

- hosts: nginx
  become: yes
  become_user: root
  roles:
    - { role: ansible-minio,
        install_nginx: yes,
        install_minio: no,
        nginx_nodes: "{{ groups['minio']|map('extract', hostvars, 'ansible_default_ipv4')|map(attribute='address') | list }}" }
```

Update local minio copy in files

```bash
cd files && curl https://dl.minio.io/server/minio/release/linux-amd64/minio -o minio && tar -cf minio.tar minio && xz -zf minio.tar && rm minio
```