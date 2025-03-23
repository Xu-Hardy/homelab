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
docker run -d --name cocoserver -p 9000:9000 infinilabs/coco:0.2.2-2000
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

### easysearch

见 easysearch.yml
