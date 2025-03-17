## spe11-csp.geos

Here are gathered input decks for running SPE11 cases with [GEOS reservoir simulator](https://github.com/GEOS-DEV/GEOS).
It is organized in subfolder per _cases/speX/_. 
Each contains a base xml (_e.g._ spe11b\_vti\_source\_base.vti) containing all re-used information about models and constitutives and a driver xml (_e.g._ spe11b\_vti\_source\_00840x00120.xml) named after the corresponding discretization, which includes all discretization specific information. 
In subfolders _cases/speX/{include,mesh,tables}/_ are respectively stored sub-xml that stores informations about constitutives, input mesh and input csv tables.

## running with dockers

The subfolder _docker/_ contains dockerfiles to build an image from geos' CI-generated thirdparty images (advised) or from scratch. 

Each of them have the following build ARG

+----------------------+-----------------------------------+-----------------------------------+
| ARG                  | Definition                        | Default                           |
+----------------------+-----------------------------------+-----------------------------------+
| DOCKER\_ROOT\_IMAGE  | name and tag of the CI image used | Required                          |
| GEOS\_TPL\_DIR       | tpl install dir from CI image     | Set from CI image (see below)     |
| GEOS\_CASE\_DIR      | path to copy generic input files  | '/opt/GEOS/cases'                 |
| GEOS\_SRC\_DIR       | path to geos sources              | '/tmp/src'                        |
| GEOS\_BUILD\_DIR     | path to build dir                 | ${GEOS\_SRC\_DIR}/build           |
| GEOS\_INSTALL\_DIR   | path to install geos binaries     | '/opt/GEOS/GEOS-version'          |
| EXE                  | path to geos executable           | ${GEOS\_INSTALL\_DIR}/bin/geosx   | 
+----------------------+-----------------------------------+-----------------------------------+

*Only* 'DOCKER\_ROOT\_IMAGE' is intended to be set. All other variables should preferably be lest to their default values.

If 'GEOS\_SRC\_DIR' is set to some place else, it will not be removed from image during compilation and can be used for in-docker dev.

The first step is then to chose a base CI image from geos' [Dockerhub](https://hub.docker.com/u/geosx). Then one should set 'GEOS\_TPL\_DIR' from this base image. 
As written bellow, this will pull the base CI image and set the appropriate env variable. Eventually, one have to compile the docker image.

```bash
export DOCKER_ROOT_IMAGE=geosx/ubuntu22.04-gcc12
export GEOS_TPL_DIR=$(docker run --rm ${DOCKER_ROOT_IMAGE} /bin/bash -c 'echo ${GEOS_TPL_DIR}')
docker buildx  build --build-arg DOCKER_ROOT_IMAGE=${DOCKER_ROOT_IMAGE} --build-arg GEOS_TPL_DIR=${GEOS_TPL_DIR} -t spe11-geos-docker -f Dockerfile.u22-g12-omp41 .
```

Once build, the command 'docker images' should include the newly created image in the list such as,

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
docker run -v ${LOCAL_CASE_PATH}/cases/spe11b:/opt/GEOS/cases --rm jafranc/geos-35a789a

```

Note that '/opt/GEOS/cases' is default for '${GEOS\_CASE\_DIR}'. If specified otherwise in the build stage, it should be adjust.
The third instuction line link the case of interest to the generic name 'case.xml' so that the docker image can be instantiated and ran as it is.
Note that it can be ran interatively and the call args can be modified.

```bash

docker run -it --entrypoint /bin/bash -v ${LOCAL_CASE_PATH}/cases/spe11b:/opt/GEOS/cases --rm jafranc/geos-35a789a

```

Eventually the results files will be located locally in '${LOCAL\_CASE\_PATH}/vtkOutput' and could be processed as proposed in [GEOS documentation](https://geosx-geosx.readthedocs-hosted.com/en/latest/)



