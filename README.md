# homelab

这些都是在群晖上，折腾的，小小的身板承受了他不该承受的东西

### wikijs

见 wikijs.yml

### 打印机改无线 cups

```bash
sudo docker run -d --name=airprint --net=host --privileged=true -e TZ="Asia/Shanghai" -e CUPSADMIN="admin" -e CUPSPASSWORD="pass" -e HOST_OS="Synology" -e TCP_PORT_631="631" chuckcharlie/cups-avahi-airprint:latest
```

也可以用 compose （未测试） cups.yml

### Coco AI

```bash
docker run -d --name cocoserver -p 9900:9000 infinilabs/coco:0.2.2-2000
```

带目录映射：

```bash
docker run -d \
           --name cocoserver \
           --hostname coco-server \
           --restart unless-stopped \
           -m 4g \
           --cpus="2" \
           -p 9000:9000 \
           -v $(pwd)/cocoserver/data:/app/easysearch/data \
           -v $(pwd)/cocoserver/logs:/app/easysearch/logs \
           -e EASYSEARCH_INITIAL_ADMIN_PASSWORD=coco-server \
           -e ES_JAVA_OPTS="-Xms2g -Xmx2g" \
           infinilabs/coco:0.2.2-2000
```

也可以用 compose （未测试）

### openspeedtest

```bash
docker run --restart=unless-stopped --name openspeedtest -d -p 80:3000 -p 443:3001 openspeedtest/latest
```

### Minio

https://min.io/docs/minio/container/index.html

```bash
mkdir -p ~/minio/data

docker run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v ~/minio/data:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server /data --console-address ":9001"
```

### harbor

### Adguard Home

### only office

### HA

```bash
sudo docker run -d --name="home-assistants" --net=host homeassistant/home-assistant
```

### NginxProxyManger

### nginxWebUI

### nginxconfig

https://github.com/digitalocean/nginxconfig.io

## ES 家族

### easysearch

见 easysearch.yml

### elasticsearch

```yml
version: "3.8"

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - bootstrap.system_call_filter=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - bootstrap.system_call_filter=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - bootstrap.system_call_filter=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:7.10.2
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://es01:9200
    ports:
      - 5601:5601
    depends_on:
      - es01
    networks:
      - elastic

volumes:
  data01:
  data02:
  data03:

networks:
  elastic:
    driver: bridge
```

### opensearch

```yml
version: "3.7"

services:
  opensearch-node1:
    image: opensearchproject/opensearch:2.19.1
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node1
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true
      - bootstrap.system_call_filter=false
      - DISABLE_SECURITY_PLUGIN=true
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
    ports:
      - 9400:9200
      - 9600:9600
    networks:
      - opensearch-net

  opensearch-node2:
    image: opensearchproject/opensearch:2.19.1
    container_name: opensearch-node2
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node2
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true
      - bootstrap.system_call_filter=false
      - DISABLE_SECURITY_PLUGIN=true
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
    networks:
      - opensearch-net

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2.19.1
    container_name: opensearch-dashboards
    ports:
      - 5602:5601
    expose:
      - "5601"
    environment:
      - OPENSEARCH_HOSTS=["http://opensearch-node1:9200","http://opensearch-node2:9200"]
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```

### doocs 排版工具

```bash
docker run --name=doocs -d -p 8080:80 doocs/md:latest
```
