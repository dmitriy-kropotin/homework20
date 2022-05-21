# homework20

1. У меня дома уже настроен zabbix, снимает метрики с сети

![Снимок экрана 2022-05-21 в 08 07 41](https://user-images.githubusercontent.com/98701086/169636398-131f2d05-3935-49b7-9c37-ab0fdb95fd4f.png)

2. Метрики с гипервизора ol8 (oracle linux+kvm), на котором крутится контроллер unifi, zabbix и машинка на fedore для выполнения домашних заданий.

![Снимок экрана 2022-05-21 в 08 13 29](https://user-images.githubusercontent.com/98701086/169636621-5f27e596-710c-4564-bb0c-bc9704c97c7c.png)

![Снимок экрана 2022-05-21 в 08 13 51](https://user-images.githubusercontent.com/98701086/169636631-0a9e03e9-f10d-4beb-99a2-e46b55873093.png)

![Снимок экрана 2022-05-21 в 08 27 59](https://user-images.githubusercontent.com/98701086/169637006-13ddfa55-30bf-490f-a47b-b4f916ba22e6.png)

3. Но для опыта надо поднять стенд prometheus + grafana. Для это создам де виртуальные машины по приложенному Vagrantfile
4. На машине `monserv` установлю из репозитория epel готовые пакеты `golang-github-prometheus golang-github-prometheus-node-exporter`

```
[root@monserv ~]# dnf install golang-github-prometheus.x86_64
...
Installed:
  golang-github-prometheus-2.32.1-2.el8.x86_64
[root@monserv ~]# golang-github-prometheus-node-exporter
Installed:
  golang-github-prometheus-node-exporter-1.3.1-4.el8.x86_64
```

5. При устанвоке из пакетов уже есть базовый конфиг, и unit файлы в systemd, поэтмоу просто запускаю службы

```
[root@monserv ~]# systemctl start prometheus-node-exporter
[root@monserv ~]# systemctl status prometheus-node-exporter
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
   Loaded: loaded (/usr/lib/systemd/system/prometheus-node-exporter.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-05-20 19:14:08 UTC; 10h ago
     Docs: https://github.com/prometheus/node_exporter
 Main PID: 26580 (prometheus-node)
    Tasks: 7 (limit: 4951)
   Memory: 33.1M
   CGroup: /system.slice/prometheus-node-exporter.service
           └─26580 /usr/bin/prometheus-node-exporter
[root@monserv ~]# systemctl start prometheus.service
[root@monserv ~]# systemctl status prometheus
● prometheus.service - Monitoring system and time series database
   Loaded: loaded (/usr/lib/systemd/system/prometheus.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-05-20 19:38:44 UTC; 10h ago
     Docs: https://prometheus.io/docs/introduction/overview/
           man:prometheus(1)
 Main PID: 26704 (prometheus)
    Tasks: 8 (limit: 4951)
   Memory: 198.0M
   CGroup: /system.slice/prometheus.service
           └─26704 /usr/bin/prometheus

```

6. Для тестового стенда firewalld можно не настраивать а просто остановить. После проверяю достпность веб интерфейса сервера prometheus и node

![Снимок экрана 2022-05-21 в 08 43 53](https://user-images.githubusercontent.com/98701086/169637583-6a69800f-5e11-4f5d-ba4b-a919ea57144a.png)

![Снимок экрана 2022-05-21 в 08 42 27](https://user-images.githubusercontent.com/98701086/169637592-4f202eb8-4351-4dfe-8ea9-c944bd25a35c.png)




