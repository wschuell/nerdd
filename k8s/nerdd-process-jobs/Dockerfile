FROM continuumio/miniconda3:4.10.3p0-alpine

# entr is used for live reloading the application
RUN apk update && apk add git entr

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application (e.g., Django logs) in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app
COPY ./nerdd-link/environment.yml .

RUN conda env create -f environment.yml

COPY ./nerdd-link ./

RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n nerdd_link pip install -e . 

# first line: watch the current directory and re-run the command if something changes
# --no-capture-output: output of the application is not buffered and we can see it 
# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT echo . | entr -dnr \
    conda run --no-capture-output -n nerdd_link \
    nerdd_job_server \
    --broker-url my-cluster-kafka-bootstrap.kafka:9092 \
    --data-dir /data