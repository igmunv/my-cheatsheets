# Suricata

Suricata - это IDS/IPS, способная выполнять глубокий анализ пакетов и мониторинг сети.

## Установка и запуск (debian)

`apt install suricata` - установка

`suricata --build-info` - проверяем установку

Основная конфигурация: `/etc/suricata/suricata.yaml`

`sudo suricata -c /etc/suricata/suricata.yaml -i eth0` - запуск на интерфейсе eth0

`sudo suricata -T -c /etc/suricata/suricata.yaml` - запуск в тестовом режиме (только проверка конфигурации, без запуска движка)

`sudo systemctl status suricata` - проверка статуса

`systemctl stop suricata` - выключить

`systemctl restart suricata` - перезапуск

## Логи

Логи хранятся по пути: `/var/log/suricata/`:

- eve.json - хранит все события обнаруженные системой в формате JSON, удобно для интеграции SIEM
- fast.log - простой способ посмотреть оповещения, все хранится в текстовом виде
- stats.log - содержит логи о работе suricata

## Режимы работы
### IDS
Мониторит трафик, при обнаружении чего-либо создаёт оповещение, не вмешиваясь в сам трафик

`sudo suricata -c /etc/suricata/suricata.yaml -i eth0` - запуск в режиме IDS

Убедитесь, что в конфигурации `/etc/suricata/suricata.yaml` все настроено правильно:

- Настройка интерфейсов
  ```
    af-packet:
    - interface: eth0
      threads: auto
      cluster-id: 99
      cluster-type: cluster_flow
      defrag: yes
  ```
- Настройка выходных форматов:
  ```
    outputs:
      - eve-log:
          enabled: yes
          filetype: regular
          filename: eve.json
          types:
            - alert:
                payload: yes
                payload-printable: yes
                packet: yes
                http-body: yes
                http-body-printable: yes
                metadata: yes
            - http:
                extended: yes
            - dns:
                query: yes
                answer: yes
            - tls:
                extended: yes
  ```
### IPS
### NFQ
