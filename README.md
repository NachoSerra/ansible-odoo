# Ansible - Odoo

Instalación de odoo y localización española mediante ansible.

### Configuración inicial
* Instalar ansible: `pip install ansible`
* Copiar el fichero **ansible.cfg** del repositorio a la ruta /etc/ansible
* Crear el fichero **hosts** en la ruta /etc/ansible

Ejemplo de fichero hosts:

Para local:

`localhost ansible_connection=local`

Para servidor remoto:

`servidorDestino ansible_host=192.168.0.35 ansible_user=root ansible_port=22`

### Ejecutar receta
El repositorio viene con este ejemplo de receta:
```yaml
- name: Odoo
  hosts: all
  become: yes
  roles:
    - ansible-odoo
  vars:
    - odoo_version: 8.0
    - odoo_config_admin_passwd: admin
    - odoo_repo_url2: https://github.com/OCA/l10n-spain.git
    - odoo_repo_type2: git
    - odoo_repo_rev2: 8.0
    - odoo_repo_dest2: "/home/{{ odoo_user }}/odoo/l10n-spain"
    - odoo_config_addons_path:
        - "/home/{{ odoo_user }}/odoo/server/openerp/addons"
        - "/home/{{ odoo_user }}/odoo/server/addons"
        - "/home/{{ odoo_user }}/odoo/l10n-spain"
```
Si quisieramos seleccionar el host donde se va a ejecutar cambiaremos _all_, por la etiqueta que pusieramos en el fichero de configuración **hosts**, en nuestro caso podría ser _localhost_ o _servidorDestino_

En _vars_ podemos añadir o modificar las variables que queramos, ya sea para cambiar la version de odoo que se va a instalar, la contraseña maestra, o el repositorio.
En este caso en la receta de ejemplo se instalaría la versión 8 de Odoo con todas sus dependencias.

Comando para ejecutar receta:

`ansible-playbook playbook.yml`

### Variables disponibles
Mirar [ansible-odoo/defaults/main.yml](ansible-odoo/defaults/main.yml)


### Posibles fallos

#### Problemas de SSH
```bash
UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh.", "unreachable": true}
```

Habrá que comprobar si podemos acceder por ssh al servidor, teniendo la clave pública en la carpeta authorized_keys.

Comprobar también en el archivo de configuración de **hosts** que el argumento ansible_user y ansible_host estan definidos correctamente.

#### Error de python.

Error al resolver dependencia cryptography:
```python
error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
```
Realizar los siguientes comandos en el servidor remoto:

`apt-get update`

`apt-get install build-essential libssl-dev libffi-dev python-dev`

Si nos da el siguiente error:
```python
locale.Error: unsupported locale setting
```
Ejecutaremos en la consola
`export LC_ALL=C`


#### Error de postgres al intentar crear una base de datos nueva en Odoo

```python
new encoding (UTF8) is incompatible with the encoding of the template database (SQL_ASCII) HINT: Use the same encoding as in the template database, or use template0 as template.
```
Para solucionarlo ejecutaremos en postgres los siguientes comandos
```sql
UPDATE pg_database SET datistemplate = FALSE WHERE datname = 'template1';
DROP DATABASE template1;
CREATE DATABASE template1 WITH TEMPLATE = template0 ENCODING = 'UTF8';
UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template1';
\c template1
VACUUM FREEZE;
```



### Más información
[Repo original osiell/ansible-odoo](https://github.com/osiell/ansible-odoo/blob/master/README.md)
