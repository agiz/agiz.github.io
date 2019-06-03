---
layout: post
title: Airflow persistent environment
---


In this post we are going to make Apache Airflow permanently store it's state and logs.
We are going to modify puckel's docker-airflow configuration available at [https://github.com/puckel/docker-airflow](https://github.com/puckel/docker-airflow).
We are going to save Airflow's state to a postgres database. We are using dedicated instance on google cloud.

Open `docker-compose-CeleryExecutor.yml` file. Remove whole `postgres` section as we are going to use dedicated postgres server. Under `webserver`, `flower`, `scheduler` and `worker` add postgres environmental variables:
```
...
environment:
    - POSTGRES_HOST=10.104.0.3
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=<redacted>
    - POSTGRES_DB=airflow
...
```
Remove `postgres` dependency from webserver so it only depends on redis:
```
...
depends_on:
    - redis
...
```

Airflow's state is now stored in the database.

Next, we don't want to clutter our system with large logs. We are going to store all logs on Google storage. First, we prepare `Dockerfile`. Modify `pip install apache-airflow[crypto,...` line by adding `gcp_api` to the mix. So it should look something like: `pip install apache-airflow[crypto,celery,postgres,hive,jdbc,gcp_api,mysql,ssh${AIRFLOW_DEPS:+,}${AIRFLOW_DEPS}]==${AIRFLOW_VERSION}`.

Modify `airflow.cfg` with:
```
...
remote_logging = True
remote_base_log_folder = gs://my-bucket/path/to/logs # use your actual bucket: gs://mydomain.com/mylogs
remote_log_conn_id = MyGCSConn # We will reference this in airflow's web UI: Admin -> Connections
...
```
![_config.yml]({{ site.baseurl }}/images/Screenshot\ 2019-06-03\ at\ 14.16.50.png)
*Figure 1: GCS connection.*

```
Conn Id: MyGCSConn
Conn Type: Google Cloud Platform
Project Id: rippr-220314
Keyfile Path: /usr/local/airflow/dags/keyfile.json
```

All you have to do now is rebuild docker image:
`docker build --rm --build-arg AIRFLOW_DEPS="datadog,dask" --build-arg PYTHON_DEPS="flask_oauthlib>=0.9" -t agiz/docker-airflow:1.10.3 .`
and run:
`docker-compose -f docker-compose-CeleryExecutor.yml up`
