# Chapter 1: Quick Start

This guide walks you through the quickest way to setup a recommender system for GitHub repositories. Make sure you have installed following softwares at the beginning: 

- [Docker](https://docs.docker.com/get-docker/), nodes will run in containers.
- [Docker Compose](https://docs.docker.com/compose/install/), nodes will be orchestrated by Docker Compose.

## Setup Gorse

There is an example [docker-compose.yml](https://github.com/zhenghaoz/gorse/blob/master/docker/docker-compose.yml) consists of a master node, a server node and a worker node, a Redis instance, and a MySQL instance.

- Download [docker-compose.yml](https://github.com/zhenghaoz/gorse/blob/master/docker/docker-compose.yml) and the config file [config.toml](https://github.com/zhenghaoz/gorse/blob/master/docker/config.toml). The config file is explained in [Section 1.3 Configuration](ch01-03-config.md).

```bash
# Create a new directory
mkdir gorse
cd gorse

# Download docker-compose.yml and config.tom
wget https://raw.githubusercontent.com/zhenghaoz/gorse/master/docker/docker-compose.yml
wget https://raw.githubusercontent.com/zhenghaoz/gorse/master/docker/config.toml
```

- Setup the Gorse cluster using Docker Compose.

```bash
docker-compose up -d
```

<details>
<summary>Expected outputs:</summary>

```
Creating network "gorse_default" with the default driver
Creating gorse_worker_1 ... done
Creating gorse_master_1 ... done
Creating gorse_mysql_1  ... done
Creating gorse_server_1 ... done
Creating gorse_redis_1  ... done
```

</details>

- Download the SQL file [github.sql](https://cdn.gorse.io/example/github.sql) and import to the MySQL instance. This dataset consists of GitHub users, GitHub repositories and star records.

```bash
# Download sample data.
wget https://cdn.gorse.io/example/github.sql

# Import sample data.
mysql -h 127.0.0.1 -u gorse -pgorse_pass gorse < github.sql
```

- Restart the master node to apply imported data.

```bash
docker-compose restart
```

<details>
<summary>Expected outputs:</summary>

```
Restarting gorse_redis_1  ... done
Restarting gorse_mysql_1  ... done
Restarting gorse_master_1 ... done
Restarting gorse_server_1 ... done
Restarting gorse_worker_1 ... done
```

</details>

## Get Recommendation

Gorse generate recommendations for users after feedbacks from users are inserted.

- Open [http://127.0.0.1:8088](http://127.0.0.1:8088) in browser and log in with default user name `admin` and password `password`. The dashboard summarizes all status and data in Gorse. Read [Section 2.5 Gorse Dashboard](ch02-05-dashboard.md) for more information.

![](img/ch3/gorse-dashboard-overview.png)

- Wait for all tasks *except "Searching collaborative filtering model" and "Searching click-through prediction model"* completed.

![](/img/ch3/gorse-dashboard-tasks.png)

- Suppose Bob is a frontend developers who starred several frontend repositories in GitHub. We insert his star feedback to Gorse via the RESTful API. [Section 1.2](ch01-02-restful-api.md) lists RESTful APIs available in Gorse.

```bash
read -d '' JSON << EOF
[
    { \"FeedbackType\": \"star\", \"UserId\": \"bob\", \"ItemId\": \"vuejs:vue\", \"Timestamp\": \"2022-02-24\" },
    { \"FeedbackType\": \"star\", \"UserId\": \"bob\", \"ItemId\": \"d3:d3\", \"Timestamp\": \"2022-02-25\" },
    { \"FeedbackType\": \"star\", \"UserId\": \"bob\", \"ItemId\": \"dogfalo:materialize\", \"Timestamp\": \"2022-02-26\" },
    { \"FeedbackType\": \"star\", \"UserId\": \"bob\", \"ItemId\": \"mozilla:pdf.js\", \"Timestamp\": \"2022-02-27\" },
    { \"FeedbackType\": \"star\", \"UserId\": \"bob\", \"ItemId\": \"moment:moment\", \"Timestamp\": \"2022-02-28\" }
]
EOF

curl -X POST http://127.0.0.1:8087/api/feedback \
   -H 'Content-Type: application/json' \
   -d "$JSON"
```

<details>
<summary>Expected outputs:</summary>

```json
{
 "RowAffected": 5
}
```

</details>

- Fetch 10 recommended items from Gorse. We can found frontend related repositories are recommended for Bob.

```bash
curl http://127.0.0.1:8087/api/recommend/bob?n=10
```

Expected outputs:

```json
[
 "mbostock:d3",
 "nt1m:material-framework",
 "mdbootstrap:vue-bootstrap-with-material-design",
 "justice47:f2-vue",
 "10clouds:cyclejs-cookie",
 "academicpages:academicpages.github.io",
 "accenture:alexia",
 "addyosmani:tmi",
 "1wheel:d3-starterkit",
 "acdlite:redux-promise"
]
```

A complete example to build a recommender system is discussed in [Section 1.5 GitRec, The Live Demo](ch01-04-demo.md), which is useful to learn Gorse.