DOCKER RUN APP
docker build -t app-service:v1.0 .

$ docker run -d -p 8099:8099 app-service:v1.0

External db in connection string: docker.for.mac.host.internal


POSTGRES
docker exec -it [container id] psql -U postgres
ALTER USER postgres WITH PASSWORD 'postgres';



KONG
KONGA
http://localhost:1337
Connection: http://kong:8001
Admin/5337393


Kafka
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic app-visits --from-beginning

kafka-console-consumer.sh --zookeeper 10.204.22.7:2181 --from-beginning --topic app-visits


ElasticSearch
es index     127.0.0.1:9200,9300 app-vistis-2019.1.14
kafka topic  127.0.0.1:9092      topic: app-visits


    

