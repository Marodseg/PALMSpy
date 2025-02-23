ARG BASE_CONTAINER=openjdk:8-jdk-slim

FROM $BASE_CONTAINER

MAINTAINER Emiliano Molinaro <molinaro@imada.sdu.dk>

LABEL software="HABITUS" \
	  author="Emiliano Molinaro <molinaro@imada.sdu.dk>" \
	  version="1.9.5" \
	  license="MIT" \
	  description="HABITUS is an implementation of the Personal Activity and Location Measurement System (PALMS),\
                   written in Python and integrated with Apache Spark for cluster-computing.\
                   The program detects personal activity patterns of individual participants wearing\
                   a GPS data logger and a physical activity monitor."

ARG SION=${SION:-1.9.5}
ARG user=${user:-"ucloud"}

## Create default user
RUN adduser --shell /bin/bash --uid 11042 --disabled-password --gecos "" $user

## Install software
###################

ARG SPARK_SION=2.4.2
ENV PATH="/opt/miniconda3/bin:${PATH}"
ENV PYSPARK_PYTHON="/opt/miniconda3/bin/python"

RUN apt-get update && \
    apt-get install -y curl bzip2 --no-install-recommends && \
    apt-get install -y libblas-dev libatlas-base-dev build-essential && \
    curl -s --url "https://repo.anaconda.com/miniconda/Miniconda3-py37_4.10.3-Linux-x86_64.sh" --output /tmp/miniconda.sh && \
    bash /tmp/miniconda.sh -b -f -p "/opt/miniconda3" && \
    rm /tmp/miniconda.sh && \
    conda config --set auto_update_conda true && \
    conda config --set channel_priority false && \
    conda update conda -y --force && \
    conda clean -tipsy && \
    echo "PATH=/opt/miniconda3/bin:\${PATH}" > /etc/profile.d/miniconda.sh && \
    python -m pip install --no-cache --upgrade pip && \
    python -m pip install --no-cache pyspark==${SPARK_SION} && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV SPARK_HOME=/opt/miniconda3/lib/python3.7/site-packages/pyspark

RUN SPARK_HOME=$(python /opt/miniconda3/bin/find_spark_home.py) && \
    echo "export SPARK_HOME=$(python /opt/miniconda3/bin/find_spark_home.py)" > /etc/profile.d/spark.sh && \
    curl -s --url "https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar" --output $SPARK_HOME/jars/aws-java-sdk-1.7.4.jar && \
    curl -s --url "https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/2.7.3/hadoop-aws-2.7.3.jar" --output $SPARK_HOME/jars/hadoop-aws-2.7.3.jar && \
    mkdir -p $SPARK_HOME/conf && \
    echo "spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem" >> $SPARK_HOME/conf/spark-defaults.conf

## Create working directory
RUN mkdir /work && \
	chown -R $user:$user /work /opt/miniconda3 $SPARK_HOME

USER 11042

COPY --chown=$user:$user environment.yml /tmp/environment.yml
COPY --chown=$user:$user wheels/habitus-${SION}-py3-none-any.whl /tmp/habitus-${SION}-py3-none-any.whl

RUN sed -i -e "s/x.y.z/$SION/g" /tmp/environment.yml && \
	conda update -n base -c defaults conda && \
    cd /tmp && conda env update -f /tmp/environment.yml && \
    conda clean -tipsy && \
    rm *.yml* *.whl

ENV PATH /opt/miniconda3/envs/habitus/bin:$PATH

## Set shell prompt
RUN echo 'PS1="\[\033[01;31m\]\u\[\033[01;37m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "' >> /home/$user/.bashrc

WORKDIR /work

# ENTRYPOINT ["spark-submit"]
# ENTRYPOINT ["habitus"]
# CMD ["--help"]
