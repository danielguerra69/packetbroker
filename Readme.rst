##### Packetbroker

Simple xinetd service for receiving PCAP files and publish each packet on a rabbitmq exchange & queue.
It also contains a consumer that writes pcap files to stdout.

### Start RabbitMQ
First you need a rabbitmq, if you have one skip this.
```bash
docker run -d --hostname rabbitmq --name rabbitmq -p 8080:15672 rabbitmq:3-management
```
Check http://<dockerhost>:8080/ (guest,guest) for management

### Start a packerbroker
The packetbroker will start network socket which receives pcap files
```bash
docker run -d --name packetbroker-case1 --link rabbitmq:rabbitmq -p 1989:1969 danielguerra/packetbroker broker rabbitmq guest guest case1 pcap
```
In this case a link to rabbitmq was used. Without the link use dns or ip for rabbitmq.
broker <rabbitmq> <user> <pass> <exchange> <routingkey>
The packetbroker listens to port 1969 but you can set it to any port (above example 1989)
When the broker is started an exchange and queue are created.
The queue name is <exchange>-<routingkey>.

### Send some pcap
then from location case1
```bash
tcpdump -i eth0 -s 0 -w - | nc <docker-ip> <port>
```
or
```bash
nc <docker-ip> <port> < my.pcap
```

### Start consuming
```bash
docker run --rm  -a stdout --link rabbitmq:rabbitmq   danielguerra/packetbroker consumer rabbitmq guest guest case2-pcap >  my.pcap
docker run --rm  -a stdout --link rabbitmq:rabbitmq   danielguerra/packetbroker consumer rabbitmq guest guest case2-pcap | tcpdump -vv -r -
```
