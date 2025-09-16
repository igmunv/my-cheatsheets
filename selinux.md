# SELinux

## Установка и запуск
`apt install selinux-basics selinux-policy-default auditd`

У SELinux 3 режима:
- Enforcing - запрет доступа на основании правил политики
- Permissive - ведение лога действий, нарушающих политику, которые в режиме enforcing были бы запрещены
- Disabled - полное отключение

`getenforce` - посмотреть текущий режим

`setenforce [Enforcing или Permissive]` - устновить режим

Отключить SELinux можно только через файл:
`nano /etc/selinux/config` 
`SELINUX=disabled`

Для начала изменим конфигурацию grub, для запуска selinux:

Редактируем файл `/etc/default/grub`:

в строку

`GRUB_CMDLINE_LINUX_DEFAULT="quiet"`

нужно добавить

`GRUB_CMDLINE_LINUX_DEFAULT="quiet selinux=1 security=selinux"`

и обновить конфигурацию

`update-grub`

Для того чтобы начать маркировкку, в файле конфигурации нужно поставить режим permissive:

`nano /etc/selinux/config` 

`SELINUX=permissive`

После нужно в корне создать файл '/.autorelabel'

`touch /.autorelabel`

И перезагружаем компьютер:

`reboot`

После завершения маркировки и загрузки системы можно перейти к файлу конфигурации и установить режим enforcing.

> Используется режим permissive для маркировки, поскольку использование режима enforcing может привести к краху системы во время перезагрузки.
