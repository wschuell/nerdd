FROM continuumio/miniconda3:4.10.3p0-alpine

# rdkit requires libxrender, libxext-dev and libxau
RUN apk update && apk add git libxrender libxext-dev libxau

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app
COPY environment.yml .

RUN conda env create -f environment.yml

COPY ./ ./

# --mount=type=cache,target=/root/.cache/pip: create a cache directory for pip, so that
# it doesn't have to download the dependencies every time we build the image
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n cypstrate pip install -e .

# install nerdd-kafka
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n cypstrate pip install nerdd-kafka

# --no-capture-output: output of the application is not buffered and we can see it 
# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT conda run --no-capture-output -n cypstrate \
    run_nerdd_server cypstrate.CypstrateModel \
    --broker-url my-cluster-kafka-bootstrap.kafka:9092 
