FROM mlebench-env

# Defaults match agents/.shared_env. Empty `--build-arg FOO=$FOO` (unset shell var)
# overrides ARG defaults with ""; ENV lines normalize so mkdir/COPY still work.
ARG SUBMISSION_DIR=/home/submission
ARG LOGS_DIR=/home/logs
ARG CODE_DIR=/home/code
ARG AGENT_DIR=/home/agent

ENV SUBMISSION_DIR=${SUBMISSION_DIR:-/home/submission}
ENV LOGS_DIR=${LOGS_DIR:-/home/logs}
ENV CODE_DIR=${CODE_DIR:-/home/code}
ENV AGENT_DIR=${AGENT_DIR:-/home/agent}

RUN mkdir -p "${LOGS_DIR}" "${CODE_DIR}" "${AGENT_DIR}"

ARG CONDA_ENV_NAME=agent

# copy just the requirements file, so that we can cache conda separately from the agent files
COPY requirements.txt ${AGENT_DIR}/requirements.txt

# Requirements for opencv; patch is used once to extend vendored AIDE (reasoning_effort).
RUN apt-get update && apt-get install -y ffmpeg libsm6 libxext6 patch

# Install deps (aideml pins openai==1.48.0; same resolve cannot also request openai>=1.58).
# Upgrade openai afterward (1.58–1.x) so reasoning_effort works; keep httpx<0.28 alongside.
RUN conda run -n ${CONDA_ENV_NAME} pip install -r ${AGENT_DIR}/requirements.txt && \
    conda run -n ${CONDA_ENV_NAME} pip install --upgrade "openai>=1.58.0,<2" "httpx>=0.27.2,<0.28" && \
    conda clean -afy

# OmegaConf: add reasoning_effort to StageConfig; agent.py passes it only for plan/code (not tool-based feedback).
COPY aideml_reasoning_effort.patch /tmp/aideml_reasoning_effort.patch
RUN AIDE_ROOT=$(conda run -n ${CONDA_ENV_NAME} python -c "import pathlib, aide; print(pathlib.Path(aide.__file__).resolve().parent.parent)") && \
    patch -d "$AIDE_ROOT" -p1 < /tmp/aideml_reasoning_effort.patch && \
    rm -f /tmp/aideml_reasoning_effort.patch

# put all the agent files in the expected location
COPY . ${AGENT_DIR}
