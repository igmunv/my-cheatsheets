# Прозрачное промежуточное устройство
Позволяет сделать так, чтобы некое устройство в сети было посредником между двумя узлами. *Это нужно, к примеру, для мониторинга трафика*.

## Пример
Mikrotik -> **Sniffer** -> Router -> Internet

В данном примере **sniffer** выступает в роли посредника, которые мониторит трафик и выполняет функции IDS/IPS

## Реализация

### Создание моста

```
ip link set dev eth0 down # Отключаем интерфейсы
ip link set dev eth1 down

ip link add name br0 type bridge # Создаём мост

ip link set dev eth0 master br0 # Добавляем интерфейсы в мост
ip link set dev eth1 master br0

ip link set dev eth0 up # Включаем интерфейсы
ip link set dev eth1 up
ip link set dev br0 up
```

Теперь трафик будет проходить 'сквозь', но **sniffer** будет видеть его и сможет им управлять благодаря мосту
