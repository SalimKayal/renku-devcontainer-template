FROM registry.renkulab.io/rok.roskar/vscode-example:0c10599 as code-server
FROM renku/renkulab-r:4.3.1-0.23.0 AS rstudio-server
FROM renku/renkulab-py:3.10-0.23.0 as jupyterlab
FROM ${templateOption:ide}

COPY environment.yml* .devcontainer/noop.txt /tmp/conda-tmp/
COPY requirements.txt* .devcontainer/noop.txt /tmp/conda-tmp/

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
