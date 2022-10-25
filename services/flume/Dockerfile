FROM openjdk:18-jdk-slim-bullseye

ADD http://archive.apache.org/dist/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz /opt/
RUN cd /opt && tar xvf apache-flume-1.9.0-bin.tar.gz && mv apache-flume-1.9.0-bin flume

ENV PATH /opt/flume/bin:$PATH
WORKDIR /opt/flume
