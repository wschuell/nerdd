FROM continuumio/miniconda3:4.10.3p0-alpine

# rdkit requires libxrender, libxext-dev and libxau
# entr is used for live reloading the application
RUN apk update && apk add git libxrender libxext-dev libxau entr

# Necessary, so Docker doesn't buffer the output and that we can see the output 
# of the application in real-time.
ENV PYTHONUNBUFFERED 1

WORKDIR /app

# Copy only the environment file first, so that we can cache the dependencies
COPY environment.yml .
RUN conda env create -f environment.yml

RUN mkdir /deps

# Copy the nerdd-module source code directory and install it
# --mount=type=cache,target=/root/.cache/pip: create a cache directory for pip, so that
# it doesn't have to download the dependencies every time we build the image
COPY --from=nerdd-module . /deps/nerdd-module
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n cyplebrity pip install -e /deps/nerdd-module

# Copy the nerdd-link source code directory and install it
COPY --from=nerdd-link . /deps/nerdd-link
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n cyplebrity pip install -e /deps/nerdd-link

# Copy the rest of the source code directory and install the main package
COPY . .
RUN --mount=type=cache,target=/root/.cache/pip \
    conda run -n cyplebrity pip install -e .

# first line: watch the current directory and re-run the command if something changes
# --no-capture-output: output of the application is not buffered and we can see it 
# Note: PYTHONUNBUFFERED is not enough, because it only affects the Python
# standard output, not the output of the application.
ENTRYPOINT echo . | entr -dnr \
    conda run --no-capture-output -n cyplebrity \
    nerdd_prediction_server cyplebrity.CyplebrityModel \
    --broker-url my-cluster-kafka-bootstrap.kafka:9092 
