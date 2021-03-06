# diedactic
Docker Image Evaluation with Docker-slim And Clair To Identify CVEs

## Overview

This tool measures the security performance of `docker-slim` by analyzing pre- and post- "slimmed" images for CVE counts via `clair`.  Given a list of images, this tool will fetch the images into a private registry for analysis, and provide comparison data on image size and number of CVE vulnerabilities.  Slimmed images are not guarunteed to be functional and should not be used for production without customization and verification.

## Software Requirements

- Docker-slim - https://github.com/docker-slim/docker-slim - binary preferred over running as a containerized command
- Klar - https://github.com/optiopay/klar - client runner for Clair (binary)
- jQ - https://stedolan.github.io/jq/ - CLI-based JSON processor

Install these binaries in the project root directory by running `run-install-deps.sh`

## Hardware Requirements

- Disk space equivalent to store images in private registry
- CPU/RAM to run a private docker registry

## Environment Setup

Ensure the file `images.list` contains a list of images to analyze, one per line.  This repository offers three examples to help you get started:

- `images.example-official.list` contains all "Official" named images on Docker Hub as of Aug 8, 2020.  
- `images.example-shorter.list` contains a subset of images that represent OS layers, apps, and services
- `images.example-short.list` is a simple, shortened list for testing purposes.

Copy any of the above into a file named `images.list` to get started, or create your own. Images in Docker Hub that do not have a `:latest` tag are not currently supported.

The `certs/` directory contains a custom CA certificate and registry certificiate (with associated private key) signed by the custom CA to allow proper TLS connections between `clair` and the private registry.

The scripts used here assume the following ports are open on localhost:

- 5000 and 5443 for private docker registry
- 6060 and 6061 for clair services

Modify values in the `env` file that match your set up as needed.

## USAGE

The `run-*` scripts are separated by task, though each uses the `images.list` file as a driver, looping over each image name.  

### Dependencies

To analyze images, first install dependencies:

1. `./docker-compose up` - Starts `clair` service and private registry (Use separate terminal without `-d` to monitor log output)
1. `./run-install-deps.sh` - Downloads `docker-slim` and `klar` binaries, and ensures `jq` is available.


Create or fill the `images.list` file with at least one image name, one per line. Namespaces and tags (i.e., `:7.8.1`) are not currently supported.

> `images.list`

### Running the Tool

Once the dependencies are installed and the docker services running, use the runner script to do everything in sequence.

> `./run.sh`

Alternatively, run each step as its own command:

1. `./run-pull-images.sh` - Pulls images down from registry
1. `./run-slim-them.sh` - Runs `docker-slim build $image` to create slimmed image, tagged as `$image.slim`
1. `./run-push-to-local-registry-slimmed.sh` - Pushes images and their slimmed copy into the private registry for `clair` access
1. `./run-klar.sh` - For every image in `images.list` and its paired `.slim` image, scan the image for CVEs
1. `./run-summarize-data.sh > summarized-data.csv` - Create a CSV file with image statistics, stored as `summarized-data.csv`

### Cleanup

The `run-cleanup.sh` script will remove log files, and images stored in the private registry.  All slimmed images will be removed from the local docker images list.  

1. `./run-cleanup.sh`

Remove containers and prune images as necessary:

1. `docker-compose down -v`
1. `docker image prune`

## TODO

This project could benefit from additional work in the following areas:

- Optimizing the private registry to local disk communication and storage
- Expanding `images.list` to have more meaning with optional parameters for slimming
  - That is, add extra parameters to each line after the image name
  - Pass options to `docker-slim` to enable/disable the HTTP probe
  - Specify SWAGGER/API definition to support `docker-slim` 1.31 API scanning feature
  - Use test runners on the image for better slimming
  - Support image tags (Only support for `:latest`, right now)
- Update use of `klar` to a better maintained `clair` client tool
- Performing a file-tree analysis on slimmed results to see what differs


## CONTRIBUTING

See `CONTRIBUTING.md` for more details.


## LICENSE

diedactic: Docker Image Evaluation with Docker-slim And Clair To Identify CVEs. This tool measures the security performance of `docker-slim` by analyzing pre- and post- "slimmed" images for CVE counts via `clair`.  Given a list of images, this tool will fetch the images into a private registry for analysis, and provide comparison data on image size and number of CVE vulnerabilities.  Slimmed images are not guarunteed to be functional and should not be used for production without customization.

Copyright (C) 2020 Zac Fowler unless otherwise indicated.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses/.
