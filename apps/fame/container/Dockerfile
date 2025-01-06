FROM continuumio/miniconda3:4.10.3p0-alpine

# necessary to display the image on Github
LABEL org.opencontainers.image.source="https://github.com/shirte/nerdd"

# rdkit requires libxrender, libxext-dev and libxau
# entr is used for live reloading the application
RUN apk update && apk add git libxrender libxext-dev libxau g++

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app

# Copy only the environment file first, so that we can cache the dependencies
COPY environment.yml .
RUN conda env create -f environment.yml

# Copy the rest of the source code directory and install the main package
COPY . .
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n fame3 pip install .

RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n fame3 pip install nerdd-link==0.2.8

# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT conda run --no-capture-output -n fame3 \
    nerdd_prediction_server fame.Fame3Model \
    --broker-url $KAFKA_BROKER_URL \
    --data-dir /data