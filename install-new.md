# نصب Zabbix 7.0 روی Debian 12 با PostgreSQL و Apache2

این راهنما مراحل کامل نصب و پیکربندی Zabbix 7.0 روی Debian 12 را با استفاده از پایگاه داده PostgreSQL و وب‌سرور Apache2 توضیح می‌دهد.

---

## مراحل نصب

```bash
# به‌روزرسانی سیستم و نصب پکیج‌های پایه
sudo apt update && sudo apt upgrade -y
sudo apt install -y nano vim wget curl net-tools network-manager resolvconf
sudo apt install -y wget curl gnupg2 lsb-release software-properties-common
sudo apt install -y build-essential libjpeg-dev libpng-dev libtiff-dev

# افزودن مخزن رسمی Zabbix 7.0
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
sudo apt install ./zabbix-release_latest_7.0+debian12_all.deb
sudo apt update && sudo apt upgrade -y

# نصب Zabbix server، frontend، agent و کتابخانه‌های مورد نیاز
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.2-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# نصب PostgreSQL و راه‌اندازی آن
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql

# ساخت یوزر و دیتابیس در PostgreSQL
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix

# ویرایش فایل pg_hba.conf برای مجاز کردن دسترسی
sudo nano /etc/postgresql/15/main/pg_hba.conf


local   all             all                                     trust
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256


# ایمپورت اسکیمای اولیه Zabbix در دیتابیس
# ریستارت PostgreSQL
sudo systemctl restart postgresql

# ایمپورت اسکیمای اولیه Zabbix در دیتابیس
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | psql -U zabbix -d zabbix -W

# پیکربندی Zabbix Server
sudo nano /etc/zabbix/zabbix_server.conf

DBName=zabbix
DBUser=zabbix
DBPassword=MyZabbixDBpass

# پیکربندی Zabbix Agent
sudo nano /etc/zabbix/zabbix_agentd.conf
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=postgres

# راه‌اندازی و فعال‌سازی سرویس‌ها
sudo systemctl restart zabbix-server zabbix-agent apache2 postgresql
sudo systemctl enable zabbix-server zabbix-agent apache2 postgresql
