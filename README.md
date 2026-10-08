# DataLab kernels

Kernelspecs served by Jupyter Enterprise Gateway 3.3.0 (namespace
`enterprise-gateway`) and the images of the DataLab's own kernels.

| Kernelspec | Image | Contents |
|---|---|---|
| `python_kubernetes`, `python_tf_kubernetes`, `r_kubernetes`, … | `elyra/kernel-*:3.3.0` | Enterprise Gateway 3.3.0 stock kernels |
| `python_climate` | `ghcr.io/ifca-datalab/kernel-py-climate:eg3.3.0-base2026-10-05` | `quay.io/jupyter/scipy-notebook:2026-10-05` (Python 3.13) + xarray, cartopy, netCDF4, iris, zarr, cdo/nco, TensorFlow… |
| `pyspark_delta_kubernetes` | `ghcr.io/ifca-datalab/kernel-pyspark-delta:spark3.5.9-delta3.3.2` | `spark:3.5.9-scala2.12-java17-python3-ubuntu` (Docker Official Image) + Delta 3.3.2, Ceph RGW jars |

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
- `spark:<version>-…-python3-ubuntu`: Docker Official Image, rebuilt with
  security updates for each Spark release line.

Dependabot (`.github/dependabot.yml`) opens a PR when a new base tag appears.

## Layout

- `<kernel>/`: one kernelspec per folder (EG 3.3.0 format).
- `_images/<image>/Dockerfile`: kernel images, built by
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

Spark kernels: the gateway's `spark-submit` is Spark 3.2.1 (Java 8), so kernel
images stay on Spark 3.x. The Delta and Ceph jars are inside the image; there is
no `spark.jars.packages` download at start-up.
