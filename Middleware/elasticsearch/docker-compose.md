```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.5.1
    container_name: elasticsearch
    networks:
    - net-es
    volumes:
    - ../data/elasticsearch/data:/usr/share/elasticsearch/data　　#这里将elasticsearch的数据文件映射本地，以保证下次如果删除了容器还有数据
    environment:
    - discovery.type=single-node
    ports:
    - "9200:9200"
 
  elastichd:
    image: containerize/elastichd:latest
    container_name: elasticsearch-hd
    networks:
    - net-es
    ports:
      - "9800:9800"
    depends_on:
      - "elasticsearch"
    links:
      - "elasticsearch:demo"
 
#这里要注意，es和eshd要在相同网络才能被links
networks:
  net-es:
    external: false
```



```yaml
version: '2.2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_01
    environment:
      - cluster.name=lloveyou
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_02
    environment:
      - cluster.name=iloveyou
      - node.name=es7_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data2:/usr/share/elasticsearch/data
    networks:
      - es7net
volumes:
  es7data1:
    driver: local
  es7data2:
    driver: local

networks:
  es7net:
    driver: bridge
```





```yaml
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - es72net
  kibana:
    image: kibana:7.2.0
    container_name: kibana72
    environment:
      #- I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es72net
  elasticsearch:
    image: elasticsearch:7.2.0
    container_name: es72_01
    environment:
      - cluster.name=mydemo
      - node.name=es72_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es72_01,es72_02
     #- network.publish_host=elasticsearch
      - cluster.initial_master_nodes=es72_01,es72_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es72data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es72net
  elasticsearch2:
    image: elasticsearch:7.2.0
    container_name: es72_02
    environment:
      - cluster.name=mydemo
      - node.name=es72_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es72_01,es72_02
     #- network.publish_host=elasticsearch
      - cluster.initial_master_nodes=es72_01,es72_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es72data2:/usr/share/elasticsearch/data
    networks:
      - es72net


volumes:
  es72data1:
    driver: local
  es72data2:
    driver: local

networks:
  es72net:
    driver: bridge

```

