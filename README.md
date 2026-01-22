# Serveis IoT

Configuracions de docker-compose i les seves instruccions sobre com instal·lar-ho

## Xarxa Iot

```bash
docker network create xarxa_IOT
```



## Mqtt:

### Crear carpetes
```bash
mkdir -p /opt/docker/mqtt/data
mkdir -p /opt/docker/mqtt/config
chown 1883:1883 /opt/docker/mqtt/config /opt/docker/mqtt/data
```


### configuracio Mqtt

```properties
password_file /mosquitto/config/pwd_usuaris
# MQTT Default listener
listener 1883 0.0.0.0

# MQTT over WebSockets with TLS
listener 9001 0.0.0.0
protocol websockets
```

Executar mosquitto_password -c fitxer usuari per crear un arxiu de passwords amb un usuari. Sense opció, per afegir usuaris a un arxiu creat

```bash
docker exec -it mqtt /usr/bin/mosquitto_password -c /mosquitto/config/pwd_usuaris IOT
```


## Influx2
```bash
mkdir -p /opt/docker/influx/data
chown 1000:1000 /opt/docker/influx/data
```

## Grafana
```bash
mkdir -p /opt/docker/grafana/data
chown 472:472 /opt/docker/grafana/data
```

## Telegraf
```bash
mkdir -p /opt/docker/telegraf/conf/telefraf.d
docker run --rm telegraf telegraf config > /opt/docker/telegraf/conf/telegraf.conf
```
Configuracio extra per a enviar dades a influx des de telegraf.:
```properties
[outputs.influxdb_v2]
   urls = ["http://influxdb:8086"]
   ## Token for authentication.
   token = "3Fb****=="
   ## Organization is the name of the organization you wish to write to; must exist.
   organization = "Itic"
   ## Destination bucket to write into.
   bucket = "telegraf"
```

## Influx db3
Directoris:

```bash
sudo mkdir -p /opt/docker/influxdb3/data
sudo mkdir -p /opt/docker/influxdb3/plugins

python3 -m venv /opt/docker/influxdb3/plugins/.venv
```

# Directoris per a la UI (en la ruta actual del docker-compose.yml)

```bash
mkdir -p /opt/docker/influxdb3/ui/db
mkdir -p /opt/docker/influxdb3/ui/config
mkdir -p /opt/docker/influxdb3/ui/ssl


sudo chown -R 1000:1000 /opt/docker/influxdb3/
sudo chown -R 1500:1500 /opt/docker/influxdb3/data
sudo chmod -R 755 /opt/docker/influxdb3/
```

## Influx db3-UI
### Generate session secret
```bash
echo "SESSION_SECRET_KEY=$(openssl rand -base64 32)" > /opt/docker/influxdb3/.env
```

### Api tokcen per a influxdb

Un cop s'hagi aixecat influxdb3 caldrà generar un token:

docker exec -it influxdb3 influxdb3 create token --admin

i posar-lo a la configuracio de influxdb3ui

Crear bbdd IoT:

```bash
docker exec -it influxdb3 influxdb3 create database IoT --token apiv3_....
```


** Nota**
La versió core (no enterpise) només suporta admin tokens.


### Create configuration
```bash
cat > /opt/docker/influxdb3/ui/config/config.json << 'EOF'
{
  "DEFAULT_INFLUX_SERVER": "http://influxdb3:8181",
  "DEFAULT_INFLUX_DATABASE": "",
  "DEFAULT_API_TOKEN": "apiv3_********",
  "DEFAULT_SERVER_NAME": ""
}
EOF
```


## Node-red
Referència:
- [https://nodered.org/docs/getting-started/docker#:~:text=To%20allow%20access%20to%20this,owner%20of%20the%20host%20directory.](https://nodered.org/docs/getting-started/docker#:~:text=To%20allow%20access%20to%20this,owner%20of%20the%20host%20directory.)

### Carpetes

```
sudo mkdir -p /opt/docker/node-red/data/
chown 1000:1000 /opt/docker/node-red/data/
chmod 750 /opt/docker/node-red/data/
```


### Settings:
Per afegir usuaris i passwords:
editar l'arxiu data/settings.js

```
   /** To password protect the Node-RED editor and admin API, the following
     * property can be used. See https://nodered.org/docs/security.html for details.
     */
    adminAuth: {
        type: "credentials",
        users: [{
            username: "user1",
            password: "$2b.....",
            permissions: "*"
        },
        {
            username: "user2",
            password: "$2b.....",
            permissions: "read"
        }]
    },
```

### Docker-compose:

[Docker-Compose de nodered](./docker-compose_nodered.yaml)
