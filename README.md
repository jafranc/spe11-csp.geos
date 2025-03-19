## spe11-csp.geos

Here are gathered input decks for running SPE11 cases with [GEOS reservoir simulator](https://github.com/GEOS-DEV/GEOS).
It is organized in subfolder per `cases/speX/`. 
Each contains 

1. a base xml (_e.g._ `spe11b_vti_source_base.vti`) containing all re-used information about models and constitutives
2. a driver xml (_e.g._ `spe11b_vti_source_00840x00120.xml`) named after the corresponding discretization, which includes all discretization specific information. 
3. subfolders `cases/speX/{include,mesh,tables}/` are respectively stored sub-xml that stores informations about constitutives, input mesh and input csv tables.

## running with dockers

The subfolder `docker/` contains dockerfiles to build an image from geos' CI-generated thirdparty images (advised) or from scratch. 

Each of them have the following build ARG

| ARG                  | Definition                        | Default                           |
|:---------------------|:----------------------------------|:----------------------------------|
| `DOCKER_ROOT_IMAGE`  | name and tag of the CI image used | **Required**                      |
| `GEOS_TPL_DIR`       | tpl install dir from CI image     | Set from CI image (see below)     |
| `GEOS_SRC_DIR`       | path to geos sources              | `/tmp/src`                        |
| `GEOS_BUILD_DIR`     | path to build dir                 | `${GEOS_SRC_DIR}/build`           |
| `GEOS_INSTALL_DIR`   | path to install geos binaries     | `/opt/GEOS/GEOS-version`          |

*Only* `DOCKER_ROOT_IMAGE` is intended to be set. All other variables should preferably be lest to their default values.

If `GEOS_SRC_DIR` is set to some place else, it will not be removed from image during compilation and can be used for in-docker dev.

The first step is then to chose a base CI image from GEOS' [Dockerhub](https://hub.docker.com/u/geosx). Then one should set `GEOS\_TPL\_DIR` from this base image. 
As written bellow, this will pull the base CI image and set the appropriate env variable. Eventually, one have to compile the docker image.

```bash
export DOCKER_ROOT_IMAGE=geosx/ubuntu22.04-gcc12:301-631
export GEOS_TPL_DIR=$(docker run --rm ${DOCKER_ROOT_IMAGE} /bin/bash -c 'echo ${GEOS_TPL_DIR}')
docker buildx  build --build-arg DOCKER_ROOT_IMAGE=${DOCKER_ROOT_IMAGE} --build-arg GEOS_TPL_DIR=${GEOS_TPL_DIR} -t spe11-geos-docker -f Dockerfile.u22-g12-omp41 .
```

Once build, the command `docker images` should include the newly created image in the list such as,

```bash

REPOSITORY                        TAG       IMAGE ID       CREATED         SIZE
ubuntu			          latest    48108150c33b   2 days ago      2.23GB
spe11-geos-docker                 latest    7fedebd52347   3 days ago      5.84GB
...

```

*This first step can be bypassed by pulling the uploaded geos images 'docker pull jafranc/geos-35a789a:latest'.*

Eventually one wants to run the cases present in this repo, 

```bash

export LOCAL_CASE_PATH=<path-to-local>
git clone https://github.com/jafranc/spe11-csp.geos ${LOCAL_CASE_PATH}
cd ${LOCAL_CASE_PATH} && cd cases/spe11b && ln -s spe11b_vti_source_00840x00120.xml case.xml
docker run --user "$(id -u):$(id -g)" -v ${LOCAL_CASE_PATH}/cases/spe11b:/opt/GEOS/cases --rm jafranc/geos-35a789a

```

Note that `/opt/GEOS/cases` is default for env `${GEOS_CASE_DIR}` but can be set to another destination via `-e <path>` docker run args.
The third instuction line link the case of interest to the generic name 'case.xml' so that the docker image can be instantiated and ran as it is.

Note that `--user` instruction while not mandatory, allows to run the docker as the current user:group. Then the outputs will be assigned to your uid:gid and no rights manipulation will be required.

Note that it can be ran interactively and the call args can be modified.

```bash

docker run -it --user "$(id -u):$(id -g)" --entrypoint /bin/bash -v ${LOCAL_CASE_PATH}/cases/spe11b:/opt/GEOS/cases --rm spe11-geos-docker

```

Eventually the results files will be located locally in `${LOCAL_CASE_PATH}/vtkOutput` and could be processed as proposed in [GEOS documentation](https://geosx-geosx.readthedocs-hosted.com/en/latest/)



