# Домашнее задание к занятию 5. «Elasticsearch»

## Задача 1

В этом задании вы потренируетесь в:

- установке Elasticsearch,
- первоначальном конфигурировании Elasticsearch,
- запуске Elasticsearch в Docker.

Используя Docker-образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для Elasticsearch,
- соберите Docker-образ и сделайте `push` в ваш docker.io-репозиторий,
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины.

Требования к `elasticsearch.yml`:

- данные `path` должны сохраняться в `/var/lib`,
- имя ноды должно быть `netology_test`.

В ответе приведите:

- текст Dockerfile-манифеста,

<details>
<summary>dockerfile</summary>

```
FROM centos:7.9.2009

RUN yum install perl-Digest-SHA -y

ENV ES_DIR="/opt/elasticsearch"
ENV ES_HOME="${ES_DIR}/elasticsearch-8.10.2"
ENV ES_USER="elasticsearch"

WORKDIR ${ES_DIR}

COPY elasticsearch-8.10.2-linux-x86_64.tar.gz  ${ES_DIR}
COPY elasticsearch-8.10.2-linux-x86_64.tar.gz.sha512  ${ES_DIR}

RUN shasum -a 512 -c elasticsearch-8.10.2-linux-x86_64.tar.gz.sha512
RUN tar -xzf elasticsearch-8.10.2-linux-x86_64.tar.gz
RUN rm elasticsearch-8.10.2-linux-x86_64.tar.gz

COPY elasticsearch.yml ${ES_HOME}/config

RUN useradd ${ES_USER}

RUN mkdir -p "/var/lib/elasticsearch"
RUN mkdir -p "/var/log/elasticsearch"

RUN chown -R ${ES_USER}: "${ES_DIR}"
RUN chown -R ${ES_USER}: "/var/lib/elasticsearch"
RUN chown -R ${ES_USER}: "/var/log/elasticsearch"

USER ${ES_USER}

WORKDIR "${ES_HOME}"

EXPOSE 9200
EXPOSE 9300

ENTRYPOINT ["./bin/elasticsearch"]
```
</details>

<details>
<summary>elasticsearch.yml</summary>

```
discovery:
  type: single-node

cluster:
  name: netology

node:
  name: netology_test

http:
  host: 0.0.0.0
  port: 9200

xpack.security.enabled: false

path:
  data: /var/lib/elasticsearch
  logs: /var/log/elasticsearch
```
</details>

- ссылку на образ в репозитории dockerhub,
- ответ `Elasticsearch` на запрос пути `/` в json-виде.

```
[wolinshtain@localhost netology-alma]$ sudo docker run -d --rm --name elastic -p 9200:9200 devopsalexey/alexnetology:eltest2
ed871c0cad7f0e3b74691eece511a8cf655965535843842469c0d349bd50ec98
[wolinshtain@localhost netology-alma]$ sudo docker exec -it elastic bash
[elasticsearch@ed871c0cad7f elasticsearch-8.10.2]$ curl -XGET http://localhost:9200
{
  "name" : "netology_test",
  "cluster_name" : "netology",
  "cluster_uuid" : "sqrMblApTsCspWE89kAsfw",
  "version" : {
    "number" : "8.10.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "6d20dd8ce62365be9b1aca96427de4622e970e9e",
    "build_date" : "2023-09-19T08:16:24.564900370Z",
    "build_snapshot" : false,
    "lucene_version" : "9.7.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

```

Подсказки:

- возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
- при некоторых проблемах вам поможет Docker-директива ulimit,
- Elasticsearch в логах обычно описывает проблему и пути её решения.

Далее мы будем работать с этим экземпляром Elasticsearch.

## Задача 2

В этом задании вы научитесь:

- создавать и удалять индексы,
- изучать состояние кластера,
- обосновывать причину деградации доступности данных.

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d' { "settings": { "number_of_shards": 1, "number_of_replicas": 0 }}'

[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'

[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'

```

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 FMxvoKXiTm2BrLWfWtpRcg   1   0          0            0       248b           248b
yellow open   ind-3 z36-8Pf3QuuxxM3MS8-PPg   4   2          0            0       992b           992b
yellow open   ind-2 SBTO1sJPT1yD0YeHX1pjIw   2   1          0            0       496b           496b
```

Получите состояние кластера `Elasticsearch`, используя API.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET "localhost:9200/_cluster/health?pretty"
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 7,
  "active_shards" : 7,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 41.17647058823529
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty'
{
  "cluster_name" : "netology",
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
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty'
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty'
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 33.33333333333333
}

```

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

> мы указали количество реплик, но имеем в кластере только одну ноду, из-за этого по двум индексам у нас появились нераспределенные шарды "unassigned_shards" и уменьшился процент активных шардов "active_shards_percent_as_number"

Удалите все индексы.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X DELETE "localhost:9200/ind-1,ind-2,ind-3?pretty"
{
  "acknowledged" : true
}
```

**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

```
[wolinshtain@localhost netology-alma]$ sudo docker exec -u root -it elastic bash
[root@027d00f391c1 elasticsearch-8.10.2]# mkdir $ES_DIR/snapshots
[root@027d00f391c1 elasticsearch-8.10.2]# chown elasticsearch:elasticsearch $ES_DIR/snapshots
[root@027d00f391c1 elasticsearch-8.10.2]# echo path.repo: [ "$ES_DIR/snapshots" ] >> "$ES_HOME/config/elasticsearch.yml"
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ exit
exit
[wolinshtain@localhost netology-alma]$ sudo docker restart elastic
elastic
[wolinshtain@localhost netology-alma]$ sudo docker exec -it elastic bash
```

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.


**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -XPOST localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d'{"type": "fs", "settings": { "location":"myrepo" }}'
{
  "acknowledged" : true
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/_snapshot/netology_backup?pretty'
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "myrepo"
    }
  }
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ 
```

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET 'http://localhost:9200/test?pretty'
{
  "test" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test",
        "creation_date" : "1696068721066",
        "number_of_replicas" : "0",
        "uuid" : "AW301RHgTre7x0kHFWrhIQ",
        "version" : {
          "created" : "8100299"
        }
      }
    }
  }
}
```

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
{"snapshot":{"snapshot":"elasticsearch","uuid":"MOI8owFNReu1in-iAqnI7g","repository":"netology_backup","version_id":8100299,"version":"8100299","indices":["test"],"data_streams":[],"include_global_state":true,"state":"SUCCESS","start_time":"2023-09-30T10:22:37.177Z","start_time_in_millis":1696069357177,"end_time":"2023-09-30T10:22:37.177Z","end_time_in_millis":1696069357177,"duration_in_millis":0,"failures":[],"shards":{"total":1,"failed":0,"successful":1},"feature_states":[]}}

[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ ls -lh $ES_DIR/snapshots/myrepo
total 36K
-rw-r--r--. 1 elasticsearch elasticsearch 589 Sep 30 10:22 index-0
-rw-r--r--. 1 elasticsearch elasticsearch   8 Sep 30 10:22 index.latest
drwxr-xr-x. 3 elasticsearch elasticsearch  36 Sep 30 10:22 indices
-rw-r--r--. 1 elasticsearch elasticsearch 23K Sep 30 10:22 meta-MOI8owFNReu1in-iAqnI7g.dat
-rw-r--r--. 1 elasticsearch elasticsearch 305 Sep 30 10:22 snap-MOI8owFNReu1in-iAqnI7g.dat
```

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X DELETE "localhost:9200/test?pretty"
{
  "acknowledged" : true
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl 'localhost:9200/_cat/indices?pretty'
green open test-2 ldhMGEs7SmuGZZ-2t9BIUQ 1 0 0 0 226b 226b
```


[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

```
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl 'localhost:9200/_cat/indices?pretty'
green open test-2 ldhMGEs7SmuGZZ-2t9BIUQ 1 0 0 0 226b 226b
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X POST localhost:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H 'Content-Type: application/json' -d'{"include_global_state":true}'
{
  "accepted" : true
}
[elasticsearch@027d00f391c1 elasticsearch-8.10.2]$ curl -X GET http://localhost:9200/_cat/indices?v
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 ldhMGEs7SmuGZZ-2t9BIUQ   1   0          0            0       226b           226b
green  open   test   dGCA255tRQC8CgEVkDG9BA   1   0          0            0       248b           248b
```

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

