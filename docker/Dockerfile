FROM mambaorg/micromamba:2.0.8 AS micromamba

USER root

COPY environment.yaml /tmp/environment.yaml
# Install packages into the empty conda environmemnt base
RUN  micromamba install --yes --name base  --file /tmp/environment.yaml --verbose \
    # Clean package cache
    && micromamba clean --all --yes \
    # List packages installed
    && micromamba env export --name base --explicit

ARG MAMBA_DOCKERFILE_ACTIVATE=1
COPY requirements.txt assets/tflite_runtime-2.20.0-cp312-cp312-linux_x86_64.whl /tmp/
RUN micromamba run --name base pip install tflite_runtime-2.20.0-cp312-cp312-linux_x86_64.whl \
    && micromamba run --name base pip install --no-cache-dir -r /tmp/requirements.txt

FROM ubuntu:jammy

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

# Configure python environment.
# This should be the same prefix 
# as the environment used in micromamba
ENV PYTHON_ENV=/opt/conda

ENV DEBIAN_FRONTEND=noninteractive \ 
    SHELL=/bin/bash \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    LANGUAGE=C.UTF-8 \
    USE_PYGEOS=0 \
    SPATIALITE_LIBRARY_PATH='mod_spatialite.so' \
    HOME="/home/${NB_USER}" \
    PATH="${PYTHON_ENV}/bin:${PATH}" \
    PROJ_DATA="${PYTHON_ENV}/share/proj"

# Delete existing user with UID="${NB_UID}" if it exists
RUN if grep -q "${NB_UID}" /etc/passwd; then \
        userdel --remove $(id -un "${NB_UID}"); \
    fi
# Create a non-root user
RUN useradd --no-log-init --create-home --shell /bin/bash --no-user-group --gid $NB_GID --uid $NB_UID  $NB_USER \
    && chown -R $NB_UID:$NB_GID $HOME
# Copy python environment
COPY --chown=$NB_UID:$NB_GID --from=micromamba /opt/conda $PYTHON_ENV

## Jupyter config

USER $NB_USER
# Enable server extensions
RUN jupyter server extension enable --py --sys-prefix jupyterlab_iframe jupyter_resource_usage \
    && jupyter lab build

USER root
ENV JUPYTERLAB_DIR=$PYTHON_ENV/share/jupyter/lab 
COPY assets/overrides.json $JUPYTERLAB_DIR/settings/
COPY assets/jupyter_lab_config.py $PYTHON_ENV/etc/jupyter/


USER $NB_USER
WORKDIR $HOME
CMD ["jupyter", "lab", \
"--ip=0.0.0.0", \
"--port=9988", \
"--no-browser", \
"--debug"]