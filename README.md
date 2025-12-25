

Проектная работа 6 (HW-03)
Условия:
```
1. Создать 3 ВМ в Я.Облаке (минимальная конфигурация: 2vCPU, 2GB RAM, 20 GB HDD): vm1 и vm2, vm3
2. Установить на vm1 Ansible.
3. Создать на vm1 пользователя для Ansible.
4. Настроить авторизацию по ключу для этого пользователя с vm1 на vm2 и vm3.
5. Добавить в inventory информацию о созданных машинах. vm2 и vm3 должны быть в группе app, vm1 — в группе database и web.
6. Написать плейбук, реализующий следующее:
	- на машинах группы app выполняется установка и запуск Docker;
	- на машинах группы database выполняется установка и запуск postgresql-server (версия и data-директория должны быть переменными, задающимися в inventory).
7. Протестировать написанный плейбук.
8. Выложить плейбук и inventory в GitHub. Создайте отдельный репозиторий Ansible.
9. Прислать ментору ссылку на репозиторий с плейбуком.
```



Структура:

<img width="230" height="107" alt="image" src="https://github.com/user-attachments/assets/24fcdfe1-9acf-4924-9e68-a78da3039ad9" />

Результат: 
<img width="952" height="804" alt="image" src="https://github.com/user-attachments/assets/ef3f337e-78f5-4c5e-9e55-6246f041a855" />

<img width="552" height="116" alt="image" src="https://github.com/user-attachments/assets/bd6944b4-4c49-4394-af0d-89d424c2128d" />



Решение: 
Создать пользователя 
```
useradd -m -s /bin/bash ansible
passwd ansible
su - ansible
ssh-keygen -t ed25519
копируем ключи на vm2 и vm3
Так же на vm2 и vm3 создаём пользователя и даём ему права 
```

с vm1 проверяем 
```
ssh ansible@158.160.85.226
ssh ansible@158.160.86.218
```


Создаем файл inventory.ini
```
[app]
vm2 ansible_host=158.160.85.226
vm3 ansible_host=158.160.86.218

[database]
vm1 ansible_host=158.160.93.118

[web]
vm1 ansible_host=158.160.93.118

[all:vars]
ansible_user=ansible
ansible_python_interpreter=/usr/bin/python3

[database:vars]
postgres_version=16
postgres_data_dir=/var/lib/postgresql/16/main


```

Создаём playbook

```
---
- name: Install and start Docker on app servers
  hosts: app
  become: true
  tasks:
    - name: Install required packages
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Enable and start Docker
      systemd:
        name: docker
        enabled: yes
        state: started


- name: Install and start PostgreSQL on database servers
  hosts: database
  become: true
  tasks:
    - name: Install PostgreSQL
      apt:
        name: "postgresql-{{ postgres_version }}"
        update_cache: yes
        state: present

    - name: Ensure PostgreSQL service is running
      systemd:
        name: postgresql
        enabled: yes
        state: started

```

Проверяем 
```
ansible all -i inventory.ini -m ping
```


Запускаем 
```
ansible-playbook -i inventory.ini playbook.yml
```



На vm2 vm3 - Docker
docker --version
systemctl status docker
на vm1 - Postgresql
psql --version
systemctl status postgresql
