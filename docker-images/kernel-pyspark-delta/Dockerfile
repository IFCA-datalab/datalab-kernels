# PySpark + Delta Lake kernel (kernelspec pyspark_delta_kubernetes), run by
# Jupyter Enterprise Gateway in Spark cluster mode: the same image is the
# driver (which runs the kernel launcher) and the executors.
#
# Base: Jupyter docker-stacks pyspark-notebook (Ubuntu 26.04, Java 21, Spark
# 4.2.0, conda Python), rebuilt every week; pinned by date. It is the same
# family as the other kernel images and the gateway image of this repo.
FROM quay.io/jupyter/pyspark-notebook:2026-10-05

ARG EG_VERSION=3.3.0
ARG SPARK_VERSION=4.2.0
ARG SCALA_VERSION=2.13
ARG DELTA_VERSION=4.4.1

USER root

# Fail the build if the base moved to another Spark: Delta must match it.
RUN test -f "${SPARK_HOME}/jars/spark-core_${SCALA_VERSION}-${SPARK_VERSION}.jar"

RUN apt-get update \
 && apt-get install -yq --no-install-recommends libkrb5-dev \
 && rm -rf /var/lib/apt/lists/*

# Kubernetes entrypoint of this Spark version (docker-stacks images do not ship
# it): driver and executors are started through it. "tini -g" signals the whole
# process group, as in the Enterprise Gateway Spark kernel images.
RUN base="https://raw.githubusercontent.com/apache/spark/v${SPARK_VERSION}/resource-managers/kubernetes/docker/src/main/dockerfiles/spark" \
 && curl -fsSL -o /opt/entrypoint.sh "${base}/entrypoint.sh" \
 && curl -fsSL -o /opt/decom.sh "${base}/decom.sh" \
 && sed -i 's/tini -s/tini -g/g' /opt/entrypoint.sh \
 && chmod a+x /opt/entrypoint.sh /opt/decom.sh \
 && mkdir -p "${SPARK_HOME}/work-dir" && chmod 1777 "${SPARK_HOME}/work-dir"

# Delta Lake and Ceph RGW (Swift/Keystone) support, baked in so kernels do not
# download them from Maven at every start.
RUN cd "${SPARK_HOME}/jars" \
 && m=https://repo1.maven.org/maven2 \
 && curl -fsSLO "${m}/io/delta/delta-spark_${SCALA_VERSION}/${DELTA_VERSION}/delta-spark_${SCALA_VERSION}-${DELTA_VERSION}.jar" \
 && curl -fsSLO "${m}/io/delta/delta-storage/${DELTA_VERSION}/delta-storage-${DELTA_VERSION}.jar" \
 && curl -fsSLO "${m}/io/github/nanhu-lab/hadoop-cephrgw/1.0.3/hadoop-cephrgw-1.0.3.jar" \
 && curl -fsSLO "${m}/org/javaswift/joss/0.10.4/joss-0.10.4.jar"

# Kernel launchers of the Enterprise Gateway release.
ADD https://github.com/jupyter-server/enterprise_gateway/releases/download/v${EG_VERSION}/jupyter_enterprise_gateway_kernel_image_files-${EG_VERSION}.tar.gz /tmp/eg-kernel-files.tar.gz
RUN tar -xzf /tmp/eg-kernel-files.tar.gz -C /usr/local/bin \
 && rm /tmp/eg-kernel-files.tar.gz \
 && chown -R "${NB_UID}:${NB_GID}" /usr/local/bin/kernel-launchers /usr/local/bin/bootstrap-kernel.sh \
 && chmod 0755 /usr/local/bin/bootstrap-kernel.sh

USER ${NB_UID}
# PySpark is the one in ${SPARK_HOME}/python, hence delta-spark without deps.
RUN mamba install --yes cffi future pycryptodomex \
 && mamba clean --all -f -y \
 && pip install --no-cache-dir --no-deps "delta-spark==${DELTA_VERSION}" \
 && fix-permissions "${CONDA_DIR}"

ENV KERNEL_LANGUAGE=python \
    PYSPARK_PYTHON=/opt/conda/bin/python
WORKDIR ${SPARK_HOME}/work-dir
ENTRYPOINT ["/opt/entrypoint.sh"]
