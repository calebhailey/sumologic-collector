FROM centos:7
ARG SUMOLOGIC_RPM_VERSION=19.253
ARG SUMOLOGIC_RPM_BUILD=14
RUN \
  yum install -y curl && \
  mkdir -p /etc/sumologic/sources/ && \
  curl -L https://d3pxv6yz143wms.cloudfront.net/8.222.10.1/java-1.8.0-amazon-corretto-devel-1.8.0_222.b10-1.x86_64.rpm -o amazon-jdk.rpm && \
  curl -L https://collectors.us2.sumologic.com/rest/download/rpm/64 -o sumologic.rpm && \
  rpm -i amazon-jdk.rpm && \
  rpm -i sumologic.rpm
CMD ["/opt/SumoCollector/collector", "console"]
