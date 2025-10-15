FROM python:3.9-slim

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
    apt-get install google-cloud-sdk -y

# Teradata Dependencies
RUN pip install teradatasql
RUN pip install oracledb

# Hive/Impala Dependencies 
# RUN pip install hdfs
# RUN pip install thrift-sasl

# --- Oracle Dependencies ---
# The following steps install the Oracle Instant Client, which is required
# for the Python cx_Oracle library to connect to an Oracle database.

# 1. Set Environment Variables for Oracle
# These tell the system where to find Oracle's files once installed.
# Make sure the version matches the RPMs you downloaded.
ENV ORACLE_ODBC_VERSION 12.2
ENV ORACLE_HOME /usr/lib/oracle/${ORACLE_ODBC_VERSION}/client64
ENV LD_LIBRARY_PATH $ORACLE_HOME/lib

# 2. Install the Python driver and system tools
# - cx_Oracle: The Python library for connecting to Oracle.
# - alien: A critical tool that converts Red Hat packages (.rpm) to Debian packages (.deb) so they can be installed.
# - libaio1: A required library for the Oracle client.
RUN pip install cx_Oracle && \
    apt-get update && \
    apt-get install -y alien libaio1

# 3. Copy and install the Oracle RPMs you downloaded
# This takes the .rpm files from your local 'oracle' folder,
# copies them into the container, installs them using 'alien',
# and then cleans up the leftover .rpm files.
COPY oracle/*.rpm ./
RUN alien -i --scripts *.rpm && rm *.rpm

# 4. Configure the dynamic linker
# This tells the operating system where to find the Oracle libraries
# so that applications (like Python) can use them.
RUN echo $ORACLE_HOME/lib > /etc/ld.so.conf.d/oracle.conf && \
    ldconfig
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
