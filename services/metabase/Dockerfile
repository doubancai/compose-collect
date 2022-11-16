FROM --platform=linux/amd64 metabase/metabase:v0.44.6

ADD https://github.com/enqueue/metabase-clickhouse-driver/releases/download/0.8.1/clickhouse.metabase-driver.jar /plugins/

RUN chmod 744 /plugins/clickhouse.metabase-driver.jar
