## 22.1 "Kubernetes". Причины появления. Команда kubectl - Вячеслав Закариев

### Цель задания

Для экспериментов и валидации ваших решений вам нужно подготовить тестовую среду для работы с Kubernetes. \
Оптимальное решение — развернуть на рабочей машине или на отдельной виртуальной машине MicroK8S.

### Чеклист готовности к домашнему заданию

1. Личный компьютер с ОС Linux или MacOS \
   или
3. ВМ c ОС Linux в облаке либо ВМ на локальной машине для установки MicroK8S  

### Инструкция к заданию

1. Установка MicroK8S:
    - sudo apt update,
    - sudo apt install snapd,
    - sudo snap install microk8s --classic,
    - добавить локального пользователя в группу `sudo usermod -a -G microk8s $USER`,
    - изменить права на папку с конфигурацией `sudo chown -f -R $USER ~/.kube`.

2. Полезные команды:
    - проверить статус `microk8s status --wait-ready`;
    - подключиться к microK8s и получить информацию можно через команду `microk8s command` \
      например, `microk8s kubectl get nodes`;
    - включить addon можно через команду `microk8s enable`; 
    - список addon `microk8s status`;
    - вывод конфигурации `microk8s config`;
    - проброс порта для подключения `microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443`.

3. Настройка внешнего подключения:
    - отредактировать файл /var/snap/microk8s/current/certs/csr.conf.template
    ```shell
    # [ alt_names ]
    # Add
    # IP.4 = 123.45.67.89
    ```
    - обновить сертификаты `sudo microk8s refresh-certs --cert front-proxy-client.crt`.

4. Установка kubectl:

- curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
- `chmod +x ./kubectl`
- `sudo mv ./kubectl /usr/local/bin/kubectl`
- добавление автодополнения в командную оболочку `bash echo "source <(kubectl completion bash)" >> ~/.bashrc`

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Инструкция](https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/#bash) по установке автодополнения **kubectl**.

---

### Задание 1. Установка MicroK8S

1. Установить MicroK8S на локальную машину или на удалённую виртуальную машину.
2. Установить dashboard.
3. Сгенерировать сертификат для подключения к внешнему ip-адресу.

---

### Задание 2. Установка и настройка локального kubectl

1. Установить на локальную машину kubectl.
2. Настроить локально подключение к кластеру.
3. Подключиться к дашборду с помощью port-forward.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода команд `kubectl get nodes` и скриншот дашборда.

### Критерии оценки
1. Зачёт — выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

2. На доработку — задание выполнено частично или не выполнено, в логике заданий есть противоречия, существенные недостатки.

---

### Решение 1

1. Устанавливаем `Microk8s`. Развёрнута ВМ Ubuntu 22.04 LTS на HyperV.

![M8s1](https://github.com/SlavaZakariev/netology-kuber/blob/cd8376c5b4f62ac73a3db41a095e5e0df2aef9f0/1.1/resources/kub_1.1.jpg)

2. Проверка статуса `Microk8s`

![M8s2](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_1.2.jpg)

3. Включение панели `dashboard`

![dash](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_1.3.jpg)

4. Добавление внешнего **IP** в файл конфигурации

![ip](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_1.4.jpg)

5. Обновление сертификатов

![certs](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_1.5.jpg)


---

### Решение 2

1. Установка `kubectl` (Также можно поставить через команду `sudo snap install kubectl`)

![ctl](https://github.com/SlavaZakariev/netology-kuber/blob/791121c5c64ab19413af63d5d728e7b6be999c0d/1.1/resources/kub_2.1.jpg)

Для корректной работы `kubectl`, скопировал конфиг:

```bash
cd $HOME
mkdir .kube
cd .kube
microk8s config > config
```

2. `Token` для аутентификации в `dashboard`

![token](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_2.2.jpg)

3. Пробрасываем порт с локальной командой 'kubectl' без `microk8s` для возможности подключиться к `dashboard`

![port](https://github.com/SlavaZakariev/netology-kuber/blob/63f58388f5bb636f689fd59a4795ee1ab681930b/1.1/resources/kub_2.3.jpg)

4. Подключение к `dashboard` через браузер и аутентификация с помощью полученного `token` (через **https://**)

![auth](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_2.4.jpg)

5. Страница `about` в `dashboard`

![about](https://github.com/SlavaZakariev/netology-kuber/blob/3903f1d5292fdbd5151286d4013e6e81039e0a66/1.1/resources/kub_2.5.jpg)
