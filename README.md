# 6-5-Elasticsearch

##  Задача 1

dockerfile

    FROM centos:7
    RUN \
        cd / && \
        yum install wget -y && \
        wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-linux-x86_64.tar.gz && \
        tar -xzf elasticsearch-7.15.0-linux-x86_64.tar.gz
    
    EXPOSE 9200
    EXPOSE 9300
    
    RUN ["groupadd", "elasticsearch"]
    RUN ["useradd", "elasticsearch", "-g", "elasticsearch", "-p", "elasticsearch"]
    RUN ["cd", "/opt"]
    RUN ["chown", "-R", "elasticsearch:elasticsearch", "/elasticsearch-7.15.0"]
    ## for path.data
    RUN ["chown", "-R", "elasticsearch:elasticsearch", "/var/lib/"]
    
    ADD elasticsearch.yml /elasticsearch-7.15.0/config/elasticsearch.yml
    
    WORKDIR /elasticsearch-7.15.0/
    
    USER elasticsearch
    
    CMD ["./bin/elasticsearch"]
    


Файл конфигурации elasticsearch.yml

    node.name: "Netology_Test"
    cluster.name: mycluster1
    node.master: true
    node.data: true
    network.bind_host: localhost
    path.data: /var/lib


команда построения образа

    docker build -t es2 -f dockerfile .

команда запуска контейнера

    docker run --name esearch2 -it -p 9200:9200 -p 9300:9300 es2

вывод curl

    curl -X GET "localhost:9200"
    
    {
     "name" : "Netology_Test",
      "cluster_name" : "mycluster1",
      "cluster_uuid" : "aN6sE2cpRMquLmQzyS-SlA",
      "version" : {
      "number" : "7.15.0",
      "build_flavor" : "default",
      "build_type" : "tar",
      "build_hash" : "79d65f6e357953a5b3cbcc5e2c7c21073d89aa29",
      "build_date" : "2021-09-16T03:05:29.143308416Z",
      "build_snapshot" : false,
      "lucene_version" : "8.9.0",
      "minimum_wire_compatibility_version" : "6.8.0",
      "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"

=======================================

    c:\Oleg>docker ps -a
    CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS                      PORTS                                                                                      NAMES
    6b9153169883   es2                      "./bin/elasticsearch"    About a minute ago   Up About a minute           0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300-     >9300/tcp, :::9300->9300/tcp   esearch2

Используемый образ

    c:\Oleg>docker images
    REPOSITORY                     TAG       IMAGE ID       CREATED          SIZE
    es2                            latest    f9665fdd18d0   45 minutes ago   1.85GB


Сборка нового образа и пуш его в репозиторий

    c:\Oleg>docker images
    REPOSITORY                     TAG       IMAGE ID       CREATED          SIZE
    elastic-or                     latest    3d77fb67d226   26 seconds ago   1.98GB
    
    
    c:\Oleg>docker tag elastic-or olegrovenskiy/elastic-01.v1
    
    c:\Oleg>docker images
    REPOSITORY                     TAG       IMAGE ID       CREATED          SIZE
    elastic-or                     latest    3d77fb67d226   2 minutes ago    1.98GB
    olegrovenskiy/elastic-01.v1    latest    3d77fb67d226   2 minutes ago    1.98GB
    
отправка образа

    docker push olegrovenskiy/elastic-01.v1

ссылка на образ

https://hub.docker.com/repository/docker/olegrovenskiy/elastic-01.v1


##  Задача 2


Создание индексов

    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl -XPUT 'localhost:9200/ind-1?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" :         {"number_of_shards" : 1, "number_of_replicas" : 0 }}}'
    {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "ind-1"
    }
    
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl -XPUT 'localhost:9200/ind-2?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" :     {"number_of_shards" : 2, "number_of_replicas" : 1 }}}'
    {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "ind-2"
    }
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl -XPUT 'localhost:9200/ind-3?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" :       {"number_of_shards" : 4, "number_of_replicas" : 2 }}}'
    {
     "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "ind-3"
    }
    
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .geoip_databases 7ngYPTY8Q3qAPC5OvrVHgQ   1   0         39            0     37.9mb         37.9mb
    green  open   ind-1            iwbrVO0ETPaLh8LdU9Kx4Q   1   0          0            0       208b           208b
    yellow open   ind-3            6DDdRux2QReSNeI1rltGsg   4   2          0            0       208b           208b
    yellow open   ind-2            71GG1ny9TGWxfS_dm2QDYw   2   1          0            0       416b           416b
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$
    
Статус кластера после создания индексов

    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl -XGET 'localhost:9200/_cluster/health?pretty'
    {
      "cluster_name" : "mycluster1",
      "status" : "yellow",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 8,
      "active_shards" : 8,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 10,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 44.44444444444444
    }


при создании индексов ind-2, ind-3, статус кластера становится жёлтым, т.к. мы указали 1 и 2реплики, 
которые не могут работать, т.к. у нас одна нода


Удаляем индексы

    curl -XDELETE localhost:9200/ind-1
    curl -XDELETE localhost:9200/ind-2
    curl -XDELETE localhost:9200/ind-3

статус кластера

    curl -XGET 'localhost:9200/_cluster/health?pretty'
    
    
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$ curl -XGET 'localhost:9200/_cluster/health?pretty'
    {
      "cluster_name" : "mycluster1",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 1,
      "active_shards" : 1,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
    }
    [elasticsearch@6b9153169883 elasticsearch-7.15.0]$


##  Задача 3

изменение в elasticsearch.yml

добавляем строчку

    path.repo: /elasticsearch-7.15.0/snapshots

и перезапуск elasticsearch

Создание директории и её регистрация

    mkdir -p /elasticsearch-7.15.0/snapshots/netology_backup
    
    curl -XPUT "http://localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d '{"type": "fs", "settings": {"location": "netology_backup"}}'
    
    
    drwxrwxr-x 2 elasticsearch elasticsearch 4096 Oct  1 18:24 netology_backup
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl -XPUT "http://localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d '{"type":     "fs", "settings": {"location": "netology_backup"}}
    >
    > '
    {
      "acknowledged" : true
    }
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$


Создайте индекс test с 0 реплик и 1 шардом

    curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" : {"number_of_shards" : 1, "number_of_replicas" : 0 }}}'
    
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" :    {"number_of_shards" : 1, "number_of_replicas" : 0 }}}'
    {
      "acknowledged" : true,
     "shards_acknowledged" : true,
      "index" : "test"
    }
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$

приведите в ответе список индексов.

    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .geoip_databases dFNOIeGORJy1g2BPAFZ4IQ   1   0         43            0     41.1mb         41.1mb
    green  open   test             uNzmFic2TBKVkTI-ISOv4g   1   0          0            0       208b           208b
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$
    
    curl -X PUT "localhost:9200/_snapshot/netology_backup/snapshot_1?wait_for_completion=true&pretty"
    
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl -X PUT "localhost:9200/_snapshot/netology_backup/snapshot_1?wait_for_completion=true&pretty"
    {
      "snapshot" : {
      "snapshot" : "snapshot_1",
      "uuid" : "Jlb7HijHSliAEYUXRqtVig",
      "repository" : "netology_backup",
      "version_id" : 7150099,
      "version" : "7.15.0",
      "indices" : [
      ".geoip_databases",
       "test"
       ],
       "data_streams" : [ ],
       "include_global_state" : true,
       "state" : "SUCCESS",
       "start_time" : "2021-10-01T18:39:52.515Z",
       "start_time_in_millis" : 1633113592515,
       "end_time" : "2021-10-01T18:40:03.044Z",
       "end_time_in_millis" : 1633113603044,
       "duration_in_millis" : 10529,
       "failures" : [ ],
       "shards" : {
       "total" : 2,
       "failed" : 0,
       "successful" : 2
      },
       "feature_states" : [
          {
            "feature_name" : "geoip",
            "indices" : [
            ".geoip_databases"
            ]
          }
        ]
      }
    }
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$

Приведите в ответе список файлов в директории со snapshotами.

    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ ls -l /elasticsearch-7.15.0/snapshots/netology_backup/
    total 44
    -rw-r--r-- 1 elasticsearch elasticsearch   828 Oct  1 18:40 index-0
    -rw-r--r-- 1 elasticsearch elasticsearch     8 Oct  1 18:40 index.latest
    drwxr-xr-x 4 elasticsearch elasticsearch  4096 Oct  1 18:39 indices
    -rw-r--r-- 1 elasticsearch elasticsearch 27096 Oct  1 18:40 meta-Jlb7HijHSliAEYUXRqtVig.dat
    -rw-r--r-- 1 elasticsearch elasticsearch   437 Oct  1 18:40 snap-Jlb7HijHSliAEYUXRqtVig.dat
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$
    
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ ls -l /elasticsearch-7.15.0/snapshots/netology_backup/indices/
    total 8
    drwxr-xr-x 3 elasticsearch elasticsearch 4096 Oct  1 18:40 Vu-hU50GTnK2G2fiTMsiNw
    drwxr-xr-x 3 elasticsearch elasticsearch 4096 Oct  1 18:40 _vobTaU3R1WHGHSHQ5ZFOA
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$
    
Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.
    
    curl -XDELETE localhost:9200/test
    
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl -XDELETE localhost:9200/test
    {"acknowledged":true}[elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .geoip_databases dFNOIeGORJy1g2BPAFZ4IQ   1   0         43            0     41.1mb         41.1mb
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$

создание индекса test-2

    curl -XPUT 'localhost:9200/test-2?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" : {"number_of_shards" : 1, "number_of_replicas" : 0 }}}'
    
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl -XPUT 'localhost:9200/test-2?pretty' -H 'Content-Type: application/json' -d'{"settings" : {"index" :      {"number_of_shards" : 1, "number_of_replicas" : 0 }}}'
    {
     "acknowledged" : true,
     "shards_acknowledged" : true,
     "index" : "test-2"
    }
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .geoip_databases dFNOIeGORJy1g2BPAFZ4IQ   1   0         43            0     41.1mb         41.1mb
    green  open   test-2           Y6OcZ1nsR4y_9isaJUxPwQ   1   0          0            0       208b           208b
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$

востановление кластера из снепшота

    curl -X POST "localhost:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty"

Востановление индекса test из снепшота


    curl -X POST "localhost:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d' {"indices": "test"}'


    [elasticsearch@416722af8411 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .geoip_databases dFNOIeGORJy1g2BPAFZ4IQ   1   0         43            0     41.1mb         41.1mb
    green  open   test-2           Y6OcZ1nsR4y_9isaJUxPwQ   1   0          0            0       208b           208b
    green  open   test             E4yUp_jTR1SmqYsiHNcLgQ   1   0          0            0       208b           208b
    [elasticsearch@416722af8411 elasticsearch-7.15.0]$

Баг с geoip и способ его решения

Есть статья на гитхабе
GeoIpDownloader can't be disable by elasticsearch.yml #76586
https://github.com/elastic/elasticsearch/issues/76586


исправляется корректировкой файла конфигурации

прописать в elasticsearch.yml для исключения геоайпи

    ingest.geoip.downloader.enabled: false

и перезапуск elasticsearch

после чего geoip БД не грузится и индекс геоайпи ДБ отсутствует

    [elasticsearch@1e15514baf30 elasticsearch-7.15.0]$ curl 'localhost:9200/_cat/indices?v'
    health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   test  4hHfAnm3R0GZlb6yrEG2Tg   1   0          0            0       208b           208b
    [elasticsearch@1e15514baf30 elasticsearch-7.15.0]$

