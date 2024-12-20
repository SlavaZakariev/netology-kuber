## 22.12 Установка Kubernetes - Вячеслав Закариев

### Цель задания

Установить кластер K8s.

### Чеклист готовности к домашнему заданию

1. Развёрнутые ВМ с ОС Ubuntu 20.04-lts.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция по установке kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).
2. [Документация kubespray](https://kubespray.io/).

---

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.

---

### Дополнительные задания (со звёздочкой)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.** Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой необязательные к выполнению и не повлияют на получение зачёта по этому домашнему заданию. 

### Задание 2*. Установить HA кластер

1. Установить кластер в режиме HA.
2. Использовать нечётное количество Master-node.
3. Для cluster ip использовать keepalived или другой способ.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl get nodes`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Разворачиваем 6 ВМ:
   - Сервер IaaC - 192.168.0.100
   - Мастер-01 - 192.168.0.50
   - Воркер-01 - 192.168.0.51
   - Воркер-02 - 192.168.0.52
   - Воркер-03 - 192.168.0.53
   - Воркер-04 - 192.168.0.54

![hv](https://github.com/SlavaZakariev/netology-kuber/blob/f8173efd018be67b4410d0741375042d76f4c9e3/3.2/resources/kub_3-2_1.1.jpg)

2. На сервер IaaC устанавливаем Ansible и раскидываем на все узлы SSH, проверяем связь с узлами

![ping](https://github.com/SlavaZakariev/netology-kuber/blob/f8173efd018be67b4410d0741375042d76f4c9e3/3.2/resources/kub_3-2_1.2.jpg)

3. Подготавливаем с помощью Ansible узлы Мастер и Воркеры:

```yaml
---
- name: pre-preparation of nodes before installation Kubernetes
  hosts: "all_k8s"
  become: yes
  roles:
  - 01_ufw            # Роль для FireWall
  - 02_Swapoff        # Роль для отключения файла подкачки
  - 03_Timezone       # Установка часового пояса МСК
  - 04_Dependent_soft # Установка зависимостей
```

- Настраиваем **FireWall** - [roles/01_ufw](https://github.com/SlavaZakariev/netology-kuber/blob/main/3.2/yaml/01_ufw/main.yaml)
- Отключаем файл подкачки **Swapoff** - [roles/02_Swapoff](https://github.com/SlavaZakariev/netology-kuber/blob/main/3.2/yaml/02_Swapoff/main.yml)
- Настраиваем часовой пояс на узлах - [roles/03_Timezone](https://github.com/SlavaZakariev/netology-kuber/blob/main/3.2/yaml/03_Timezone/main.yml)
- Устанавливаем необходимые зависимости - [roles/04_Dependent_soft](https://github.com/SlavaZakariev/netology-kuber/blob/main/3.2/yaml/04_Dependent_soft/tasks/main.yml)

<details>
<summary>Результат выполнения сборника</summary>

```yaml
ansible@iaac:/opt/ansible/k8s$ ansible-playbook main.yml

PLAY [pre-preparation of nodes before installation Kubernetes] **************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [master-01]
ok: [worker-01]
ok: [worker-02]
ok: [worker-03]
ok: [worker-04]

TASK [01_ufw : Enable UFW] **************************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Allow ssh] ***************************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Kubernetes API Server] ***************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : etcd server client API] **************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Kubelet API] *************************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Kube-scheduler] **********************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Kube-controller-manager] *************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Kube-proxy] **************************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [01_ufw : Ports for k8s workers] ***************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [02_Swapoff : Disable SWAP (1/2)] **************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [02_Swapoff : Disable SWAP (2/2)] **************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [03_Timezone : Set timezone] *******************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [04_Dependent_soft : Update cache] *************************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

TASK [04_Dependent_soft : install dependencies] *****************************************************************************
changed: [master-01]
changed: [worker-01]
changed: [worker-02]
changed: [worker-03]
changed: [worker-04]

PLAY RECAP ******************************************************************************************************************
master-01                  :ok=15   changed=13   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
worker-01                  :ok=15   changed=13   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
worker-02                  :ok=15   changed=13   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
worker-03                  :ok=15   changed=13   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
worker-04                  :ok=15   changed=13   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
```
   
</details>

4. Проверяем наличие python и pip

![pip](https://github.com/SlavaZakariev/netology-kuber/blob/3f6e3b5b76f6e44d064ad02531317c93a32029c0/3.2/resources/kub_3-2_1.3.jpg)

5. Клонируем git к нам на ВМ IaaC

![clone](https://github.com/SlavaZakariev/netology-kuber/blob/3f6e3b5b76f6e44d064ad02531317c93a32029c0/3.2/resources/kub_3-2_1.4.jpg)

6. Переходим в папку **kubespray** и устанавливаем рекомендации из файла **requirements.txt**

![txt](https://github.com/SlavaZakariev/netology-kuber/blob/3f6e3b5b76f6e44d064ad02531317c93a32029c0/3.2/resources/kub_3-2_1.5.jpg)

7. Обновляем файл инвентаризации с помощью **inventory builder**

![invent](https://github.com/SlavaZakariev/netology-kuber/blob/3f6e3b5b76f6e44d064ad02531317c93a32029c0/3.2/resources/kub_3-2_1.6.jpg)

8. Проверяем сформированный файл инвентаризации по пути `~/kubespray/inventory/mycluster/hosts.yaml`

```yaml
all:
  hosts:
    node1:
      ansible_host: 192.168.0.50
      ip: 192.168.0.50
      access_ip: 192.168.0.50
    node2:
      ansible_host: 192.168.0.51
      ip: 192.168.0.51
      access_ip: 192.168.0.51
    node3:
      ansible_host: 192.168.0.52
      ip: 192.168.0.52
      access_ip: 192.168.0.52
    node4:
      ansible_host: 192.168.0.53
      ip: 192.168.0.53
      access_ip: 192.168.0.53
    node5:
      ansible_host: 192.168.0.54
      ip: 192.168.0.54
      access_ip: 192.168.0.54
  children:
    kube_control_plane:
      hosts:
        node1:
    kube_node:
      hosts:
        node2:
        node3:
        node4:
        node5:
    etcd:
      hosts:
        node1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

9. Запускаем установку кластера Kubernetes \
`ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml -v`

![install](https://github.com/SlavaZakariev/netology-kuber/blob/d12f5a5ac73f7f7d5cf29d0233b03b1d35559829/3.2/resources/kub_3-2_1.7.jpg)

10. Подключаемся на мастер, создаём папку в корне пользователя, копируем конфиг и меняем владельца для полного доступа

![dir](https://github.com/SlavaZakariev/netology-kuber/blob/42ae0596c23ed5664f5fb644db5ba9d7570439a1/3.2/resources/kub_3-2_1.8.jpg)

11. Проверяем все узлы кластера на целостность

![nodes](https://github.com/SlavaZakariev/netology-kuber/blob/42ae0596c23ed5664f5fb644db5ba9d7570439a1/3.2/resources/kub_3-2_1.9.jpg)

12. Проверяем статус всех подов кластера

![podes](https://github.com/SlavaZakariev/netology-kuber/blob/42ae0596c23ed5664f5fb644db5ba9d7570439a1/3.2/resources/kub_3-2_1.10.jpg)
