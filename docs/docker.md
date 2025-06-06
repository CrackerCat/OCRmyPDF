# OCRmyPDF Docker image {#docker}

OCRmyPDF is also available in Docker images that packages recent
versions of all dependencies.

For users who already have Docker installed this may be an easy and
convenient option.

On platforms other than Linux, Docker runs in a virtual machine, and so
may be less performant. You may also want to adjust the Docker virtual
machine\'s memory and CPU allocation. On Linux, the Docker image runs
natively and performance is comparable to a system installation.

{#docker-install}
## Installing the Docker image

If you have [Docker](https://docs.docker.com/) installed on your system,
you can install a Docker image of the latest release.

If you can run this command successfully, your system is ready to
download and execute the image:

:::{code} bash
docker run hello-world
:::

:::{list-table} Docker Images
:header-rows: 1

* - Image
  - Architecture
  - Description
* - `jbarlow83/ocrmypdf-alpine`
  - x86_64 and arm64
  - Recommended image, based on Alpine Linux.
* - `jbarlow83/ocrmypdf-ubuntu`
  - x86_64 and arm64
  - Alternate image, based on Ubuntu. When the Alpine image is considered stable and available for arm64, this image will be deprecated.
* - `jbarlow83/ocrmypdf`
  - x86_64 and arm64
  - Currently an alias for ocrmypdf-ubuntu. When the Alpine image is considered stable and available for arm64, this name will point to the Alpine image. If you don\'t know about the difference between Alpine and Ubuntu, use this image.
:::

To install:

:::{code} bash
docker pull jbarlow83/ocrmypdf-alpine
:::

The `ocrmypdf` image is also available, but is deprecated and will be
removed in the future.

OCRmyPDF will use all available CPU cores. See the Docker documentation
for [adjusting memory and CPU on other
platforms](https://docs.docker.com/config/containers/resource_constraints/)
if you are using Docker on macOS or Windows, where you may need to
manually assign more resources. On Linux, all resources will be
available automatically.

The underlying operating system and other details in Docker images are
considered implementation details and **subject to change at minor
releases**. If you are modifying the image, you should pin the version
you intend to use.

## Using the Docker image on the command line

**Unlike typical Docker containers**, in this section the OCRmyPDF
Docker container is ephemeral -- it runs for one OCR job and terminates,
just like a command line program. We are using Docker to deliver an
application (as opposed to the more conventional case, where a Docker
container runs as a server). For that reason we usually use the `--rm`
argument to delete the container when it exits.

To start a Docker container (instance of the image):

:::{code} bash
docker run --rm -i jbarlow83/ocrmypdf-alpine (... all other arguments here...) - -
:::

For convenience, create a shell alias to hide the Docker command. It is
easier to send the input file as stdin and read the output from stdout
-- **this avoids the messy permission issues with Docker entirely**.

:::{code} bash
alias docker_ocrmypdf='docker run --rm -i jbarlow83/ocrmypdf-alpine'
docker_ocrmypdf --version  # runs docker version
docker_ocrmypdf - - <input.pdf >output.pdf
:::

Or in the wonderful [fish shell](https://fishshell.com/):

:::{code} fish
alias docker_ocrmypdf 'docker run --rm jbarlow83/ocrmypdf-alpine'
funcsave docker_ocrmypdf
:::

Alternately, you could mount the local current working directory as a
Docker volume:

:::{code} bash
alias docker_ocrmypdf='docker run --rm  -i --user "$(id -u):$(id -g)" --workdir /data -v "$PWD:/data" jbarlow83/ocrmypdf-alpine'
docker_ocrmypdf /data/input.pdf /data/output.pdf
:::

## Podman

Especially if you use [Podman](https://podman.io/) (or have SELinux
enabled on your system), you may need to add `--userns keep-id` there,
otherwise you may get access errors, because the user is otherwise not
mapped to the same UID as on the host:

:::{code} bash
alias podman_ocrmypdf='podman run --rm  -i --user "$(id -u):$(id -g)" --userns keep-id --workdir /data -v "$PWD:/data" ocrmypdf'
podman_ocrmypdf /data/input.pdf /data/output.pdf
:::

If you use SELinux you may additionally need to add the `:Z` [suffix to
the
volume](https://docs.podman.io/en/stable/markdown/podman-run.1.html#volume-v-source-volume-host-dir-container-dir-options)
or disable SELinux for the container using
`--security-opt label=disable`, which is suggested for system files as
they should not be re-labelled. Please refer to the „Note" section at
the end of the linked podman documentation for details.

{#docker-lang-packs}
## Adding languages to the Docker image

By default the Docker image includes English, German, Simplified
Chinese, French, Portuguese and Spanish, the most popular languages for
OCRmyPDF users based on feedback. You may add other languages by
creating a new Dockerfile based on the public one.

:::{code} dockerfile
FROM jbarlow83/ocrmypdf

# Example: add Italian
RUN apt install tesseract-ocr-ita
:::

To install language packs (training data) such as the
[tessdata\_best](https://github.com/tesseract-ocr/tessdata_best) suite
or custom data, you first need to determine the version of Tesseract
data files, which may differ from the Tesseract program version. Use
this command to determine the data file version:

:::{code} bash
docker run -i --rm --entrypoint /bin/ls jbarlow83/ocrmypdf /usr/share/tesseract-ocr
:::

As of 2021, the data file version is probably `4.00`.

You can then add new data with either a Dockerfile:

:::{code} dockerfile
FROM jbarlow83/ocrmypdf:{TAG}

# Example: add a tessdata_best file
COPY chi_tra_vert.traineddata /usr/share/tesseract-ocr/<data version>/tessdata/
:::

When creating your own image, you should always pin a specific version
of the OCRmyPDF Docker image. This ensures that your image will not
break when a new version of OCRmyPDF is released.

Alternately, you can copy training data into a Docker container as
follows:

:::{code} bash
docker cp mycustomtraining.traineddata name_of_container:/usr/share/tesseract-ocr/<tesseract version>/tessdata/
:::

Extending the Docker image
--------------------------

You can extend the Docker image with your own customizations, similar to
the way it is extended to add language packs.

Note that the Docker image is subject to change at any time. For
example, the base image may be updated to a newer version of Ubuntu or
Debian. Such changes will be noted in the release notes but might occur
at minor versions releases, unless the way a \"casual\" user of the
Docker image is affected.

If you extend the Docker image, you should pin a specific version of the
OCRmyPDF Docker image.

Executing the test suite
------------------------

The OCRmyPDF test suite is installed with image. To run it:

:::{code} bash
docker run --rm --entrypoint python  jbarlow83/ocrmypdf -m pytest
:::

Accessing the shell
-------------------

To use the shell in the Docker image:

:::{code} bash
docker run -it --entrypoint sh  jbarlow83/ocrmypdf
:::

Using the OCRmyPDF web service wrapper
--------------------------------------

The OCRmyPDF Docker image includes an example, barebones HTTP web
service. The webservice may be launched as follows:

:::{code} bash
docker run --entrypoint python -p 5000:5000  jbarlow83/ocrmypdf webservice.py
:::

We omit the `--rm` parameter so that the container will not be
automatically deleted when it exits.

This will configure the machine to listen on port 5000. On Linux
machines this is port 5000 of localhost. On macOS or Windows machines
running Docker, this is port 5000 of the virtual machine that runs your
Docker images. You can find its IP address using the command
`docker-machine ip`.

Unlike command line usage this program will open a socket and wait for
connections.

:::{warning}
The OCRmyPDF web service wrapper is intended for demonstration or
development. It provides no security, no authentication, no protection
against denial of service attacks, and no load balancing. The default
Flask WSGI server is used, which is intended for development only. The
server is single-threaded and so can respond to only one client at a
time. While running OCR, it cannot respond to any other clients.
:::

Clients must keep their open connection while waiting for OCR to
complete. This may entail setting a long timeout; this interface is more
useful for internal HTTP API calls.

Unlike the rest of OCRmyPDF, this web service is licensed under the
Affero GPLv3 (AGPLv3) since Ghostscript is also licensed in this way.

In addition to the above, please read our
`general remarks on using OCRmyPDF as a service <ocr-service>`{.interpreted-text
role="ref"}.
