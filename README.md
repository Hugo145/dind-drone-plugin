# Docker-in-Docker Drone plugin

This is a plugin for **Drone 0.8** that is aimed mainly at enabling [Testcontainers](https://www.testcontainers.org) to be used during CI build/test steps. 
Due to Drone's architecture, Docker-in-Docker is often the most practical way to run builds that require a Docker daemon.

This plugin:

* Is based upon an Docker-in-Docker image
* Includes a startup script that:
	* Starts a nested docker daemon
	* Optionally starts a pull of required images (in parallel with your build, so as to reduce overall time spent waiting for images to be pulled)
	* Starts a specified build container inside the Docker-in-Docker context, containing your source code and with a docker socket available to it

## Prerequisites

Either:

* (Drone 0.8): To enable on a per-repository basis, enable the *Trusted* setting for the repository. *Or*
* (Drone 0.8): To enable this plugin globally in your Drone instance, add the image name to the `DRONE_ESCALATE` environment variable that the Drone process runs under.

## Usage/Migration (Drone 0.8)

Modify the `build` step of the pipeline to resemble:

```yaml
pipeline:
  ...
  build:
    image: quay.io/testcontainers/dind-drone-plugin
    build_image: openjdk:8-jdk-alpine
    # This specifies the command that should be executed to perform build, test and integration tests. Not to be confused with Drone's `command`:
    cmd: ./gradlew clean check --info
    # Not mandatory, but enables pre-fetching of images in parallel with the build, so may save time:
    prefetch_images:
      - "redis:4.0.6"
```

When migrating to use this plugin from an ordinary build step, note that:

* `commands` should be changed to `cmd`. Note that _commas_ are not supported within `cmd` lines due to the way these are passed in between Drone and this plugin.
* `image` should be changed to `build_image`
* `prefetch_images` is optional, but recommended. This specifies a list of images that should be pulled in parallel with your build process, thus saving some time.

## Copyright

This repository contains code which was mainly developed at [Skyscanner](https://www.skyscanner.net/jobs/), and is licenced under the [Apache 2.0 Licence](LICENSE).

(c) 2017-2019 Skyscanner Ltd.
