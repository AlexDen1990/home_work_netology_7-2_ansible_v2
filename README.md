# Домашнее задание к занятию «Ansible.Часть 2» - Александров Денис

### Оформление домашнего задания

1. Домашнее задание выполните в [Google Docs](https://docs.google.com/) и отправьте на проверку ссылку на ваш документ в личном кабинете.  
1. В названии файла укажите номер лекции и фамилию студента. Пример названия:  Ansible. Часть 2 — Александр Александров.
1. Перед отправкой проверьте, что доступ для просмотра открыт всем, у кого есть ссылка. Если нужно прикрепить дополнительные ссылки, добавьте их в свой Google Docs.

Любые вопросы по решению задач задавайте в чате учебной группы.

---

### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.

*Ответ* <details>

---
```
- name: Download and extract Apache Kafka
  hosts: all
  become: true
  vars:
    kafka_version: 3.6.1
    kafka_scala_version: 2.13
    kafka_download_url: "https://dlcdn.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz"
    kafka_install_dir: /opt/kafka
    kafka_archive_name: "kafka_{{ kafka_scala_version }}-{{ kafka_version }}.tgz"

  tasks:
    - name: Create installation directory
      file:
        path: "{{ kafka_install_dir }}"
        state: directory
        mode: '0755'

    - name: Download Kafka archive
      get_url:
        url: "{{ kafka_download_url }}"
        dest: /tmp/{{ kafka_archive_name }}
        checksum: "sha512:00722ab0a6b954e0006994b8d589dcd8f26e1827c47f70b6e820fb45aa35945c19163b0f188caf0caf976c11f7ab005fd368c54e5851e899d2de687a804a5eb9"  # Added checksum for>

    - name: Extract Kafka archive
      unarchive:
        src: /tmp/{{ kafka_archive_name }}
        dest: "{{ kafka_install_dir }}"
        remote_src: yes
        creates: "{{ kafka_install_dir }}/kafka_{{ kafka_scala_version }}-{{ kafka_version }}/LICENSE"  # Check if extraction already happened
        owner: root
        group: root

    - name: Clean up downloaded archive
      file:
        path: /tmp/{{ kafka_archive_name }}
        state: absent
```
---
![Kafka](https://github.com/user-attachments/assets/2cae9fc5-7a9b-45fd-9121-defa05a24c74)

![Kafka_ls_wm1](https://github.com/user-attachments/assets/afff0b45-d488-48e3-b1e8-e635abebd0f0)

![Kafka_ls_wm2](https://github.com/user-attachments/assets/a511d98b-f2c4-4b71-b228-9f746423c09a)

</details>

2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.

*Ответ* <details>
```
---
- name: Install and configure tuned
  hosts: all
  become: true

  tasks:
    - name: Install tuned package
      package:
        name: tuned
        state: present

    - name: Start and enable tuned service
      systemd:
        name: tuned
        state: started
        enabled: yes
```


![tuned](https://github.com/user-attachments/assets/a70112ad-14a0-462a-896f-ebf1201748d7)

![tuned_wm1_status](https://github.com/user-attachments/assets/537bb963-3501-4da6-8de9-53d92159c928)

![tuned_wm2_status](https://github.com/user-attachments/assets/9a401f93-505d-4d6a-abae-15588a26fb6d)

</details>


3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

*Ответ* <details>
```
---
- name: Change MOTD
  hosts: all
  become: true
  vars:
    motd_message: |
      #########################################
      #                                       #
      #        Welcome to the System!         #
      #                                       #
      #########################################
      This system is managed by Ansible.
      Please be careful!

  tasks:
    - name: Set MOTD content
      copy:
        dest: /etc/motd
        content: "{{ motd_message }}"
        owner: root
        group: root
        mode: '0644'
```

![Motd](https://github.com/user-attachments/assets/52441c96-ae98-417b-a024-d1593acd4881)

![motd_wm1](https://github.com/user-attachments/assets/bc772c3e-1853-4831-a044-326147859390)

![motd_wm2](https://github.com/user-attachments/assets/56397f1e-9374-47f8-adc4-b460cb28b26b)

</details>

### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

*Ответ* <details>
```
---
- name: Change MOTD
  hosts: all
  become: true
  gather_facts: true
  vars:
    motd_header: |
      #########################################
      #                                       #
      #        Welcome to the System!         #
      #                                       #
      #########################################
    motd_footer: |
      This system is managed by Ansible.
      Please be careful!

  tasks:
    - name: Set MOTD content with host info
      copy:
        dest: /etc/motd
        content: |
          {{ motd_header }}
          Hostname: {{ ansible_facts["hostname"] }}
          IP Address: {{ ansible_facts["default_ipv4"]["address"] | default('N/A') }}
          Have a nice day, System Administrator!
          {{ motd_footer }}
        owner: root
        group: root
        mode: '0644'
```
![motd_new](https://github.com/user-attachments/assets/ee19a54c-df20-4ed3-8f25-840ba447ec86)

![motd_wm1_new](https://github.com/user-attachments/assets/7146facd-49ed-4d7f-ac9f-3ccecffdde2e)

![motd_wm2_new](https://github.com/user-attachments/assets/4ceebbf1-ad26-4f8b-83f5-4cb8d53abb48)

</details>

### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/)
3. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
4. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

### Ответ

Ссылка на все плейбуки и роли - https://drive.google.com/file/d/1fxv8pu1FvfMWBoP5BazEsVGMjiOiAjKS/view?usp=drive_link

Содержание файла плейбука.

<details>

```
---
- name: Deploy Apache Web Server
  hosts: all
  become: true
  roles:
    - apache_web
```

</details>

Содержание файла Index.html.j2.

<details>

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Managed Host Info</title>
</head>
<body>
    <h1>System Information for {{ ansible_hostname }}</h1>
    <ul>
        <li><strong>CPU:</strong> {{ ansible_processor_vcpus }} vCPU ({{ ansible_processor[1] | default('N/A') }})</li>
        <li><strong>RAM:</strong> {{ ansible_memtotal_mb }} MB</li>
        <li><strong>First HDD Size:</strong> {{ ansible_devices.sda.size if ansible_devices.sda is defined else ansible_devices.vda.size }}</li>
        <li><strong>IP Address:</strong> {{ ansible_default_ipv4.address }}</li>
    </ul>
</body>
</html>

```
</details>

Установка Apache, замена файла Index.html, перезауск Apache.

<details>

```
---
- name: Install Apache2 package
  apt:
    name: apache2
    state: present
    update_cache: yes

- name: Allow HTTP traffic (Port 80)
  ufw:
    rule: allow
    port: '80'
    proto: tcp

- name: Deploy custom index.html from template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: '0644'
  notify: Restart Apache

- name: Ensure Apache is started and enabled
  service:
    name: apache2
    state: started
    enabled: yes

- name: Wait for Apache to be available and verify status 200
  uri:
    url: "http://{{ ansible_default_ipv4.address }}"
    status_code: 200
    return_content: no
```

![ubuntu_apache](https://github.com/user-attachments/assets/5ed2ed54-81db-421e-afbf-e1d42f201687)

</details>

Задача используется в качестве handlers (обработчиков), которая срабатывает только после изменения конфигурационных файлов, чтобы новые настройки вступили в силу.

<details>

```
---
- name: Restart Apache
  service:
    name: apache2
    state: restarted

```
</details>

На управляющей машине запущен бразуер и осуществлен вход на 2 из управляемых хостов.

<details>

![ubuntu_apache_wm1](https://github.com/user-attachments/assets/246202d7-8344-4ab0-9aaf-382def47a787)

![ubuntu_apache_wm2](https://github.com/user-attachments/assets/eddf2885-f1b2-4873-b623-d64eda6d9237)

</details>

Проверка сайта через curl с ответом 200.
  
<details>

![ubuntu_apache_curl](https://github.com/user-attachments/assets/fae942a3-9a57-4e40-a276-d9290180b6ae)

</details>







