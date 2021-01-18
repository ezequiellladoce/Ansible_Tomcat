# Automatización Ansible:
# Despliegue de Webapp (archivo .war) con Tomcat en EC2 Ubuntu 

El objetivo de este repositorio es compartir una Automatización del despliegue de una **Webapp (archivo .war)** en un servidor **Tomcat**  instado en una Instancia **AWS EC2** con Sistema Operativo Ubuntu mediante **Ansible**.

## Descripción

Las partes principales de este Rol son las siguientes:

- En playbook setup.yml definiremos:

  - Los Hosts a los que aplicaremos la configuración.
  - El método de bienvenida
  - El Usuario
  - El rol 
  
```
    ---
    - name: Tomcat deployment playbook
        hosts: Tomcat-servers     # Inventory hosts group / server to act on
        become: yes               # If to escalate privilege
        become_method: sudo       # Set become method
        remote_user: ubuntu       # Update username for remote server
        roles:
            - Tomcat

```

- En el rol tasks definiremos:

  - Instalación de Paquetes de Update
  - Instalación de Dependencias
  - Instalacion de Java
  - Creación de Grupo y Usuario
  - Descarga y descompresión de Tomcat
  - Copia del archivo War
  - Inicio del servicio.

```
---
# tasks file for Tomcat

- name: Update and upgrade apt packages
  become: true
  apt:
    upgrade: yes
    update_cache: yes
    cache_valid_time: 86400 

- name: Ensure the system can use the HTTPS transport for APT.
  stat:
    path: /usr/lib/apt/methods/https
  register: apt_https_transport

- name: Install APT HTTPS transport.
  apt:
    name: "apt-transport-https"
    state: present
    update_cache: yes
  when: not apt_https_transport.stat.exists

- name: Install basic packages
  package:
    name: ['vim','aptitude','bash-completion','tmux','tree','htop','wget','unzip','curl','git']
    state: present
    update_cache: yes

- name: Install Default Java (Debian/Ubuntu)
  apt:
    name: default-jdk
    state: present

- name: Add tomcat group
  group:
    name: tomcat

- name: Add "tomcat" user
  user:
    name: tomcat
    group: tomcat
    home: /usr/share/tomcat
    createhome: no
    system: yes

- name: Download Tomcat
  get_url:
    url: "{{ tomcat_archive_url }}"
    dest: "{{ tomcat_archive_dest }}"

- name: Create a tomcat directory
  file:
    path: /usr/share/tomcat
    state: directory
    owner: tomcat
    group: tomcat

- name: Extract tomcat archive
  unarchive:
    src: "{{ tomcat_archive_dest }}"
    dest: /usr/share/tomcat
    owner: tomcat
    group: tomcat
    remote_src: yes
    extra_opts: "--strip-components=1"
    creates: /usr/share/tomcat/bin

- name: Copy tomcat service file
  template:
    src: templates/tomcat.service.j2
    dest: /etc/systemd/system/tomcat.service
  when: ansible_service_mgr == "systemd"

- name: Copy warfile
  copy:
    src: /home/ANS/warfile/hello.war
    dest: /usr/share/tomcat/webapps/hello.war
    owner: tomcat
    group: tomcat

- name: Start and enable tomcat
  service:
    daemon_reload: yes
    name: tomcat
    state: started

```

## Pre-requisitos 📋

- ANSIBLE SERVER 
  - Ansible
- CUENTA FREE TIER AWS 

## Comenzando 🚀

### Preparamos el ambiente:

1) Creamos cuenta free tier en AWS  https://aws.amazon.com/
2) Creamos una instancia EC2 En AWS t2 micro (free tier) con sistema Operativo Ubuntu (free tier) y guardamos la Key Pair de la instancia.
3) En nuestro servidor Ansible instalamos Ansible.

## Despliegue 📦

### Actualizamos el inventario

  - En nuestro servidor Ansible Editamos el archivo de Ansible hosts:
  
```

  nano /etc/ansible/hosts
  
```
  - Agregamos 

```

[Tomcat-servers]
18.191.141.29 # aca colocamos la ip de nuestro servidor destino 

```
### Clonamos el repositorio

  - En nuestro servidor Ansible clonamos el repositorio:

```

git clone https://github.com/ezequiellladoce/Ansible_Tomcat.git

```

### Ejecutamos el Playbook

```

cd Ansible_Tomcat

ansible-playbook setup.yml -u ubuntu --key-file /home/ubuntu/key/SRV2.pem

```

En donde es la ruta al key pair de la instancia AWS EC2 en al que instalaremos Tomcat y desplegaremos el archivo War

### Comprobamos la instalación

 - Al ejecutar el playbook  obtenemos:

![runplaybook](https://user-images.githubusercontent.com/67485607/104951164-cd5d1100-59a0-11eb-879b-7adfee2ae145.PNG)

 - Comprobamos que el servidor Tomcat funcione correctamente colocando en nuestro Web Browser la ip publica de nuestra instancia en el puerto 8080 Ej: http://18.191.179.14:8080/



 - Comprobamos que el archivo war se haya desplegado correctamente colocando en nuestro web browser la ip publica de nuestra instancia en el puerto 8080 agregando a la url /hello/index.jsp Ej:  http://18.191.179.14:8080/hello/index.jsp
















