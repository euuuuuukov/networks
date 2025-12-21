# ЛР 1. HA Postgres Cluster

## Задача

Развернуть и настроить высокодоступный кластер Postgres

![img.png](img.png)

## Ход работы

### Часть 1. Поднимаем Postgres

1. Подготавливаем Dockerfile для нашего постгреса. Кластеризацию будем делать с помощью [Patroni](https://github.com/patroni/patroni), а ему необходим доступ к бинарникам самого постгреса. Поэтому будем билдить образ, который сразу содержит в себе Postgres + Patroni

```Dockerfile
FROM postgres:15

# Ставим нужные для Patroni зависимости
RUN apt-get update -y && \
    apt-get install -y netcat-openbsd python3-pip curl python3-psycopg2 python3-venv iputils-ping

# Используем виртуальное окружение, доустанавливаем, собственно, Patroni
RUN python3 -m venv /opt/patroni-venv && \
    /opt/patroni-venv/bin/pip install --upgrade pip && \
    /opt/patroni-venv/bin/pip install patroni[zookeeper] psycopg2-binary

# Копируем конфигурацию для двух узлов кластера Patroni
COPY postgres0.yml /postgres0.yml
COPY postgres1.yml /postgres1.yml

ENV PATH="/opt/patroni-venv/bin:$PATH"

USER postgres

# CMD не задаем, т.к. все равно будем переопределять его далее в compose
```