ARG ELK_VERSION

# https://github.com/elastic/elasticsearch-docker
FROM docker.elastic.co/elasticsearch/elasticsearch-oss:${ELK_VERSION}

COPY config/elasticsearch.yml config/elasticsearch.yml
COPY config/sg/ config/sg/
COPY bin/ bin/
COPY search-guard-tlstool-1.8/ search-guard-tlstool-1.8/

RUN chown -R elasticsearch search-guard-tlstool-1.8 && \
    chmod +x search-guard-tlstool-1.8/tools/sgtlsdiag.sh && \
    chmod +x search-guard-tlstool-1.8/tools/sgtlstool.sh

RUN yum update -y && \
    yum install -y wget && \
    yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel && \
    yum clean all

ENV JAVA_HOME /usr/lib/jvm/jre-1.8.0-openjdk/

RUN search-guard-tlstool-1.8/tools/sgtlstool.sh -c search-guard-tlstool-1.8/config/elk.yml -ca -crt -t config/

# check if certs are validated
RUN search-guard-tlstool-1.8/tools/sgtlsdiag.sh -ca config/root-ca.pem -crt config/node1.pem && \
    search-guard-tlstool-1.8/tools/sgtlsdiag.sh -ca config/root-ca.pem -crt config/admin.pem

#RUN cat config/node1_elasticsearch_config_snippet.yml >> config/elasticsearch.yml

# Search Guard plugin
ARG ELK_VERSION
ARG SG_VERSION
RUN elasticsearch-plugin install --batch https://releases.floragunn.com/search-guard-7/${ELK_VERSION}-${SG_VERSION}/search-guard-7-${ELK_VERSION}-${SG_VERSION}.zip \
	&& chmod +x plugins/search-guard-7/tools/*.sh \
	&& chown -R elasticsearch config/sg/ \
	&& chmod -R go= config/sg/ \
        && chown -R elasticsearch config/ \
        && chmod -R go= config/

USER elasticsearch

RUN mkdir ssl && \
    cp config/root-ca.pem ssl/

