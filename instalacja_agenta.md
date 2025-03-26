## Instalacja agenta Zabbix na monitorowanym systemie

1. Dodaj repozytorium Zabbix

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/7.2/release/rocky/9/noarch/zabbix-release-latest-7.2.el9.noarch.rpm
sudo dnf clean all
```

2. Zainstaluj Zabbix Agent

```bash
sudo dnf install zabbix-agent -y
```

3. Skonfiguruj Zabbix Agent:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```


Znajdź linię Server= i ustaw adres IP lub nazwę hosta serwera Zabbix:

```bash
Server=<IP-serwera-Zabbix>
```

Znajdź linię ServerActive= i ustaw adres IP lub nazwę hosta serwera Zabbix (opcjonalne, dla aktywnego monitorowania):

```bash
ServerActive=<IP-serwera-Zabbix>
```

Znajdź linię Hostname= i ustaw nazwę hosta (nazwa musi być unikalna i pasować do nazwy hosta dodawanego w Zabbix Web Interface):

```bash
Hostname=<nazwa-hostname>
```

4. Uruchamiamy usługi agenta

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

5. Konfigurujemy firewalla, aby otworzyć port 10050

```bash
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --reload
```
