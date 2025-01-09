# Graylog

# 🛠️ Installation pas à pas de Graylog

Commençons par une mise à jour du cache des paquets et l'installation d'outils nécessaires pour la suite des événements :

```bash
sudo apt-get update
sudo apt-get install curl lsb-release ca-certificates gnupg2 pwgen
```

## 🗂️ A. Installation de MongoDB <br> 
:rotating_light::exclamation:⚠️ ATTENTION RISQUE DE CONFLIT : CPU et certaines versions de **mongodb** :exclamation: Vérifier avant installation  ⚠️:exclamation:

### 📥 Téléchargez la clé GPG :
```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor
```

### ➕ Ajoutez le dépôt MongoDB 6 :
```bash
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg] http://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

### 🔄 Mettez à jour le cache et installez MongoDB :

⚠️ **Dépendance manquante `libssl1.1`**

Installez `libssl1.1` manuellement :
```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb
```
Et une fois la dépendance manquante installée poursuivre avec :
```bash
sudo apt-get update
sudo apt-get install -y mongodb-org
```

Relancez l'installation de MongoDB :
```bash
sudo apt-get install -y mongodb-org
```

### 🔧 Configurez et démarrez MongoDB :
```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl restart mongod.service
sudo systemctl --type=service --state=active | grep mongod
```

## 🗂️ B. Installation d'OpenSearch

### 📥 Ajoutez la clé GPG :
```bash
curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo gpg --dearmor --batch --yes -o /usr/share/keyrings/opensearch-keyring
```

### ➕ Ajoutez le dépôt OpenSearch :
```bash
echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch-2.x.list
```

### 🔄 Mettez à jour et installez OpenSearch :
```bash
sudo apt-get update
sudo env OPENSEARCH_INITIAL_ADMIN_PASSWORD=Azertygraylog1 apt-get install opensearch
```

### ⚙️ Configurez OpenSearch :
Modifiez le fichier de configuration :
```bash
sudo nano /etc/opensearch/opensearch.yml
```

Ajoutez ou modifiez les paramètres suivants :
```yaml
cluster.name: graylog
node.name: ${HOSTNAME}
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch
discovery.type: single-node
network.host: 127.0.0.1
action.auto_create_index: false
plugins.security.disabled: true
```

### ⚙️ Configurez la JVM :
Modifiez la mémoire allouée :
```bash
sudo nano /etc/opensearch/jvm.options
```

Remplacez :
```text
-Xms1g
-Xmx1g
```
par :
```text
-Xms4g
-Xmx4g
```

Vérifiez ou configurez `max_map_count` :
```bash
cat /proc/sys/vm/max_map_count
sudo sysctl -w vm.max_map_count=262144
```

Activez et démarrez OpenSearch :
```bash
sudo systemctl daemon-reload
sudo systemctl enable opensearch
sudo systemctl restart opensearch
```

## 🗂️ C. Installation de Graylog

### 📥 Téléchargez et installez Graylog :
```bash
wget https://packages.graylog2.org/repo/packages/graylog-6.1-repository_latest.deb
sudo dpkg -i graylog-6.1-repository_latest.deb
sudo apt-get update
sudo apt-get install graylog-server
```

### 🔧 Configurez Graylog :
Générez une clé `password_secret` :
```bash
pwgen -N 1 -s 96
```
Modifiez le fichier de configuration :
```bash
sudo nano /etc/graylog/server/server.conf
```

Ajoutez :
```text
password_secret=VOTRE_CLÉ_GÉNÉRÉE
root_password_sha2=HASH_DU_MOT_DE_PASSE
http_bind_address=0.0.0.0:9000
elasticsearch_hosts=http://127.0.0.1:9200
```

### ⚙️ Démarrez Graylog :
```bash
sudo systemctl enable --now graylog-server
```

Accédez à l'interface web via :
```
http://<IP_DU_SERVEUR>:9000
```

## 🏁 Conclusion
Félicitations, vous avez installé Graylog ! Vous pouvez maintenant gérer vos logs de manière centralisée. 🎉
