# Kafka Installation In Ubuntu

1. Install the Confluent public key. This key is used to sign the packages in the APT repository.

```bash
wget -qO - https://packages.confluent.io/deb/7.4/archive.key | sudo apt-key add -
```

1. Add the repository to your `/etc/apt/sources.list` by running this command:

```bash
sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/7.4 stable main"
sudo add-apt-repository "deb https://packages.confluent.io/clients/deb $(lsb_release -cs) main"
```

1. Update `apt-get` and install the entire Confluent Platform platform.
    - Confluent Platform with [RBAC](https://docs.confluent.io/platform/current/security/rbac/index.html#rbac-overview):
    
    ```bash
    sudo apt-get update && \
    sudo apt-get install confluent-platform && \
    sudo apt-get install confluent-security
    ```
    

# Kafka server start & status command

1. For ZooKeeper mode, start ZooKeeper. For KRaft mode, skip to step 2.

```bash
sudo systemctl start confluent-zookeeper
```

1. Start Kafka
    - Confluent Platform
    
    ```bash
    sudo systemctl start confluent-server
    ```
    
    - Confluent Platform using only Confluent Community components:
    
    ```bash
    sudo systemctl start confluent-kafka
    ```
    
2. Start Schema Registry

```bash
sudo systemctl start confluent-schema-registry
```

1. Start other Confluent Platform components as desired.
    - Control Center
    
    ```bash
    sudo systemctl start confluent-control-center
    ```
    
    - Kafka Connect
    
    ```bash
    sudo systemctl start confluent-kafka-connect
    ```
    
    - Confluent REST Proxy
    
    ```bash
    sudo systemctl start confluent-kafka-rest
    ```
    
    - ksqlDB
    
    ```bash
    sudo systemctl start confluent-ksqldb
    ```
    

---

************Note:************ You can check service status with this command: `systemctl status confluent*`. For more information about the systemd service unit files, see [Use Confluent Platform systemd Service Unit Files](https://docs.confluent.io/platform/current/installation/installing_cp/scripted-install.html#installing-systemd-unit).

# Uninstall

- Run this command to remove Confluent Platform, where `<component-name>` is either `confluent-platform` (Confluent Platform) or `confluent-community-2.13` (Confluent Platform using only Confluent Community components).

```bash
sudo apt-get remove <component-name>
```

- For example, run this command to remove Confluent Platform:

```bash
sudo apt-get remove confluent-platform
```