FROM python:3.9-slim-bullseye

# Allow statements and log messages to immediately appear in the Knative logs
ENV PYTHONUNBUFFERED True

# Copy local code to the container image.
ENV APP_HOME /appgi
WORKDIR $APP_HOME

COPY execute_dvt_partitions/execute_dvt_partition_yamls.sh ./
COPY execute_dvt_partitions/partitions_main.py ./ 
COPY execute_dvt_partitions/partition_connections.sh ./

# Install production dependencies.
RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean
RUN pip install --upgrade pip
RUN pip install --upgrade google-auth
RUN pip install oauth2client
RUN pip install Flask gunicorn
RUN pip install --upgrade google-pso-data-validator

# Install google cloud SDK
RUN apt-get update && apt-get install -y apt-transport-https ca-certificates gnupg curl && \
    curl -sS https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" > /etc/apt/sources.list.d/google-cloud-sdk.list && \
    apt-get update -y && \
    apt-get install google-cloud-sdk -y && \
    apt-get install -y wget unzip libaio1

# Teradata Dependencies
RUN pip install teradatasql
RUN pip install oracledb

# Hive/Impala Dependencies 
# RUN pip install hdfs
# RUN pip install thrift-sasl

# --- Oracle Dependencies ---
# The following steps install the Oracle Instant Client, which is required
# for python-oracledb thick mode.

# 1. Set the Oracle home directory
ENV ORACLE_HOME=/opt/oracle/instantclient_21_12
ENV LD_LIBRARY_PATH=$ORACLE_HOME

# 2. Download and unzip the Oracle Instant Client
RUN mkdir -p $ORACLE_HOME && \
    cd $ORACLE_HOME && \
    wget https://download.oracle.com/otn_software/linux/instantclient/2112000/instantclient-basiclite-linux.x64-21.12.0.0.0dbru.zip && \
    unzip instantclient-basiclite-linux.x64-21.12.0.0.0dbru.zip && \
    rm instantclient-basiclite-linux.x64-21.12.0.0.0dbru.zip && \
    mv instantclient_21_12/* . && \
    rmdir instantclient_21_12

# 3. Configure the dynamic linker
RUN echo $ORACLE_HOME > /etc/ld.so.conf.d/oracle.conf && \
    ldconfig

# Return to the app directory
WORKDIR $APP_HOME
# --- End of Oracle Dependencies ---

# UNCOMMENT TO RUN AS SERVICE
# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
# Timeout is set to 0 to disable the timeouts of the workers to allow Cloud Run to handle instance scaling.
# CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 --timeout 0 dvt_main:app

# UNCOMMENT TO RUN AS JOB
CMD ["python", "partitions_main.py"]
