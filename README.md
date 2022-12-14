# Домашнее задание Резервное копирование
<ol> 
  <li> Создать стенд для выполнения домашнего задания
  <li> Настройка стенда
  <li> Бэкап
  <li> Настройка автоматизации
</ol>  

# 1. Создать стенд для выполнения домашнего задания
<ul>
  <p> Развернуты две VM на базе CentOS-7-x86_64 backup_server и backup_client
</ul> 

# 2. Настройка стенда
<ul>
  <p> Подключил EPEL репозиторий
  <p> yum install epel-release
  <p> Установил на backup_server и backup_client borgbackup
  <p> yum install borgbackup
  <p> На backup_server создал пользователя и каталог
  <p> useradd -m borg
  <p> mkdir /var/backup
  <p> chown borg:borg /var/backup/
  <p> На backup_server создал каталоги
  <p> su - borg 
  <p> mkdir .ssh
  <p> touch .ssh/authorized_keys
  <p> chmod 700 .ssh
  <p> chmod 600 .ssh/authorized_keys
  <p> На backup_client сгенерировал ssh ключ
  <p> ssh-keygen
  <p> Скопировал ssh ключ на сервер backup_server
</ul>  

# 3. Бэкап
<ul>
<p> Инициализировал репозиторий borg на backup_server с backup_client:
<p> borg init --encryption=repokey borg@192.168.56.120:/var/backup/
<p> Проверка создания бэкапа
<p> borg create --stats --list borg@192.168.56.120:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc
<p> Проверил результат
<p> borg list borg@192.168.56.120:/var/backup/
<p> Enter passphrase for key ssh://borg@192.168.56.120/var/backup:
<p> etc-2021-10-15_23:00:15 Fri, 2021-10-15 23:00:21
<p> [573f7b4071bd2e079957217f397394c336eaf172208755110b311ada735e16d3]
<p> Список файлов
<p> borg list borg@192.168.56.120:/var/backup/::etc-2022-08-03_22:28:53
<p> Достал файл из бекапа
<p> borg extract borg@192.168.56.120:/var/backup/::etc-2022-08-03_22:28:53 etc/hostname
</ul>

# 4. Настройка автоматизации
<ul>
<p> Автоматизируем создание бэкапов
<p> /etc/systemd/system/borg-backup.service
<p> [Unit]
<p> Description=Borg Backup
<p> [Service]
<p> Type=oneshot
<p> Environment="BORG_PASSPHRASE=Otus1234"
<p> Environment=REPO=borg@192.168.56.120:/var/backup/
<p> Environment=BACKUP_TARGET=/etc
<p> ExecStart=/bin/borg create \ --stats \ ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}
<p> ExecStart=/bin/borg check ${REPO}
<p> ExecStart=/bin/borg prune \
<p> --keep-daily 90 \
<p> --keep-monthly 12 \
<p> --keep-yearly 1 \
<p> ${REPO}
<p> Создаем таймер  
<p> [Unit]
<p> Description=Borg Backup
<p> [Timer]
<p> OnUnitActiveSec=5min
<p> [Install]
<p> WantedBy=timers.target
<p> Включил и запустил службу таймера
<p> systemctl enable borg-backup.timer
<p> systemctl start borg-backup.timer 
</ul>  
