# Jupyter Enterprise Gateway with a current Spark for the Spark kernelspecs.
#
# The official elyra/enterprise-gateway image is built from
# jupyter/minimal-notebook:2023-03-13 (Ubuntu 22.04) with Java 8 and Spark 3.2.1:
# kernelspecs run that spark-submit to create the driver pods. Same recipe
# (upstream etc/docker/enterprise-gateway/Dockerfile) on docker-stacks
# pyspark-notebook: Ubuntu 26.04, Java 21, Spark 4.2.0.
FROM quay.io/jupyter/pyspark-notebook:2026-10-05

ARG EG_VERSION=3.3.0
ARG EG_RELEASE=https://github.com/jupyter-server/enterprise_gateway/releases/download/v${EG_VERSION}
ARG EG_SOURCE=https://raw.githubusercontent.com/jupyter-server/enterprise_gateway/v${EG_VERSION}

# The kernelspecs (and the upstream image) use /opt/spark.
USER root
RUN ln -s "${SPARK_HOME}" /opt/spark
ENV SPARK_HOME=/opt/spark

USER ${NB_UID}
# EG 3.3.0 pins an old Jupyter stack (pyzmq<25, jupyter-client<7,
# jupyter-server<2) and supports Python 3.10-3.11 only: pyzmq 24 has no wheels
# for the base's Python 3.13. Give the gateway its own Python 3.11 env, first
# on PATH, and leave the base environment untouched.
# ipykernel 7 has an asyncio rewrite that keeps kernels from becoming idle (upstream note).
RUN mamba create --yes -n eg python=3.11 cffi send2trash requests future pycryptodomex \
 && mamba clean --all -f -y \
 && "${CONDA_DIR}/envs/eg/bin/pip" install --no-cache-dir "ipykernel<7" "jupyter-enterprise-gateway==${EG_VERSION}" \
 && fix-permissions "${CONDA_DIR}"
ENV PATH=${CONDA_DIR}/envs/eg/bin:${PATH}

USER root
ADD ${EG_RELEASE}/jupyter_enterprise_gateway_kernelspecs-${EG_VERSION}.tar.gz /tmp/kernelspecs.tar.gz
ADD ${EG_RELEASE}/jupyter_enterprise_gateway_kernel_image_files-${EG_VERSION}.tar.gz /tmp/kernel-files.tar.gz
ADD ${EG_SOURCE}/etc/docker/enterprise-gateway/start-enterprise-gateway.sh /usr/local/bin/start-enterprise-gateway.sh
RUN mkdir -p /usr/local/share/jupyter/kernels \
 && tar -xzf /tmp/kernelspecs.tar.gz -C /usr/local/share/jupyter/kernels \
 && tar -xzf /tmp/kernel-files.tar.gz -C /usr/local/bin \
 && rm -f /tmp/kernelspecs.tar.gz /tmp/kernel-files.tar.gz /usr/local/bin/bootstrap-kernel.sh \
 && chmod 0755 /usr/local/bin/start-enterprise-gateway.sh \
 && touch /usr/local/share/jupyter/enterprise-gateway.log \
 && chmod 0666 /usr/local/share/jupyter/enterprise-gateway.log \
 && chown -R "${NB_UID}:${NB_GID}" /usr/local/share/jupyter /usr/local/bin/kernel-launchers /usr/local/bin/start-enterprise-gateway.sh

USER ${NB_UID}
WORKDIR /usr/local/bin
EXPOSE 8888
CMD ["/usr/local/bin/start-enterprise-gateway.sh"]
