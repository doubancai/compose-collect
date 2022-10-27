FROM openjdk:18-jdk-slim-bullseye

ENV KE_HOME=/opt/efak
ENV EFAK_VERSION=3.0.1
# Set config defaults
ENV EFAK_CLUSTER_ZK_LIST=zookeeper:2181
ENV EFAK_CLUSTER_KAFKA_EAGLE_BROKER_SIZE=1
ENV EFAK_KAFKA_ZK_LIMIT_SIZE=16
ENV EFAK_WEBUI_PORT=8048
ENV EFAK_JMX_ACL=false
ENV EFAK_JMX_USER=keadmin
ENV EFAK_JMX_PASSWORD=keadmin123
ENV EFAK_JMX_SSL=false
ENV EFAK_JMX_TRUSTSTORE_LOCATION=/data/ssl/certificates/kafka.truststore
ENV EFAK_JMX_TRUSTSTORE_PASSWORD=ke123456
ENV EFAK_CLUSTER_KAFKA_EAGLE_OFFSET_STORAGE=kafka
ENV EFAK_METRICS_CHARTS=true
ENV EFAK_METRICS_RETAIN=15
ENV EFAK_SQL_TOPIC_RECORDS_MAX=5000
ENV EFAK_SQL_TOPIC_PREVIEW_RECORDS_MAX=10
ENV EFAK_TOPIC_TOKEN=keadmin
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_ENABLE=false
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_PROTOCOL=SASL_PLAINTEXT
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_MECHANISM=SCRAM-SHA-256
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_JAAS_CONFIG='org.apache.kafka.common.security.scram.ScramLoginModule required username="admin" password="admin-secret";'
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_CLIENT_ID=
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_CGROUP_ENABLE=false
ENV EFAK_CLUSTER_KAFKA_EAGLE_SASL_CGROUP_TOPICS=
ENV EFAK_DB_DRIVER=org.sqlite.JDBC
ENV EFAK_DB_URL=jdbc:sqlite:/hadoop/efak/db/ke.db
ENV EFAK_DB_USERNAME=root
ENV EFAK_DB_PASSWORD=smartloli

ADD system-config.properties /tmp
ADD entrypoint.sh /usr/bin

# RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
# 	echo "deb https://mirrors.aliyun.com/debian/ bullseye main non-free contrib" >>/etc/apt/sources.list && \
# 	echo "deb-src https://mirrors.aliyun.com/debian/ bullseye main non-free contrib" >>/etc/apt/sources.list && \
# 	echo "deb https://mirrors.aliyun.com/debian-security/ bullseye-security main" >>/etc/apt/sources.list && \
# 	echo "deb-src https://mirrors.aliyun.com/debian-security/ bullseye-security main" >>/etc/apt/sources.list && \
# 	echo "deb https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib" >>/etc/apt/sources.list && \
# 	echo "deb-src https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib" >>/etc/apt/sources.list && \
# 	echo "deb https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib" >>/etc/apt/sources.list && \
# 	echo "deb-src https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib" >>/etc/apt/sources.list
	
RUN apt-get update && apt-get upgrade -y && apt-get install -y wget sqlite3 gettext

RUN mkdir -p /opt/efak;cd /opt && \
    wget https://github.com/smartloli/kafka-eagle-bin/archive/v${EFAK_VERSION}.tar.gz && \
    tar zxvf v${EFAK_VERSION}.tar.gz --strip-components 1 && \
	tar zxvf efak-web-${EFAK_VERSION}-bin.tar.gz -C efak --strip-components 1 && \
    echo "export KE_HOME=/opt/efak" >> /etc/profile && \
    echo "export PATH=$PATH:/opt/efak/bin" >> /etc/profile && \
    chmod +x /opt/efak/bin/ke.sh && chmod +x /usr/bin/entrypoint.sh

EXPOSE 8048

ENTRYPOINT ["/usr/bin/entrypoint.sh"]

WORKDIR /opt/efak