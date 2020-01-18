FROM docker.elastic.co/elasticsearch/elasticsearch:7.5.1
RUN elasticsearch-plugin install analysis-kuromoji

ADD jvm.options /usr/share/elasticsearch/config/
ADD elasticsearch.yml /usr/share/elasticsearch/config/
USER root
RUN chown elasticsearch:elasticsearch \
  /usr/share/elasticsearch/data \
  /usr/share/elasticsearch/config \
  /usr/share/elasticsearch/config/jvm.options \
  /usr/share/elasticsearch/config/elasticsearch.yml

USER elasticsearch
EXPOSE 9200 9300
