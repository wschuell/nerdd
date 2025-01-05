FROM continuumio/miniconda3:4.10.3p0-alpine

# necessary to display the image on Github
LABEL org.opencontainers.image.source="https://github.com/shirte/nerdd"

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app

# create the conda environment
COPY environment.yml .
RUN conda env create -f environment.yml

COPY . .

# --mount=type=cache,target=/root/.cache/pip: create a cache directory for pip, so that
# it doesn't have to download the dependencies every time we build the image
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n nerdd_backend pip install .

# --no-capture-output: output of the application is not buffered and we can see it 
# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT conda run --no-capture-output -n nerdd_backend \
    python -m nerdd_backend.main --config-name production \
    ++db.host=$RETHINK_DB_HOST ++db.port=$RETHINK_DB_PORT ++channel.broker_url=$KAFKA_BROKER_URL