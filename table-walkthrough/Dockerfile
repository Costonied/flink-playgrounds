################################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
################################################################################

# Собираем свой образ контейнера
# Основа образа - последний образ контейнера Apahce Flink (на момент написания этих строк это был 2.1.1)
# Дополнительно в образ добавляем:
# - JAR с нашей, написанной Job-ой (задачей/приложением), который предварительно собираем
# - JAR файлы с библиотеками, необходимыми для работы нашей Job-ы, которые предварительно мы выкачиваем из Maven Repo

# Отличия от ванильной версии примера из Apache Playground репозитория:
# - в ванильной версии используется старая версия Flink (1.6.0)
# - в данной версии Dockerfile пришлось потратить время и поискать какие версии библиотек подходят под Flink 2.1.1
# - кроме несовпадения версий, оказалось, что библиотека flink-connector-jdbc не подходит под Flink 2.1.1, нужно использовать flink-connector-jdbc-core

FROM maven:3.8-eclipse-temurin-17 AS builder

COPY ./pom.xml /opt/pom.xml
COPY ./src /opt/src
RUN cd /opt; mvn clean verify -Dmaven.test.skip

FROM flink:latest

# Download connector libraries - версии для Flink 2.x
# В Flink 2.x JDBC коннектор разделен на модули: core + database-specific
RUN wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka/4.0.1-2.0/flink-sql-connector-kafka-4.0.1-2.0.jar && \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-jdbc-core/4.0.0-2.0/flink-connector-jdbc-core-4.0.0-2.0.jar && \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-jdbc-mysql/4.0.0-2.0/flink-connector-jdbc-mysql-4.0.0-2.0.jar && \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-csv/2.1.1/flink-csv-2.1.1.jar && \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/mysql/mysql-connector-java/8.0.19/mysql-connector-java-8.0.19.jar && \
    ls -la /opt/flink/lib/flink-sql-connector-kafka-4.0.1-2.0.jar /opt/flink/lib/flink-connector-jdbc-core-4.0.0-2.0.jar /opt/flink/lib/flink-connector-jdbc-mysql-4.0.0-2.0.jar /opt/flink/lib/flink-csv-2.1.1.jar

COPY --from=builder /opt/target/spend-report-*.jar /opt/flink/usrlib/spend-report.jar

RUN echo "execution.checkpointing.interval: 10s" >> /opt/flink/conf/flink-conf.yaml; \
    echo "pipeline.object-reuse: true" >> /opt/flink/conf/flink-conf.yaml; \
    echo "pipeline.time-characteristic: EventTime" >> /opt/flink/conf/flink-conf.yaml; \
    echo "taskmanager.memory.jvm-metaspace.size: 256m" >> /opt/flink/conf/flink-conf.yaml;
