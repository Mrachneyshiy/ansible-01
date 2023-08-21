# Домашнее задание к занятию 1 «Введение в Ansible»

## Подготовка к выполнению

1. Установите Ansible версии 2.10 или выше.
#### Пометка:

Я к сожалению не смог на свой RedOS установить Ansible версии 2.10 или выше. Смог установить только вот такой:

```bush
[skvorchenkov@localhost ~]$ ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/skvorchenkov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.8/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.2 (default, Apr 14 2023, 00:00:00) [GCC 8.3.1 20191121 (Red Hat 8.3.1-6)]

```   

2. Создайте свой публичный репозиторий на GitHub с произвольным именем.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть

#### 1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте значение, которое имеет факт `some_fact` для указанного хоста при выполнении playbook.
```bush
[skvorchenkov@localhost playbook]$ ansible-playbook site.yml -i inventory/test.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
[WARNING]: Platform linux on host localhost is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change this. See https://docs.ansible.com/ansible/2.9/referen
ce_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Print OS] ****************************************************************
ok: [localhost] => {
    "msg": "REDOS"
}

TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```  

#### 2. Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на `all default fact`.

```bush
cat group_vars/all/examp.yml
---
  some_fact: "all default fact"
``` 

#### 3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.

```bush
[skvorchenkov@localhost playbook]$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
7859dc955768   centos    "/bin/bash"   About a minute ago   Up About a minute             centos
675520656dd7   ubuntu    "/bin/bash"   7 minutes ago        Up 6 minutes                  ubuntu
```

#### 4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.

```bush
[skvorchenkov@localhost playbook]$ sudo ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] **********************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************
ok: [centos]
ok: [ubuntu]

TASK [Print OS] ****************************************************************************************************************************
ok: [centos] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **************************************************************************************************************************
ok: [centos] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *********************************************************************************************************************************
centos                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
#### 5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились значения: для `deb` — `deb default fact`, для `el` — `el default fact`.

```bush
[skvorchenkov@localhost playbook]$ cat group_vars/{deb,el}/*

  some_fact: "deb default fact"
  some_fact: "el default fact"
```
6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```bush
[skvorchenkov@localhost playbook]$ sudo ansible-playbook -i inventory/prod.yml site.yml
[sudo] пароль для skvorchenkov: 

PLAY [Print os facts] **********************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************
ok: [ubuntu]
ok: [centos]

TASK [Print OS] ****************************************************************************************************************************
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos] => {
    "msg": "CentOS"
}

TASK [Print fact] **************************************************************************************************************************
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [centos] => {
    "msg": "el default fact"
}

PLAY RECAP *********************************************************************************************************************************
centos                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
#### 7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.

```bush
[skvorchenkov@localhost playbook]$ ansible-vault encrypt group_vars/deb/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
[skvorchenkov@localhost playbook]$ ansible-vault encrypt group_vars/el/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
[skvorchenkov@localhost playbook]$ cat group_vars/{deb,el}/*
$ANSIBLE_VAULT;1.1;AES256
36343464313335383366623061623435656363666561636439396332336661653364393164353731
3863626133393136636332316136613432343063386564350a653839643762333865333963383238
65353135323863623738373830373239356266626137633638313337313531626264373639333131
3862353630373731320a656363613862613237333365343434333861636261333930363839396263
64666532623236303733626430666434613237343735306434663732373734313032353631613033
3235363433323762376563353133373964386630643333633334
$ANSIBLE_VAULT;1.1;AES256
66366464393561613464393033646439663334656231643332656362303030613261353537356438
3562343564366563636263393437616464626133386163320a303730393762623639333333303737
64643035343537343336313333623566373064653338613539646633643333336439396266393464
6437346237646266360a353530323831343737396139336232633361353761613133356564316164
36323136306231333966383933393936626565353938633662666334323138633438323264353361
3865383339396631616462363935626132316638396463316661
```
#### 8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.

```bush
[skvorchenkov@localhost playbook]$ sudo ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] **********************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************
ok: [ubuntu]
ok: [centos]

TASK [Print OS] ****************************************************************************************************************************
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos] => {
    "msg": "CentOS"
}

TASK [Print fact] **************************************************************************************************************************
ok: [centos] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *********************************************************************************************************************************
centos                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
#### 9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.

Запустил комманду ansible-doc -t connection -l и выбрал из списка плагин local.

```bush
[skvorchenkov@localhost playbook]$ ansible-doc -t connection local
> LOCAL    (/usr/lib/python3.8/site-packages/ansible/plugins/connection/local.p>

        This connection plugin allows ansible to execute tasks on the
        Ansible 'controller' instead of on a remote host.

  * This module is maintained by The Ansible Community
NOTES:
      * The remote user is ignored, the user with which the
        ansible CLI was executed is used instead.


AUTHOR: ansible (@core)
        METADATA:
          status:
          - preview
          supported_by: community
```
#### 10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.

```bush
[skvorchenkov@localhost playbook]$ cat inventory/prod.yml
  el:
    hosts:
      centos:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```
#### 11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.

```bush
[skvorchenkov@localhost playbook]$ sudo ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] ************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [ubuntu]
[WARNING]: Platform linux on host localhost is using the discovered Python interpreter at /usr/bin/python, but future installation of another
Python interpreter could change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information.
ok: [localhost]
ok: [centos]

TASK [Print OS] ******************************************************************************************************************************
ok: [localhost] => {
    "msg": "REDOS"
}
ok: [centos] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ****************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ***********************************************************************************************************************************
centos                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
#### 12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.