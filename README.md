# homework20

1. У меня дома уже настроен zabbix, снимает метрики с сети

![Снимок экрана 2022-05-21 в 08 07 41](https://user-images.githubusercontent.com/98701086/169636398-131f2d05-3935-49b7-9c37-ab0fdb95fd4f.png)

2. Метрики с гипервизора ol8 (oracle linux+kvm), на котором крутится контроллер unifi, zabbix и машинка на fedore для выполнения домашних заданий.

![Снимок экрана 2022-05-21 в 08 13 29](https://user-images.githubusercontent.com/98701086/169636621-5f27e596-710c-4564-bb0c-bc9704c97c7c.png)

![Снимок экрана 2022-05-21 в 08 13 51](https://user-images.githubusercontent.com/98701086/169636631-0a9e03e9-f10d-4beb-99a2-e46b55873093.png)

![Снимок экрана 2022-05-21 в 08 27 59](https://user-images.githubusercontent.com/98701086/169637006-13ddfa55-30bf-490f-a47b-b4f916ba22e6.png)

3. Но для опыта надо поднять стенд prometheus + grafana. Для это создам две виртуальные машины по приложенному Vagrantfile
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

6. Для тестового стенда firewalld можно не настраивать а просто остановить. После проверяю достпность веб интерфейса сервера prometheus и node-exporter

![Снимок экрана 2022-05-21 в 08 43 53](https://user-images.githubusercontent.com/98701086/169637583-6a69800f-5e11-4f5d-ba4b-a919ea57144a.png)

![Снимок экрана 2022-05-21 в 08 42 27](https://user-images.githubusercontent.com/98701086/169637592-4f202eb8-4351-4dfe-8ea9-c944bd25a35c.png)

7. Prometheus работает, node-exporter работает
8. Теперь на машине monclient установлю из репозитория node-exporter и запущу его 

```
[root@monclient ~]# dnf install golang-github-prometheus-node-exporter
...
Installed:
  golang-github-prometheus-node-exporter-1.3.1-4.el8.x86_64

Complete!
[root@monclient ~]# systemctl start prometheus-node-exporter
[root@monclient ~]# systemctl status prometheus-node-exporter
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
   Loaded: loaded (/usr/lib/systemd/system/prometheus-node-exporter.service; di>
   Active: active (running) since Fri 2022-05-20 19:32:29 UTC; 6s ago
     Docs: https://github.com/prometheus/node_exporter
 Main PID: 26247 (prometheus-node)
    Tasks: 5 (limit: 4951)
   Memory: 8.0M
   CGroup: /system.slice/prometheus-node-exporter.service
           └─26247 /usr/bin/prometheus-node-exporter
...
[root@monclient ~]# systemctl stop firewalld.service
```

9. В конфиг prometheus добавлю параметры для monclient
```
[root@monserv ~]# vim /etc/prometheus/prometheus.yml
...
  - job_name: monclient
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.56.151:9100']
...
```
10. перезапущу prometheus и проверю, что все target на месте

```
[root@monserv ~]# systemctl restart prometheus.service
```

![Снимок экрана 2022-05-21 в 08 51 45](https://user-images.githubusercontent.com/98701086/169637802-09b3db71-37d7-4a3b-979a-35be6a9704e5.png)

11. Все готово для установки grafana. Качаю, устанавливаю и запускаю.

```
[root@monserv prometheus]# wget https://dl.grafana.com/oss/release/grafana-8.5.3-1.x86_64.rpm
--2022-05-20 19:40:14--  https://dl.grafana.com/oss/release/grafana-8.5.3-1.x86_64.rpm
Resolving dl.grafana.com (dl.grafana.com)... 151.101.38.217, 2a04:4e42:9::729
Connecting to dl.grafana.com (dl.grafana.com)|151.101.38.217|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 77331470 (74M) [application/x-redhat-package-manager]
Saving to: 'grafana-8.5.3-1.x86_64.rpm'

grafana-8.5.3-1.x86_64.rpm     100%[====================================================>]  73.75M  11.0MB/s    in 6.9s

2022-05-20 19:40:22 (10.8 MB/s) - 'grafana-8.5.3-1.x86_64.rpm' saved [77331470/77331470]

[root@monserv prometheus]# dnf install grafana-8.5.3-1.x86_64.rpm
...
Installed:
  fontconfig-2.13.1-4.el8.x86_64                            fontpackages-filesystem-1.44-22.el8.noarch
  grafana-8.5.3-1.x86_64                                    libICE-1.0.9-15.el8.x86_64
  libSM-1.2.3-1.el8.x86_64                                  libX11-1.6.8-5.el8.x86_64
  libX11-common-1.6.8-5.el8.noarch                          libXau-1.0.9-3.el8.x86_64
  libXcursor-1.1.15-3.el8.x86_64                            libXext-1.3.4-1.el8.x86_64
  libXfixes-5.0.3-7.el8.x86_64                              libXi-1.7.10-1.el8.x86_64
  libXinerama-1.1.4-1.el8.x86_64                            libXmu-1.1.3-1.el8.x86_64
  libXrandr-1.5.2-1.el8.x86_64                              libXrender-0.9.10-7.el8.x86_64
  libXt-1.1.5-12.el8.x86_64                                 libXxf86misc-1.0.4-1.el8.x86_64
  libXxf86vm-1.1.4-9.el8.x86_64                             libfontenc-1.1.3-8.el8.x86_64
  libmcpp-2.7.2-20.el8.x86_64                               libxcb-1.13.1-1.el8.x86_64
  mcpp-2.7.2-20.el8.x86_64                                  urw-base35-bookman-fonts-20170801-10.el8.noarch
  urw-base35-c059-fonts-20170801-10.el8.noarch              urw-base35-d050000l-fonts-20170801-10.el8.noarch
  urw-base35-fonts-20170801-10.el8.noarch                   urw-base35-fonts-common-20170801-10.el8.noarch
  urw-base35-gothic-fonts-20170801-10.el8.noarch            urw-base35-nimbus-mono-ps-fonts-20170801-10.el8.noarch
  urw-base35-nimbus-roman-fonts-20170801-10.el8.noarch      urw-base35-nimbus-sans-fonts-20170801-10.el8.noarch
  urw-base35-p052-fonts-20170801-10.el8.noarch              urw-base35-standard-symbols-ps-fonts-20170801-10.el8.noarch
  urw-base35-z003-fonts-20170801-10.el8.noarch              xorg-x11-font-utils-1:7.5-41.el8.x86_64
  xorg-x11-server-utils-7.7-27.el8.x86_64

Complete!
[root@monserv prometheus]# systemctl start grafana-server.service
[root@monserv prometheus]# systemctl status grafana-server.service
● grafana-server.service - Grafana instance
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-05-20 19:41:55 UTC; 7s ago
     Docs: http://docs.grafana.org
 Main PID: 27372 (grafana-server)
    Tasks: 10 (limit: 4951)
   Memory: 118.3M
   CGroup: /system.slice/grafana-server.service
           └─27372 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid>

May 20 19:41:55 monserv grafana-server[27372]: logger=plugin.manager t=2022-05-20T19:41:55+0000 lvl=info msg="Plugin regist>
May 20 19:41:55 monserv grafana-server[27372]: logger=plugin.finder t=2022-05-20T19:41:55+0000 lvl=warn msg="Skipping findi>
May 20 19:41:55 monserv grafana-server[27372]: logger=query_data t=2022-05-20T19:41:55.05+0000 lvl=info msg="Query Service >
May 20 19:41:55 monserv grafana-server[27372]: logger=live.push_http t=2022-05-20T19:41:55.06+0000 lvl=info msg="Live Push >
May 20 19:41:55 monserv grafana-server[27372]: logger=server t=2022-05-20T19:41:55.23+0000 lvl=info msg="Writing PID file" >
May 20 19:41:55 monserv systemd[1]: Started Grafana instance.
May 20 19:41:55 monserv grafana-server[27372]: logger=http.server t=2022-05-20T19:41:55.24+0000 lvl=info msg="HTTP Server L>
May 20 19:41:55 monserv grafana-server[27372]: logger=ngalert t=2022-05-20T19:41:55.24+0000 lvl=info msg="warming cache for>
May 20 19:41:55 monserv grafana-server[27372]: logger=ngalert.multiorg.alertmanager t=2022-05-20T19:41:55.24+0000 lvl=info >
May 20 19:41:55 monserv grafana-server[27372]: logger=grafanaStorageLogger t=2022-05-20T19:41:55.25+0000 lvl=info msg="stor>
```

12. Теперь в веб grafana добавляю мой источник prometeus (с localhost)

![Снимок экрана 2022-05-21 в 08 57 08](https://user-images.githubusercontent.com/98701086/169638032-f702a973-811c-483a-9242-aa0f3f33bf73.png)

13. Импортирую готовый дашбоард с id = 1860
14. И собсвенно все, стенд готов

![Снимок экрана 2022-05-21 в 08 59 49](https://user-images.githubusercontent.com/98701086/169638073-35a27102-ccd6-4609-b9a8-da8255f1872f.png)

![Снимок экрана 2022-05-21 в 09 00 00](https://user-images.githubusercontent.com/98701086/169638076-d0582191-d0af-4397-8e1f-ad9d0b7054a0.png)

