# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.provision "shell", inline: <<-SHELL
    # Установка необходимых пакетов
    yum -y install epel-release
    yum -y install spawn-fcgi httpd

    # Настройка файла /etc/sysconfig/myapp
    echo 'LOG_FILE=/var/log/myapp.log' > /etc/sysconfig/myapp
    echo 'KEYWORD=mykeyword' >> /etc/sysconfig/myapp

    # Настройка файла /etc/systemd/system/spawn-fcgi.service
    cat << EOF > /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn FastCGI service

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
ExecStart=/usr/bin/spawn-fcgi -n -p 9000 -u apache -g apache -f /usr/bin/php-cgi
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
EOF

    # Настройка файла /etc/systemd/system/multi-httpd@.service
    cat << EOF > /etc/systemd/system/multi-httpd@.service
[Unit]
Description=Apache Web Server (%i)
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/sbin/httpd -f /etc/httpd/conf/%i/httpd.conf -DFOREGROUND
ExecReload=/usr/sbin/httpd -k graceful
ExecStop=/usr/sbin/httpd -k stop
PIDFile=/var/run/httpd-%i.pid
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

    # Настройка файла /etc/httpd/conf/myapp/httpd.conf
    mkdir -p /etc/httpd/conf/myapp
    cat << EOF > /etc/httpd/conf/myapp/httpd.conf
ServerName localhost

Listen 8080

<VirtualHost *:8080>
  DocumentRoot /var/www/html/myapp
  <Directory /var/www/html/myapp>
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
EOF

    # Создание директории и файла для логов
    mkdir -p /var/log/myapp
    touch /var/log/myapp.log

    # Настройка файла /etc/cron.d/myapp
    echo '*/30 * * * * root grep -q $KEYWORD $LOG_FILE || logger "Keyword not found in $LOG_FILE"' > /etc/cron.d/myapp

    # Запуск сервисов
    systemctl daemon-reload
    systemctl enable spawn-fcgi
    systemctl enable multi-httpd@myapp
    systemctl start spawn-fcgi
    systemctl start multi-httpd@myapp
  SHELL
end
