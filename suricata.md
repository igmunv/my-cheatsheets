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
Мониторит трафик и при обнаружении чего-либо пытается предотвратить вторжение

`sudo suricata -c /etc/suricata/suricata.yaml --af-packet -D` - запуск в режиме IPS. Используем механиз af-packet для работы с пакетами

Убедитесь, что в конфигурации `/etc/suricata/suricata.yaml` все настроено правильно:

- Настройка интерфейсов:
  ```
  af-packet:
    - interface: eth0
      threads: auto
      cluster-id: 99
      cluster-type: cluster_flow
      defrag: yes
      use-mmap: yes
      ring-size: 200000
      buffer-size: 64535
      checksum-checks: auto
    ```

- Настройка выходных форматов (такая же как для IDS):
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

### NFQ
Позволяет suricata взаимодействовать с iptables для более гибкого управления трафиком.

Настройка iptables:

`sudo iptables -I FORWARD -j NFQUEUE --queue-num 0`

Направляет весь проходящий трафик (forward) в очередь NFQUEUE с номером 0

`sudo suricata -c /etc/suricata/suricata.yaml --nfqueue --nfqueue-mode=repeat --nfqueue-id=0` - запуск с использованием NFQ. Обрабатываем трафик из очереди NFQUEUE с номером 0


Убедитесь, что в конфигурации `/etc/suricata/suricata.yaml` все настроено правильно:

- Настройка NFQ:
  ```
  nfqueue:
    mode: repeat
    fail-open: yes
    qids: [ 0 ]
    batchcount: 20
    max-pending-packets: 1024
    defrag: yes
  ```

- Настрйока выходных форматов:
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

## Правила

Формат правила:

`<действие> <протокол> <IP-источник> <порт-источник> -> <IP-назначение> <порт-назначение>`

Формат тела:

`(<ключевое слово>:<значение>; ...)`

Пример правила:

`alert http any any -> any any (msg:"Example rule"; content:"example"; sid:1000001; rev:1;)`

Это правило генерирует оповещение для любого пакета с протоколом http, если он содержит слово "example"

### Основные компоненты правила:

1. Действие
   - alert - создает оповещение
   - log - записать информацию
   - pass - пропустить пакет
   - drop - отбросить пакет (только для IPS)
2. Протокол - любой http, tcp, udp, icmp

### Осноны ключеве слова тела правила:

1. msg - описание правила. Отображается в логах
2. content - шаблон для поиска в пакете
3. sid - идентификатор правила. Уникальный
4. rev - версия правила
5. classtype - класс типа атаки
6. priority - приоритет правила
7. flow - определяет направление потока данных (to_server, to_client)
8. pcre - использует регулярные выражения для поиска в трафике
9. threshold - устанока пороговых значений для предотвращения чрезмерного срабатывания правила

### Примеры правил:

1. Оповещение при обнаружении специфического HTTP запроса:

  `alert http any any -> any any (msg:"Detected specific HTTP request"; content:"/admin"; http_uri; sid:1000002; rev:1;)`
  
  это правило реагирует при обнаружении в URI строки `/admin`

2. Оповещение при обнаружении специфического User-Agent в HTTP запросе

   `alert http any any -> any any (msg:"Detected suspicious user-agent"; content:"SuspiciousAgent"; http_header; sid:1000003; rev:1;)`

   это правило реагирует при обнаружении в заголовке HTTP строки `SuspiciousAgent`

4. Оповещение при обнаружении определенного DNS-запроса:

   `alert dns any any -> any any (msg:"Detected DNS request for example.com"; content:"example.com"; sid:1000004; rev:1;)`

   это правило реагирует при обнаружении запроса на домен `example.com`
  
6. Оповещении при обнаружении команды в FTP:

   `alert ftp any any -> any any (msg:"Detected FTP command"; content:"USER admin"; sid:1000005; rev:1;)`

   это правило реагирует при обнаружении команды `USER admin` в FTP трафике

8. Регулярное выражение для поиска паттернов в трафике:

   `alert http any any -> any any (msg:"Detected SQL Injection attempt"; pcre:"/select.+from.+users/i"; sid:1000006; rev:1;)`

   это правило реагирует при обнаружении попыток SQL-инъекции

### Добавляем свои правила:

Можно скачать правила из интернета:

`sudo suricata-update`

Создаём файл с правилами:

`sudo nano /etc/suricata/rules/local.rules`

Обновяем список правил в секции `rule-files` в файле конфигурации `/etc/suricata/suricata.yaml`:

```
rule-files:
  - suricata.rules
  - local.rules
```

### Тестирование правил:



`sudo suricata -T -c /etc/suricata/suricata.yaml -S /path/to/custom.rules` - проверяет файл конфигурации и указанные правила на наличие ошибок без запуска движка

# Ресуры
https://habr.com/ru/articles/825460/
