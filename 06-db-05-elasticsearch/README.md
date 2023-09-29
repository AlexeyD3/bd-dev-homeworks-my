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

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

Получите состояние кластера `Elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

