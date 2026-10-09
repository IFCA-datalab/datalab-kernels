# DataLab kernels

Kernelspecs served by Jupyter Enterprise Gateway 3.3.0 (namespace
`enterprise-gateway`) and the images of the DataLab's own kernels.

| Kernelspec | Image | Contents |
|---|---|---|
| `python_kubernetes`, `python_tf_kubernetes`, `r_kubernetes`, … | `elyra/kernel-*:3.3.0` | Enterprise Gateway 3.3.0 stock kernels |
| `python_climate` | `ghcr.io/ifca-datalab/kernel-py-climate:eg3.3.0-base2026-10-05` | `quay.io/jupyter/scipy-notebook:2026-10-05` (Python 3.13) + xarray, cartopy, netCDF4, iris, zarr, cdo/nco, TensorFlow… |
| `pyspark_delta_kubernetes` | `ghcr.io/ifca-datalab/kernel-pyspark-delta:spark4.2.0-delta4.4.1-base2026-10-05` | `quay.io/jupyter/pyspark-notebook:2026-10-05` (Ubuntu 26.04, Java 21, Spark 4.2.0) + Delta 4.4.1, Ceph RGW jars |

The volumes and namespace of each environment are not in the kernelspecs: the
hub sends `KERNEL_NAMESPACE`, `KERNEL_SERVICE_ACCOUNT_NAME`, `KERNEL_VOLUMES`
and `KERNEL_VOLUME_MOUNTS` (see the hub configmaps of datalab-api).

## Base images

The `elyra/kernel-*` images are only rebuilt with each Enterprise Gateway
release. An EG kernel image is just a regular image plus the EG kernel
launchers (`jupyter_enterprise_gateway_kernel_image_files-<version>.tar.gz`,
from the EG GitHub release), so ours start from bases that are maintained:

- `quay.io/jupyter/*-notebook`: Jupyter docker-stacks, a new dated tag every
  week. Pin the date tag (`python-3.x` tags freeze once the series moves on).

Dependabot (`.github/dependabot.yml`) opens a PR when a new base tag appears.

## Gateway image

`docker-images/enterprise-gateway` rebuilds the upstream gateway image on the
same base: `elyra/enterprise-gateway:3.3.0` has Java 8 and Spark 3.2.1 (its
`spark-submit` creates the driver pods of the Spark kernels) on Ubuntu 22.04.
EG 3.3.0 declares Python 3.10–3.11 and the base has 3.13: try the image next to
the current gateway before switching the Helm release (`image:` value) to it.

## Layout

- `<kernel>/`: one kernelspec per folder (EG 3.3.0 format).
- `docker-images/<image>/Dockerfile`: kernel images, built by
  `.github/workflows/kernel-images.yml` and pushed to `ghcr.io/ifca-datalab/`.
- `_deploy/sync-to-jeg.sh`: copies the kernelspecs to the gateway's
  `kernelspecs` volume. It refuses to run while an image is missing.
- `_backup-2026-10-07/`: kernelspecs before the 3.3.0 update (not in Git).

## Updating a kernel

1. Change its Dockerfile and the tag in the workflow matrix; push to `main`.
2. When the workflow finishes, point the kernelspec at the new tag.
3. `_deploy/sync-to-jeg.sh`
4. Allow new kernelspecs in the gateway (`kernel.allowedKernels` of the
   enterprise-gateway Helm release).

Spark kernels: the gateway's `spark-submit` (3.2.1) only builds the driver pod;
the driver runs the image's own Spark, so kernel images can use Spark 4. The Delta and Ceph jars are inside the image; there is
no `spark.jars.packages` download at start-up.
