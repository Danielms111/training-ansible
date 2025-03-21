# Documentación del Proyecto - Despliegue de Super Mario con Ansible y Docker

Este proyecto utiliza **Ansible** para automatizar la instalación de Docker y la ejecución de un contenedor con el juego **Super Mario Bros** en una máquina virtual de Azure.

## **Estructura del Proyecto**

```
training-ansible/
├── inventory/
│   ├── hosts.ini
├── playbooks/
│   ├── install_docker.yml
│   ├── run_container.yml
├── roles/
│   ├── docker_install/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   ├── docker_container/
│   │   ├── tasks/
│   │   │   ├── main.yml
├── ansible.cfg
```

---

## **1. Configuración de Inventario**
Ubicado en `inventory/hosts.ini`, define los hosts a administrar con Ansible.

```ini
[azure_vm]
ip ansible_user=tu_user ansible_ssh_pass=tu_contraseña
```

> **Nota:** Cambia la IP y credenciales según tu entorno.

---

## **2. Configuración de Ansible**
Ubicado en `ansible.cfg`, configura parámetros clave para la ejecución de Ansible.

```ini
[defaults]
host_key_checking = False
roles_path = ./roles
inventory = ./inventory/hosts.ini
```

---

## **3. Instalación de Docker**
Ubicado en `playbooks/install_docker.yml`, ejecuta el rol `docker_install`.

```yaml
---
- hosts: azure_vm
  become: yes
  roles:
    - docker_install
```

### **Tareas de instalación de Docker**
Ubicado en `roles/docker_install/tasks/main.yml`.

```yaml
- name: Instalar dependencias de Docker
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - software-properties-common
    state: present
    update_cache: yes

- name: Agregar la clave GPG oficial de Docker
  ansible.builtin.apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Agregar el repositorio de Docker
  ansible.builtin.apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
    state: present

- name: Instalar Docker CE
  apt:
    name: docker-ce
    state: present
    update_cache: yes
```

---

## **4. Ejecución del Contenedor Super Mario**
Ubicado en `playbooks/run_container.yml`, ejecuta el rol `docker_container`.

```yaml
---
- hosts: azure_vm
  become: yes
  roles:
    - docker_container
```

### **Tareas para desplegar el contenedor**
Ubicado en `roles/docker_container/tasks/main.yml`.

```yaml
- name: Ejecutar Mario Bros
  docker_container:
    name: supermario-container
    image: "pengbai/docker-supermario:latest"
    state: started
    ports:
      - "8787:8080"
```

---

## **5. Ejecución del Proyecto**

### **1️ Instalar dependencias en la VM**
Ejecuta:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/install_docker.yml
```

### **2️ Desplegar el contenedor de Super Mario**
Ejecuta:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/run_container.yml
```

### **3️ Acceder al juego en el navegador**
Abre en tu navegador:
```
http://13.95.88.126:8787
```
> **Nota:** Reemplaza `13.95.88.126` con la IP pública de tu VM.

---

## **Notas Finales**
- **Verifica la IP pública** de tu VM antes de ejecutar los playbooks.
- **Asegúrate de que el puerto 8787 esté abierto** en las reglas de seguridad de Azure.

Este proyecto permite la ejecución automática de un contenedor con Super Mario en una VM remota usando Ansible.

