# Base Docker Image Creation

First, pull AWS base image and run interactively launching a bash shell:

```bash
docker pull public.ecr.aws/lambda/provided
docker run --entrypoint "/bin/bash" --name rlambda -it public.ecr.aws/lambda/provided
```

Now in the container, run the following:

```bash
R_VERSION=4.1.0

yum -y update
yum -y install wget git openssl-devel tar deltarpm
yum -y update
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
wget https://cdn.rstudio.com/r/centos-7/pkgs/R-${R_VERSION}-1-1.x86_64.rpm
yum -y install R-${R_VERSION}-1-1.x86_64.rpm
rm R-${R_VERSION}-1-1.x86_64.rpm

PATH="${PATH}:/opt/R/${R_VERSION}/bin/"

Rscript -e 'R.home(component="home")'
Rscript -e 'install.packages(c("httr", "jsonlite", "logger", "remotes", "dplyr", "lubridate", "stringr", "purrr"), repos = "https://packagemanager.rstudio.com/all/__linux__/centos7/latest")'
Rscript -e 'remotes::install_github("mdneuzerling/lambdr")'

mkdir /lambda
touch /lambda/example_runtime.R
chmod 755 -R /lambda

printf '#!/bin/sh\ncd /lambda\nRscript runtime.R' > /var/runtime/bootstrap
chmod +x /var/runtime/bootstrap

history > /command-history
```

this performs the following tasks:

- Sets `R_VERSION`
- Updates `yum`
- Installs `wget`, `git`, `tar`, `deltarpm` and `openssl-devel`
- Installs `epel`
- Downloads (via `wget`) the specified `R_VERSION`'s `.rpm` for `CentOS`
- Installs downloaded `.rpm` for R
- Removes the `.rpm` file
- Sets `PATH` to include new R installation
- Installs R Packages: `httr`, `jsonlite`, `logger`, `remotes`, `dplyr`, `lubridate`, `stringr`, and `purrr` from `CRAN` using `RSPM` instead of default `CRAN` repository which has to compile the binaries.
- Installs `lambdr` package from GitHub via `remotes`
- Creates `/lambda` root level directory to house runtime scripts and sets permissions
- Creates `/var/runtime/bootstrap` to run the runtime script with R and sets permissions
- Copies all history commands to `/command-history`

Next, from outside the container:

- Copy the `/command-history` to my host machine for reference
- Commit the container as our new base image prefixed with my `ghcr.io` URL syntax
- Login to `ghcr.io` 
- Push to GitHub

```bash
docker cp rlambda:/command-history ./command-history
docker ps
docker commit --author "Jimmy Briggs" rlambda ghcr.io/jimbrig/rlambda/rlambda:v1
docker login ghcr.io --username jimbrig --password $GITHUB_TOKEN
docker push ghcr.io/jimbrig/rlambda/rlambda:v1
```

Resulting in the new `rlambda:v1` or `rlambda:latest` base docker image to be used for future AWS Lambda R projects!

To use simply add `FROM ghcr.io/jimbrig/rlambda/rlambda:latest` to the top of your `Dockerfile`.