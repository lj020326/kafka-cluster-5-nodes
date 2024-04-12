
# How to install a cluster of five nodes Linux Red Hat 8 with Ansible

Create kafka cluster Red Hat 8 with 5 nodes Ansible Playbook

Apache Kafka is a well known real time streaming solution. This article it is about how to install a cluster of five nodes in Linux Red Hat 8 in a easy way and reproducible with no human intervention.

From the point of view of SRE (Site Reliability Engineering) they tray to enhance up-time greatly. The approach focuses on keeping the platform or service no matter what. Task like disaster prevention, risk mitigation, reliability and redundancy are of the most importance. The SRE teams main goal is find the best ways to prevent problems that can cause downtime. This is crucial especially when you manage large-scale system. Another benefit is that SRE helps brands **eliminate manual work** which gives developers much more time to innovate.

The first is we need our inventory file to define which machines or kind of VM, on premise or in a Cloud provider use. 

# VMs Red Hat 8.4

The example kafka inventory in inventory/hosts.yml:
```yaml
---
all:
  children:
    redhat8_5nodes:
      rh8-nodo1: {}
      rh8-nodo2: {}
      rh8-nodo3: {}
      rh8-nodo4: {}
      rh8-nodo5: {}
```

The second one is download the [Apache Kafka](https://kafka.apache.org/) in my case kafka_2.13–3.1.0.tgz

next step is run the Playbook:

```shell
$ ansible-playbook -i ./inventory/ install-kafka.yml -u root -K
```

install-kafka.yml:
```yaml
---
- name: Apache Kafka kafka_2.13-3.1.0 playbook install Red Hat 8.4
  hosts: redhat8_5nodes
  remote_user: root
  vars:
    package_name: kafka
    package_version: kafka_2.13-3.1.0
  tasks:

  - name: Create kafka user and data folders
    shell: |
      useradd kafka
      mkdir -p /data/kafka
      mkdir -p /data/zookeeper
      echo {{inventory_hostname}} | tail -c 2 > /data/zookeeper/myid
      chown -R kafka:kafka /data/kafka*
      chown -R kafka:kafka /data/zookeeper*

  - name: Copy binary to /opt
    ansible.builtin.copy:
      src: kafka_2.13-3.1.0.tgz
      dest: /opt
      owner: kafka
      group: kafka

  - name: Extract binary
    ansible.builtin.unarchive:
      src: /opt/kafka_2.13-3.1.0.tgz
      dest: /opt
      owner: kafka
      group: kafka
      remote_src: yes

  - name: Create symbolic link to folder kafka
    shell: |
      ln -s /opt/kafka_2.13-3.1.0 /opt/kafka
      chown -R kafka:kafka /opt/kafka*

  - name: Createt Zookeeper system service
    ansible.builtin.template:
      src: zookeeper.service
      dest: /etc/systemd/system/zookeeper.service
      owner: root
      group: root

  - name: Createt Kafka system service
    ansible.builtin.template:
      src: kafka.service
      dest: /etc/systemd/system/kafka.service
      owner: root
      group: root

  - name: Select Server config file by node name
    ansible.builtin.template:
      src: server-{{inventory_hostname}}.properties
      dest: /opt/kafka/config/server.properties
      owner: kafka
      group: kafka

  - name: Select Zookeeper config file by node name
    ansible.builtin.template:
      src: zookeeper.properties
      dest: /opt/kafka/config/zookeeper.properties
      owner: kafka
      group: kafka

  - name: start services Zookeeper and Kafka
    shell: |
      systemctl daemon-reload
      systemctl start zookeeper
      systemctl start kafka
      systemctl enable zookeeper.service
      systemctl enable kafka.service
```

Really simple a nice way to implement it Kafka cluster.
