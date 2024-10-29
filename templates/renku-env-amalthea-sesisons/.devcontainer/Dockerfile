FROM docker.io/r-base:latest as r

USER root
# Generally, Dev Container Features assume that the non-root user (in this case jovyan)
# is in a group with the same name (in this case jovyan). So we must first make that so.
RUN useradd -m user \
    && usermod -g user -a -G staff,users user

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      tini \
    && rm -rf /var/lib/apt/lists/*

USER user

RUN R -e "install.packages('renv', repos = c(CRAN = 'https://cloud.r-project.org'))"

FROM quay.io/condaforge/miniforge3:latest as python

USER root
# Generally, Dev Container Features assume that the non-root user (in this case jovyan)
# is in a group with the same name (in this case jovyan). So we must first make that so.
RUN useradd -m user \
    && usermod -g user -a -G staff,users user

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      tini \
    && rm -rf /var/lib/apt/lists/*

ENV SHELL=/bin/bash

# Copy environment.yml (if found) to a temp location so we update the environment. Also
# copy "noop.txt" so the COPY instruction does not fail if no environment.yml exists.
COPY requirements.txt* environment.yml* .devcontainer/noop.txt /tmp/conda-tmp/

USER user

RUN <<EOF
set -euf -o pipefail
eval "`/opt/conda/bin/conda shell.bash hook 2>/dev/null`"
if [ -f "/opt/conda/etc/profile.d/mamba.sh" ]; then
  source "/opt/conda/etc/profile.d/mamba.sh"
fi
mamba init
mamba create -y -n project ipykernel pip
mamba activate project
if [ -f "/tmp/conda-tmp/environment.yml" ]; then umask 0002
    mamba env update -n project -f /tmp/conda-tmp/environment.yml
fi
if [ -f "/tmp/conda-tmp/requirements.txt" ]; then umask 0002
    pip install -r /tmp/conda-tmp/requirements.txt
fi
python -m ipykernel install --name project --display-name "Project environment" --user
echo "mamba activate project" >> $HOME/.bashrc
EOF

FROM r as universal

ENV SHELL=/bin/bash

COPY --from=python --chown=user:user /opt/conda /opt/conda
COPY --from=python --chown=user:user /home/user/.bashrc /home/user/.bashrc
COPY --from=python --chown=user:user /home/user/.conda /home/user/.conda
COPY --from=python --chown=user:user /home/user/.local/share/jupyter /home/user/.local/share/jupyter

FROM quay.io/condaforge/miniforge3:latest as jupyterlab

RUN eval "`/opt/conda/bin/conda shell.bash hook 2>/dev/null`" && \
    conda create -y --prefix=/ide jupyterlab

RUN <<EOF
echo '#!/bin/bash' > /entrypoint.sh
echo '/ide/bin/jupyter lab --no-browser --ip=0.0.0.0 --NotebookApp.token="" --NotebookApp.password=""' >> /entrypoint.sh
EOF

FROM quay.io/condaforge/miniforge3:latest as code-server

RUN eval "`/opt/conda/bin/conda shell.bash hook 2>/dev/null`" && \
    conda create -y --prefix=/ide code-server && \
    mkdir -p /ide/share/code-server/extensions && \
    chmod g+rwx /ide/share/code-server/extensions


RUN <<EOF
echo '#!/bin/bash' > /entrypoint.sh
echo 'export PATH=/ide/bin:$PATH' >> /entrypoint.sh
echo '/ide/bin/code-server --host 0.0.0.0 --auth none --port 8888' >> /entrypoint.sh
EOF

FROM ${templateOption:ide} as ide

FROM ${templateOption:language}

COPY --from=ide --chown=root:staff /ide /ide
COPY --from=ide --chmod=755 --chown=root:staff /entrypoint.sh /entrypoint.sh

WORKDIR /home/user/

ENV XDG_DATA_HOME=/home/user/.local/share

ENTRYPOINT ["tini", "--", "/entrypoint.sh"]
CMD [""]
