# APACHE-KAFKA
![kafka3](https://github.com/user-attachments/assets/38cb1b6e-6c5b-4cf8-a2d9-752340112c62)

![kafka2](https://github.com/user-attachments/assets/19762c43-55cf-407e-a8c2-9e50c2feb40e)

![Architecture (1)](https://github.com/user-attachments/assets/7fca43cb-f11a-44a7-8171-cfca6bf2c7c4)



Apache Kafka
It is used to deliver live data and is used for building data pipelines and streaming applications.
Eg: Amzon needs to know what each customer is doingnon there website or what are they viewing this all information is sent real time using Apache kafka.
So apache kafka is highly used oin moving data between different systems in real time 

Apache Kafka consists of:
1) Producer: Producers are data generator which consitunosly send data to pipeline 
2) Broker: They store the data sent by producer 
3) Topic: Topic are logical representation inside kafka broker.
4) Partition: Each topic has multiple partition.
5) Consumer: Consumer collect data from broker for further use.
6) Event: It is message or smallest unit of data sent by producer to kafka topics and consumed by consumer

                                        /---PARTITION1   
                    BROKER1  -- TOPIC1 /----PARTITION2   \
                   /                                     -   CONSUMER1
		  /	     -- TOPIC2/----PARTITION1    -   CONSUMER2
  | Producer1 |  /                   /-----PARTITION2    /
                /
  | Producer2 | \
                 \
                  \ BROKER2  -- TOPIC1 /---PARTITION1    
				       /---PARTITION1  \
                                                       -  CONSUMER1
                             -- TOPIC2 /---PARTITION1  -  CONSUMER2
                                       /---PARTITION1  /




Eg: User using e-commerce website, here application(producer) is producing data about users there clicks there searches, this data is sent through data pipeline which is stored in servers(broker) each category is consider as topic(type of data generated eg: user clicks, user id) each topic in broker has partitions(data is divied based on user id) and then it is collected by analytical system(consumer) for further analysis for making dashboards and reports.







Apache kafka also uses zookeeper to manage the task. There are so many brokers handelling data so apache kafka needs tool to manage and coordinate this all brokers or any failure so it relies on apache kafka, but eventually newer version of apache kafka are moving on to KRaft based architecture. Kraft nadles metadate internally within kafka, eliminating need for external system.
KRaft does same thing of zookeeper but internally.






SO NOW EC2 MACHINE HAS  BOTH PRIVATE AND PUBLIC IP ADDRESS LISTED , SO WE CAN'T CONNECT OUR LOCAL DEVICE TO EC2 MACHINE THROUGH IT'S PRIVATE ID ADDRESS UNLESS UNTIL WE ARE IN SAME NETWROK SO FOR THIS WE WILL CONNECT OUR LOCAL DEVOCE TO EC2 USING IT'S PUBLIC IP ADDRESS
NOW USING AWS EC2 TO INSTALL KAFKA AND ALL OTHER DEPENDENCIEES

1) CMD1
SO CONNECTING OUR LOCAL MACJHINE TO AWS EC2 INSTAMCE USING SSH KEY
THEN DOWNLOADING REQUIRED DEPENDENCIES EG: APACHE KAFKA, JAVA
AFTER THIS WE WILL START ZOOKEEPER 

2) CMD2
NOW OPEN ONE MORE CMD SO AGAIN CONNECTING TO EC2 MACHINE USING SSH KEY
SO HERE WE WILL START KAFKA SERVER

3) CMD3
SO NOW OPENING NEW WINDOW FOR CMD AGAIN CONNECTING TO AWS EC2 INSTANCE USING EC2 SSH KEY
WE WILL USE THIS TO CREATE TOPIC AND PRODUCER 

4) CMD4
SO NOW SAME WAY CONNECTING TO EC2 USING SSH KEY
HERE CRETING CONSUMER

SO NOW YOU CAN SEE WHAT EVER WE TYPE(GVE INPUT) ON CMD3 IN PORODUCER WE WILL GET IT ON CONSUMER END.

SO IN ALL PIPLEINE IS CTETED SO WHAT EVER WE TYPE ON PRODUCER END WE ARE IMMEDIATLY GETTING IT ON CONSUER END

SO HERE WE ARE GIVING MANUAL INPUT ON PRODUCER END. INSTEAD WE WANT PYTHON PROGRAM TO DO SO OR TO GENERATE DATA AND SEND IT TO CONSUMER.
SO HERE WILL WRITE PYTHON CODE ON PRODUCER END USING KAFKA-PYTHON

PRODUCER.ipynb:
# INSTALL PACKAGES
pip install kafka-python
import pandas as pd
from kafka import KafkaProducer
from time import sleep
from json import dumps
import json

# CONNECT TO EC2 INSTANCE
producer = KafkaProducer(bootstrap_servers=[':9092'], #change ip here
                         value_serializer=lambda x: 
                         dumps(x).encode('utf-8'))

# SEND DATA TO APACHE KAFKA ON EC2 INSTANCE 
producer.send('demo_test', value={'Name':'VaibhavPawar'})


SO NOW YOU WILL IMEDIATELY SEE OUTPUT ON CMD4:
{'Name':'VaibhavPawar'}

**** AGAIN WE DON'T WANT TO SEE OUR CONSUMER OUTPUT ON CMD4 SO WE WILL CRETE CONSUMER.ipynb
# INSTALL PACKAGES
from kafka import KafkaConsumer
from time import sleep
from json import dumps,loads
import json
from s3fs import S3FileSystem

# CONNECT TO EC2 INSTANCE
consumer = KafkaConsumer(
    'demo_test',
     bootstrap_servers=[':9092'], #add your IP here
    value_deserializer=lambda x: loads(x.decode('utf-8')))

# PRINT RECIVED DATA FROM PRODUCER ON CONSUMER END
for c in consumer:
    print(c.value)


SO NOW WE WILL DOWNLOAD DATASET EG: STOCK MARKET DATASET
HERE IN PRODUCER.ipynb END WE WILL PLACE THIS DATASET IN WHILE LOOP

# Read Dataset
df = pd.read_csv("data/indexProcessed.csv")

# Place dataset in loop to randomly select each record send it to kafka server and then it will immediately send it to consumer
while True:
    dict_stock = df.sample(1).to_dict(orient="records")[0]
    producer.send('demo_test', value=dict_stock)
    sleep(1)

SO NOW WILL GET THIS EACH RECORD IN REAL TIME ON CONSUMER.ipynb END FOR EVERY RECORD(IN FORM OF KEY VALUE PAIR, KEY(COLUMN NAME): VALUE(CELL VALUE) FOR THAT RECORD) SENT BY CONSUMER IN ENDLESSS LOOP THAT IS REAL TIME WE WILL GET IT IMMDIATELY ON CONSUMER END.





NOW PUSHING ALL DATA GOT INTO CONSUMER END IN AWS S3:
FOR THIS NOW CREATING S3 BUCKET FOR STORING THIS KEY VALUE PAIR
FOR THIS DONWLOAD AWS CLI SO WE CAN CONFIGURE OR USE S3 THROUGH OUR CMD 

SO NOW CONNECTING TO S3 BUCKET ON CONSUMER END 

SO NOW ON PRDOCUER END WE WILL RUN INFINE LOOP WHICH WILL RANDMLY SELECT RECORD AND SEND IT TO APACHE KAFKA SERVER AND EVENTUALLY TO CONSUER END AND THIS WILL BE COLLECTED ON AT CONSUMER.IPYNB AND SENT TO S3 BUCKET. SO WE WILL HAVE JSON FILES STORED IN S3 BUCKET IN REAL TIME.
THIS ALL WE WILL BE HAENNING REAL TIME.


SO NOW USING AWS GLUE CRWALER THAT WILL RUN ON S3 BUCKET FOR THUS WE NEED TO CREATE AWS GLUE CRAWLEER AND CONNECT IT TO AWS S3 BUCJET WHERE OUR DATA IS STORING. FOR THIS WE NEED TO MODDIFY IAM TO GIVE ACCES TO THIS CRAWLER TO RUN ON S3 BUCKET.

NOW CREATING DATABASE IN AWS GLUE WHERE CRAWLER WILL HAVE ALL DATA STORING INSTRUTURE MANNER
NOW USING AWS ATHENA AND CONNECTING AWS ATHEN TO AWS GLUE DATABASE CREATED.

NOW PERFOMING SQL QUEIRS ON THUS DATABASE:
EG: SELECT COUNT(*) FROM KAFKA_DATABSE

EVERY SEC WE RUN THIS QUERY WE WILL GET INCREAING COUNT OF RECORDS.


THISS WILL CREATE FULL END TO END BIG DATA PROJECT


WHERE WE HAVE:

STOCK_DATASET -> PYTHON APPLICATION(PRODUCER) -> APACHE KAFKA SERVER(BROKER,TOPIC,PARTION) -> PYTHON APPLICATION(CONSUMER) -> AWS S3 BUCKET(DATA STORAGE) -> AWS GLUE CRAWLER -> AWS GLUE DATA BASE(DATA CATLOGUE) -> AWS ATHENA(DATA ANALYSIS USING SQL)

