# Python kernel for the IPCC / climate environment, launched by Jupyter
# Enterprise Gateway (kernelspec python_climate).
#
# An elyra/kernel-py image is just a Jupyter docker-stacks image plus the
# Enterprise Gateway kernel launchers, but it is only rebuilt with each EG
# release. docker-stacks publishes a new, dated image every week: use one of
# those (pinned by date) and add the launchers of the EG release ourselves.
FROM quay.io/jupyter/scipy-notebook:2026-10-05

ARG EG_VERSION=3.3.0

USER root
RUN apt-get update \
 && apt-get install -yq --no-install-recommends libkrb5-dev \
 && rm -rf /var/lib/apt/lists/*

# Kernel launchers and bootstrap script of the Enterprise Gateway release.
ADD https://github.com/jupyter-server/enterprise_gateway/releases/download/v${EG_VERSION}/jupyter_enterprise_gateway_kernel_image_files-${EG_VERSION}.tar.gz /tmp/eg-kernel-files.tar.gz
RUN tar -xzf /tmp/eg-kernel-files.tar.gz -C /usr/local/bin \
 && rm /tmp/eg-kernel-files.tar.gz \
 && chown -R "${NB_UID}:${NB_GID}" /usr/local/bin/kernel-launchers /usr/local/bin/bootstrap-kernel.sh \
 && chmod 0755 /usr/local/bin/bootstrap-kernel.sh

USER ${NB_UID}

# Launcher dependencies + the climate stack of the previous image (2023), minus
# classic-notebook extensions, which do nothing in a remote kernel. TensorFlow
# comes from PyPI: its wheels follow new Python versions sooner than conda-forge.
RUN mamba install --yes \
      cffi future pycryptodomex \
      xarray cartopy netcdf4 cftime dask iris zarr s3fs h5py gcsfs siphon \
      seaborn nco cdo \
 && mamba clean --all -f -y \
 && pip install --no-cache-dir tensorflow-cpu \
 && fix-permissions "${CONDA_DIR}" \
 && fix-permissions "/home/${NB_USER}"

ENV KERNEL_LANGUAGE=python
CMD ["/usr/local/bin/bootstrap-kernel.sh"]
