FROM alpine:latest
ARG version=2.4.4

LABEL maintainer="Carlos Giraldo <cgiraldo@gradiant.org>" \
      organization="gradiant.org"
ENV SPARK_VERSION=$version \
    SPARK_HOME=/opt/spark \
    SPARK_NO_DAEMONIZE=true \
    JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk/jre
ENV PATH=$PATH:$SPARK_HOME/sbin:$SPARK_HOME/bin \
    SPARK_LOCAL_DIRS=$SPARK_HOME/work-dir \
    SPARK_WORKER_DIR=$SPARK_HOME/worker
# You may improve spark access to fs if SPARK_LOCAL_DIRS and SPARK_WORKER_DIR are mounted as volumes

# Adding Hadoop native libs compiled for alpine linux
COPY --from=gradiant/hadoop-base:2.7.7-alpine /opt/hadoop/lib/native/* /lib/

RUN set -ex && \
    ln -s /lib /lib64 && \
    apk add --no-cache \
        bash \
        tini \
        linux-pam \
        procps \
        coreutils \
        libc6-compat \
        openjdk8-jre \
        snappy \
        zlib \
        && mkdir -p /opt && \
   wget -qO- https://archive.apache.org/dist/spark/spark-$SPARK_VERSION/spark-$SPARK_VERSION-bin-hadoop2.7.tgz | tar xvz -C /opt && \
   ln -s /opt/spark-$SPARK_VERSION-bin-hadoop2.7 /opt/spark && \
   mkdir -p /opt/spark/work-dir && \
   mkdir -p /opt/spark/worker && \
   touch /opt/spark/RELEASE && \
   rm /bin/sh && \
   ln -sv /bin/bash /bin/sh && \
   echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
   chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
   cp /opt/spark/kubernetes/dockerfiles/spark/entrypoint.sh /opt/entrypoint.sh && \
   ln -s /opt/spark/kubernetes/tests /opt/spark/tests

# upgrading kubernetes-client.jar to 4.4.2 to FIX BUG https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/issues/591
RUN rm $SPARK_HOME/jars/kubernetes-client-*.jar
ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar $SPARK_HOME/jars

ADD https://repo1.maven.org/maven2/org/apache/spark/spark-avro_2.11/$SPARK_VERSION/spark-avro_2.11-$SPARK_VERSION.jar $SPARK_HOME/jars

RUN cp $SPARK_HOME/examples/jars/spark-examples_2.11-$SPARK_VERSION.jar $SPARK_HOME/jars/

WORKDIR $SPARK_HOME/local

COPY standalone /opt/spark/sbin/standalone

ENTRYPOINT [ "/opt/entrypoint.sh" ]


