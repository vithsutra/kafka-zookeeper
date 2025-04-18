# kafka-zookeeper
✅ STEP 1: Install Java
bash
Copy
Edit
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
✅ STEP 2: Create a Kafka User
bash
Copy
Edit
sudo useradd kafka -m
sudo passwd kafka
Give it a password when prompted.

✅ STEP 3: Download Kafka
Switch to the Kafka user:

bash
Copy
Edit
su - kafka
Download and extract Kafka:

bash
Copy
Edit
KAFKA_VERSION="3.6.0"
SCALA_VERSION="2.13"
wget https://downloads.apache.org/kafka/${KAFKA_VERSION}/kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz
tar -xzf kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz
mv kafka_${SCALA_VERSION}-${KAFKA_VERSION} kafka
Add Kafka bin to PATH (optional):

bash
Copy
Edit
echo 'export PATH=$PATH:/home/kafka/kafka/bin' >> ~/.bashrc
source ~/.bashrc
✅ STEP 4: Configure Zookeeper & Kafka
🐘 Create data directories:
bash
Copy
Edit
mkdir -p /home/kafka/kafka/data/zookeeper
mkdir -p /home/kafka/kafka/data/kafka
🐘 Edit Zookeeper config:
bash
Copy
Edit
nano /home/kafka/kafka/config/zookeeper.properties
Update:

ini
Copy
Edit
dataDir=/home/kafka/kafka/data/zookeeper
clientPort=2181
🐦 Edit Kafka config:
bash
Copy
Edit
nano /home/kafka/kafka/config/server.properties
Update:

ini
Copy
Edit
log.dirs=/home/kafka/kafka/data/kafka
zookeeper.connect=localhost:2181
✅ STEP 5: Create systemd Services
Exit Kafka user:

bash
Copy
Edit
exit
🐘 Create Zookeeper systemd unit:
bash
Copy
Edit
sudo nano /etc/systemd/system/zookeeper.service
Paste:

ini
Copy
Edit
[Unit]
Description=Apache Zookeeper
After=network.target

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
🐦 Create Kafka systemd unit:
bash
Copy
Edit
sudo nano /etc/systemd/system/kafka.service
Paste:

ini
Copy
Edit
[Unit]
Description=Apache Kafka
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
✅ STEP 6: Enable & Start Services
bash
Copy
Edit
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

sudo systemctl enable zookeeper
sudo systemctl start zookeeper

sudo systemctl enable kafka
sudo systemctl start kafka
✅ STEP 7: Check Service Status
bash
Copy
Edit
sudo systemctl status zookeeper
sudo systemctl status kafka
✅ STEP 8: Test Kafka
Switch to Kafka user:

bash
Copy
Edit
su - kafka
Create a topic:

bash
Copy
Edit
kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
List topics:

bash
Copy
Edit
kafka-topics.sh --list --bootstrap-server localhost:9092
Produce a message:

bash
Copy
Edit
kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
(Write a few messages, then Ctrl+C)

Consume:

bash
Copy
Edit
kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --from-beginning
