FROM continuumio/miniconda3:4.10.3p0-alpine

# necessary to display the image on Github
LABEL org.opencontainers.image.source="https://github.com/shirte/nerdd"

# entr is used for live reloading the application
RUN apk update && apk add git entr

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application (e.g., Django logs) in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app

COPY environment.yml .
RUN conda env create -f environment.yml

RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n nerdd_link pip install nerdd-link==0.2.4

# --no-capture-output: output of the application is not buffered and we can see it 
# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT conda run --no-capture-output -n nerdd_link \
    nerdd_job_server \
    --broker-url $KAFKA_BROKER_URL \
    --data-dir /data