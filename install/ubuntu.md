## Custom Installation Guide (pfSense/OPNsense) + Elastic Stack 

## Table of Contents

- [Preparation](#preparation)
- [Installation](#installation)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

# Preparation

### 1. Add MaxMind Repository
```
sudo add-apt-repository ppa:maxmind/ppa
```

### 2. Download and install the public GPG signing key
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

### 3. Download and install apt-transport-https package
```
sudo apt install apt-transport-https
```

### 4. Add Elasticsearch|Logstash|Kibana Repositories (version 7+)
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

### 5. Update
```
sudo apt update
```

### 6. Install Java 14 LTS
```
sudo apt install openjre-14-jre-headless
```

### 7. Install MaxMind
```
sudo apt install geoipupdate
```

### 8. Configure MaxMind
- Create a MaxMind Account @ https://www.maxmind.com/en/geolite2/signup
- Login to your MaxMind Account; navigate to "My License Key" under "Services" and Generate new license key
```
sudo nano /etc/GeoIP.conf
```
- Modify lines 7 & 8 as follows (without < >):
```
AccountID <Input Your Account ID>
LicenseKey <Input Your LicenseKey>
```
- Modify line 13 as follows:
```
EditionIDs GeoLite2-City GeoLite2-Country GeoLite2-ASN
```
- Modify line 18 as follows:
```
DatabaseDirectory /usr/share/GeoIP/
```

### 9. Download Maxmind Databases
```
sudo geoipupdate
```

### 10. Add cron (automatically updates Maxmind everyweek on Sunday at 1700hrs)
```
sudo nano /etc/cron.weekly/geoipupdate
```
- Add the following and save/exit
```
00 17 * * 0 geoipupdate
```

# Installation
- Elasticsearch v7+ | Kibana v7+ | Logstash v7+

### 11. Install Elasticsearch|Kibana|Logstash
```
sudo apt install elasticsearch; sudo apt install kibana; sudo apt install logstash
```

# Configuration

### 12. Configure Kibana
```
sudo nano /etc/kibana/kibana.yml
```

### 13. Modify host file (/etc/kibana/kibana.yml)
- server.port: 5601
- server.host: "0.0.0.0"

### 14. Change Directory
```
cd /etc/logstash/conf.d
```

### 15. (Required) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/01-inputs.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/02-types.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/03-filter.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/05-firewall.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/30-geoip.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/50-outputs.conf
```

### 15a. (Optional) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/10-others.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/15-squid.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/20-suricata.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/25-snort.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/35-rules-desc.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/36-ports-desc.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/45-cleanup.conf
```

### 16a. Make Patterns Folder
```
sudo mkdir /etc/logstash/conf.d/patterns
```

### 16b. Navigate to Patterns Folder
```
cd /etc/logstash/conf.d/templates
```

### 16c. Download the grok pattern
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/patterns/pfelk.grok
```

### 17a. Make Templates Folder
```
sudo mkdir /etc/logstash/conf.d/templates
```

### 17b. Navigate to Templates Folder
```
cd /etc/logstash/conf.d/templates
```

### 17c. Download Template(s)
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/templates/pf-geoip.json
```

### 18a. Make Patterns Folder
```
sudo mkdir /etc/logstash/conf.d/patterns
```

### 18a. Navigate to Patterns Folder
```
sudo mkdir /etc/logstash/conf.d/patterns
```

### 18b. Download the Database(s)
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/databases/rule-names.csv
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/databases/service-names-port-numbers.csv
```

### 19. Enter your pfSense/OPNsense IP address (/data/pfELK/configurations/02-types.conf)
```
Change line 5; the "if [host] == ..." should point to your pfSense/OPNsense IP address
Change line 8; rename "firewall" (OPTIONAL) to identify your device (i.e. backup_firewall)
Change line 11-20; (OPTIONAL) to point to your second PF IP address or ignore
```

### 20. Revise/Update w/pf IP address (/data/pfELK/configurations/03-filter.conf)
```
For OPNsense uncommit line 4 and commit out line 5 (#opn#)
For pfSense uncommit line 5 and commit out line 4 (#pf#)
```

# Troubleshooting
### 21. Create Logging Directory 
```
sudo mkdir -p /etc/pfELK/logs
```

### 22. Navigate to pfELK  
```
cd /etc/pfELK/
```

### 23. Download `error-data.sh`
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/error-data.sh
```

### 24. Make `error-data.sh` Executable
```
sudo chmod +x /etc/pfELK/error-data.sh
```

### 25. Complete Configuration --> [Configuration](configuration.md)
