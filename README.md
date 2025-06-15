### ОГЛАВЛЕНИЕ

[МОДУЛЬ 1](#Модуль-1)                       |  [МОДУЛЬ 2](#Модуль-2)                      |  [МОДУЛЬ 3](#Модуль-3)

# Модуль 1 
### 1. Базовая настройка устройств 
Включает в себя: наименование устройств полным доменным именем;

```
hostnamectl set-hostname имя_устройства; exec bash
```

конфигурацию IPv4 согласно схеме:

| Имя устройства | IP-адрес | Шлюз по умолчанию |
| -------------- | -------- | ----------------- |
| ISP to HQ | 172.16.4.1/28 | - |
| ISP to BR | 172.16.5.1/28 | - |
| HQ-RTR | 172.16.4.2/28 | 172.16.4.1 |
| BR-RTR | 172.16.5.2/28 | 172.16.5.1 |
| HQ in | 172.16.0.1/28 | - |
| BR in | 172.16.6.1/27 | - |
| HQ-SRV | 172.16.0.2/28 | 172.16.0.1 |
| BR-SRV | 172.16.6.2/27 | 172.16.6.1 |
| HQ-CLI | 172.16.0.3/28 | 172.16.0.1 |

Заносим в отчет необходимые данные. Проверяем сетевую связанность с помощью ping.

### 2. Создание локальных учетных записей 
На серверах HQ-SRV и BR-SRV: пользователь sshuser, пароль P@ssw0rd, id 1010

```
useradd -m -u 1010 sshuser
passwd sshuser
nano /etc/sudoers
sshuser ALL=(ALL:ALL)NOPASSWD:ALL
ctrl+x
y
enter
usermod -aG wheel sshuser
```

На роутерах HQ-RTR и BR-RTR: пользователь net_admin, пароль P@$$word

```
useradd -m net_admin
passwd net_admin
nano /etc/sudoers
net_admin ALL=(ALL:ALL)NOPASSWD:ALL
ctrl+x
y
enter
usermod -aG wheel net_admin
```

### 3. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV:
```
nano /etc/mybanner
в файл: Authorized access only!
ctrl+x
y
enter

nano /etc/openssh/sshd_config
port 2024
Banner /etc/mybanner
MaxAuthTries 2
AllowUsers sshuser
ctrl+x
y
enter

systemctl restart sshd.service
```

### 4. Между офисами HQ и BR необходимо сконфигурировать ip туннель
Перед настройкой: проверяем отключен ли на всех роутерах firewall, если нет - отключаем. Включаем на всех роутерах ip forwarding.

```
nano /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
```

Настраиваем GRE через nmtui

HQ-RTR:

