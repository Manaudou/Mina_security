# Mina_security
Общие рекомендации
1. Если вы вошли в систему как пользователь «root», вместо этого создайте пользователя с правами администратора:
adduser <yourusername>
Предоставьте ему права администратора:
usermod -aG sudo <yourusername>  
  Выполните эту команду, если вы использовали ключи SSH для подключения к VSP.
  rsync --archive --chown=<yourusername>:<yourusername> ~/.ssh /home/<yourusername>

2. Отключите порт SSH по умолчанию и измените его на случайный:
Выберите случайный номер порта от 1024 до 49151
Найдите строку Port 22 в файле конфигурации SSH:
sudo nano /etc/ssh/sshd_config
Удалите # (если есть) и измените 22 на выбранное вами число.
Сохраните файл (Ctrl + X, затем Y).
Перезапустите службу SSH:
sudo systemctl restart ssh
Выход из VPS:
exit
Авторизуйтесь через новый порт:
ssh <yourusername>@<ipaddressofvps> -p <yourSSHportnumber>
Установите брандмауэр UFW:
sudo apt install ufw
Разрешите входящий трафик на ваш порт:
sudo ufw allow <yourSSHportnumber>/tcp
Запретить трафик на порт 22:
sudo ufw deny 22/tcp
Разрешить входящий трафик на порт 8302 TuCP (для узла Mina):
sudo ufw allow 8302/tcp
При желании, чтобы использовать службу GraphQL, откройте TCP-порт 3085:
sudo ufw allow 3085/tcp
  
  При использовании хостинг-провайдеров как Hetzner надо настроить брандмауэр, чтобы разрешить трафик с ssh, http, https и запретить исходящий трафик на все частные IP-адреса.
  sudo ufw allow 80/tcp
  sudo ufw allow 443/tcp
Включите брандмауэр:  
sudo ufw enable

  и Заблокировать исходящее частное соединение:
  sudo ufw deny out from any to 10.0.0.0/8
  sudo ufw deny out from any to 172.16.0.0/12
  sudo ufw deny out from any to 192.168.0.0/16
  sudo ufw deny out from any to 100.64.0.0/10
  sudo ufw deny out from any to 198.18.0.0/15
  sudo ufw deny out from any to 169.254.0.0/16
  
3. Отключить вход по SSH для пользователя root
Войдите, используя имя пользователя, созданное на 1. (пользователь с правами администратора)
Откройте файл конфигурации SSH:
sudo nano /etc/ssh/sshd_config
Замените строку #PermitRootLogin yes на PermitRootLogin no (удалите #)
Перезапустите службу SSH:
sudo systemctl restart ssh
Выход из VPS:
logout
Теперь используйте <yourusername> для подключения к вашему серверу и sudo перед командой, требующей прав администратора.
  
4. Используйте Fail2Ban - это программное обеспечение сканирует системные журналы и блокирует IP-адреса, которые не могут войти в систему слишком много раз.
Более подробную инструкция по установке можно взять по сылке: https://help.skysilk.com/support/solutions/articles/9000149908--basic-how-to-install-and-configure-fail2ban-for-linux-vps#Downloading-and-Installing-Fail2Ban
Ubuntu/Debian
sudo apt-get install fail2ban
Настройка параметров Fail2Ban - Fail2Ban будет работать с настройками по умолчанию, но есть определенные настройки, которые может быть интересно изменить
Вместо того, чтобы редактировать файл /etc/fail2ban/jail.conf напрямую, мы сделаем копию /etc/fail2ban/jail.local
sudo cp -pf /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
(Примечание. Параметры файла .local имеют приоритет над параметрами .conf.)
Чтобы открыть файл с помощью nano, введите команду
sudo nano /etc/fail2ban/jail.local
измените кофигурации под себя
Ignoreip - добавить свой IP для избежания бана самому себе при наепрвильно наборе пароля
Bantime - время в секундах, на которое IP-адрес будет заблокирован.
Findtime - время, в течение которого количество неудачных подключений приведет к блокировке
Maxretry - число повторных попыток, разрешенных в течение определенного времени FIndtime для определения того, является ли адрес заблокированным.
обязательно перезапустите службу Fail2Ban, чтобы изменения конфигурации вступили в силу
sudo service fail2ban restart
1.  Чтобы узнать, какие тюрьмы запущены
sudo fail2ban-client status
2.  Проверить статус ssh jail и получить список заблокированных IP-адресов
sudo fail2ban-client -v status ssh
3.  Вы также можете проверить iptables на наличие списка заблокированных IP-адресов.
sudo iptables -L -n
4.  Чтобы удалить забаненный IP из ssh jail
sudo fail2ban-client set ssh unbanip IPADDRESS
5.  Чтобы вручную заблокировать IP
sudo fail2ban-client set ssh banip IPADDRESS
Просмотр файлов журнала
sudo cat /var/log/auth.log | grep 'Failed password'
Более подробную информацию о командах см. На страницах руководства Fail2Ban
sudo man fail2ban-client
sudo man fail2ban-server

