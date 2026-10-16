# GitLab CI Overview

This project is used to build and push QIMSDK Docker containers from CLO source.
Its `.gitlab-ci.yml` builds and validates an ARM64 Docker image for the QIMSDK deploy runtime.

## What the pipeline does

- Uses `docker:25.0.3-dind` on an `arm64-runner`
- Starts Docker-in-Docker in `before_script`
- Defines image coordinates for:
  - `artifacts.codelinaro.org/iot-solutions-microservices/qimsdk`
  - versioned tag: `2.1.0`
  - `latest` tag
- Clones `sdk-tools` from CodeLinaro
- Checks out a pinned commit
- Builds the QIMSDK and QIMSDK python deploy images from `qimsdk-debian/Dockerfile`
- Passes build arguments for:
  - `QIMSDK_ARG_QNP_VERSION`
  - `QIMSDK_ARG_CAMERA_SERVICE_TAG`
  - `QIMSDK_ARG_GST_PLUGINS_TAG`
  - `QIMSDK_ARG_SOLUTIONS_MICROSERVICES_TAG`
  - `QIMSDK_ARG_GST_QTI_OSS_PATCHES_TAG`

- Pushes both version-specific and latest image tags to the JFrog registry for the QIMSDK and QIMSDK Python Deploy images.
- Executes smoke tests in a dedicated pipeline stage by launching containers from both the version-specific and latest tags of the QIMSDK and QIMSDK Python Deploy images.

## Pipeline stages

### `build`
Builds the QIMSDK and QIMSDK Python runtime images and pushes:
- `${IMAGE_NAME}:${IMAGE_TAG}`
- `${IMAGE_NAME}:latest`

### `test`
Runs a basic smoke test against all pushed images to confirm:
- the container starts successfully
- the OS image is accessible
- the runtime image is available as expected

## Build configuration

The pipeline uses the following key settings:

- **Registry:** `artifacts.codelinaro.org`
- **Image path:** `iot-solutions-microservices`
- **Image name:** `qimsdk`
- **Image tag:** `2.1.0`
- **Source repo:** `https://git.codelinaro.org/clo/le/sdk-tools.git`
- **Source branch:** `imsdk-tools.lnx.1.0`
- **Pinned commit sdk tools:** `fcc71f5f12791f77d8174cb5772eab6c785578d7`
- **Pinned QAIRT version:** `2.50.0.260828`
- **Pinned commit qti gst patches:** `a4db6968a1f467875407a2008d7c13974a826fe4`
- **Pinned tag camera service:** `1.0.6`
- **Pinned tag qimsdk:** `2.0.2`
- **Build target native:** `qimsdk_deploy_arm64`
- **Build target python:** `qimsdk_deploy_py_arm64`

## Kubernetes resource requirements

The pipeline requests the following Kubernetes resources:

- **CPU:** `16`
- **Memory:** `128G`
- **Ephemeral storage:** `256G`
- **Pod annotation:** `MOUNT_EFS=/data`

## Updating the source version

To build images from the latest CLO source, update:

- `QIMSDK_TOOLS_COMMIT`

to the desired commit from `QIMSDK_TOOLS_REPO_URL`.

If any build arguments also need to change, update:

- `QIMSDK_QAIRT_VERSION`
- `QIMSDK_CAMERA_SERVICE_COMMIT`
- `QIMSDK_GST_PLUGINS_COMMIT`
- `QIMSDK_SOLUTIONS_MICROSERVICES_COMMIT`
- `QIMSDK_GST_QTI_PATCHES_COMMIT`

**Important:** whenever the source versions are updated, the **versioned image tag must also be updated**, since a new container image will be produced from the new source baseline.

After updating the CI file, the pipeline will rebuild the image and push the new version to the registry.

## Smoke test behavior

The test stage runs a simple validation for both:
- `latest`
- the versioned image tag

It confirms that the image can be pulled and executed successfully.

---

This repository contains only the README and `.gitlab-ci.yml`, so the CI file is the main source of build and release behavior.