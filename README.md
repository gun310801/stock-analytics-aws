# Analytics on Stock Market Data Live Stream using AWS

## Data :

 Live Market Data:
 1. During Market Hours
 - Quote Data for AAPL
 - Bar Data for AAPL, MSFT, GOOG, SPY, META, DIA, TSLA
 2. Outside Market Hours 
 - Quote Data for FAKEPACA
 - Bar Data for FAKEPACA 

Websocket : 
- wss://stream.data.alpaca.markets/v2/iex
- wss://stream.data.alpaca.markets/v2/test

## AWS

1. Create EMR Cluster
- Configure Cluster
- Connect to Primary node using ssh


2. Edit EC2 Security Groups - give access to local machine's ip


3. SSH EMR Cluster on local machine
- Navigate to the aws-key.pem directory
- ssh into the emr cluster


4. Set up kafka
- pip install kafka-python
- pip install boto3
- pip install websocket-client streamlit watchdog plotly
-  wget https://downloads.apache.org/kafka/3.5.2/kafka_2.13-3.5.2.tgz
- tar -xzf kafka_2.13-3.5.2.tgz
- cd kafka_2.13-3.5.2
- nano config/server.properties
- look for advertized listeners and change it to ec2's host ip for master node
- look for zookeeper connect and change it to ec2's host ip for master node
- bin/kafka-server-start.sh config/server.properties


5. Set up Producer.py
- vim producer.py
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic symbol_topic
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic symbol_topic2
- mkdir -p /home/hadoop/producer1
- nano /home/hadoop/producer1/log4j.properties
- spark-submit --conf “spark.driver.extraJavaOptions=-Dlog4j.configuration=producer1/log4j.properties" \
             --conf “spark.executor.extraJavaOptions=-Dlog4j.configuration=producer1/log4j.properties" \
             producer.py



6. Set up Consumer.py
- vim consumer.py
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic visual_topic
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic visual_topic2
- mkdir -p /home/hadoop/consumer1
- nano /home/hadoop/consumer1/log4j.properties
- spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.2.1 \
             --conf "spark.driver.extraJavaOptions=-Dlog4j.configuration=file:/home/hadoop/consumer1/log4j.properties" \
             --conf "spark.executor.extraJavaOptions=-Dlog4j.configuration=file:/home/hadoop/consumer1/log4j.properties" \
             consumer1.py


7. Set up Visualizer (Viz.py)
- ~/.local/bin/streamlit run viz.py --server.port 8501 --server.address 0.0.0.0
- ec2-> Security Groups -> Edit Inbound Rules -> add 8501, 0.0.0.0/0 ->Save rules


8. Set up Log Visualizer (log_viz.py)
- ~/.local/bin/streamlit run log_viz.py --server.port 8502 --server.address 0.0.0.0
- ec2-> Security Groups -> Edit Inbound Rules -> add 8502, 0.0.0.0/0 ->Save rules


9. Ganglia (data monitoring)
    - look for ganglia application under emr
    - ssh tunneling to port 8050
    - create a SOCKS5 proxy on the browser
    - look for master/dns url
    - ssh -i ./aws-master-node.pem -ND 8050 hadoop@ec2-44-201-28-163.compute-1.amazonaws.com
    - proxy onto SOCKS5
