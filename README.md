**Bitrixdock**
OS: Astra Linux
Docker Rootless
Bitrix(php-mysql-nginx) -Traefik-Memcached

**Getting started**

**1. Install Docker on Server**
   Пользователь на папки с сайтом
```
grep ^$(whoami): /etc/subuid
chown -R 166535:dev1 name_site
sudo apt-get install -y dbus-user-session
```


-) Чтобы предоставить привилегированные порты (< 1024)
```
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
sudo vim /etc/sysctl.conf
kernel.unprivileged_userns_clone=1
net.ipv4.ip_unprivileged_port_start=0
```


-) `sudo apt-get install -y fuse-overlayfs`

-) `sudo modprobe overlay permit_mounts_in_userns=1`

-) Rootless docker requires version of slirp4netns greater than v0.4.0
`slirp4netns --version`

-) Установка rootless:
`curl -fsSL https://get.docker.com/rootless | sh`

\*) Если выходит ошибка:
Aborting because rootful Docker is running and accessible. Set FORCE_ROOTLESS_INSTALL=1 to ignore.

-) `sudo systemctl disable --now docker.service docker.socket`
-) `sudo reboot now`

-) `export PATH=/home/$USER/bin:$PATH`
-) `export DOCKER_HOST=unix:///run/user/$UID/docker.sock`
-) `export XDG_RUNTIME_DIR=/run/user/$UID`

После установки обращаем внимание на строчку:
-) `unix:///run/user/1001/docker.sock`
Запоминаем и учитываем ее когда будем писать docker-compose.yml

При создании контейнеров, на монтированную правку назначать права из вывода команды:
-) `cat /etc/subuid`
dev1:166535:65536
Далее поменяет user:group на папку:
-) `sudo chown -R 166535:65536 folder-site`

```
systemctl --user start docker
systemctl --user enable docker
sudo loginctl enable-linger $(whoami)

sudo apt-get install docker-compose
```


2. `git clone https://git.ict29.ru/filimonov.io/bitrixdock.git`
3. `cd /bitrixdock/`

## Install

1. Traefik install
  ```
 cd /bitrixdock/traefik
   docker compose up -d
```

2. Install docker-Bitrix
  ```
 cd /bitrixdock/bitrix-distr
   cp -r bitrix-distr sites/name_site
   cd sites/name_site
   cp .env_template .env
```

   Edit .env - редактируем где есть знаки вопросов
   `docker compose up -d`
3. Install Bitrix-setup or Bitrix-restore
  ```
 cp /sites/name_site/.env ~/bitrixdock/
   cd ~/bitrixdock/
   make bitrix-setup or make bitrix-restore
```


## Settings
   # Права
    1. Назначаем права на папку с сайтом
```
    sudo chmod -R 775 name_site
    sudo chown -R 166535:dev1 name_site
```
   2. Выполняем установку ACL
```
sudo apt install acl
```
   3. Назначаем дополнительные права на папку сайта
```
sudo setfacl -Rm d:u:dev1:rwX,u:dev1:rwX ./data
```
   # Настройка почты
   Настроенная почта находится в папке php81. Следует редактировать файл msmtprc и указать почтовые настройки под сайт.
   Важно создать файл msmtprc в папке config/msmtp и назначить пользователя 
  
  ```
   sudo chown 166535:165568
   sudo chmod 600.
   ```

   # Выполнение по crontab
   Пока данный функционал не настроен грамотно, но есть костыльное решение:
   Из хост машины выполняем тестово данную команду:
   ```
   docker exec 1eed12a01186 php /var/www/bitrix/sendData.php
   ```
Где 1eed12a01186 это ID контейнера где находится у нас сайт,
php /var/www/bitrix/sendData.php - далее выполнить команду в контейнере

Далее в crontab заносим обязательно через команду, где dev01 имя пользователя в системе 
  ```
sudo crontab -u dev01 -e
  ```
  ```
0 9,17  *   *   *  docker exec 1eed12a01186 php /var/www/bitrix/sendData.php
  ```
