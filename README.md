# `rlambda` - R on AWS Lambda



## Docker

- All containers start using the `public.ecr.aws/lambda/provided` base Docker image, which provides the basic components necessary to serve a Lambda.
  - *Note this base image is not ubuntu/debian based but CentOS - so `yum` not `apt`*
- Install R, R Package System Requirements, R Packages, and the `lambr` package.
- Copy a `runtime.R` into container plus any other necessary files.
- Generate a simple bootstrap which runs `runtime.R` with R
- Set the handler as the `CMD` 

Basic Scaffolding:

```dockerfile
FROM public.ecr.aws/lambda/provided

ENV R_VERSION={{{r_ver}}}

# Pre-Reqs
RUN yum install -y git openssl-devel tar wget deltarpm

# Install R
RUN yum -y update \
  && yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
  && wget https://cdn.rstudio.com/r/centos-7/pkgs/R-${R_VERSION}-1-1.x86_64.rpm \
  && yum -y install R-${R_VERSION}-1-1.x86_64.rpm \
  && rm R-${R_VERSION}-1-1.x86_64.rpm
  
# Add R to PATH
ENV PATH="${PATH}:/opt/R/${R_VERSION}/bin/"

# Install System Requirements for R Packages
{{{sysreqs}}}

# Install R Packages
{{{cran_pkgs}}}

# Install Remotes
{{{gh_pkgs}}}

RUN mkdir /lambda
COPY runtime.R /lambda
RUN chmod 755 -R /lambda

RUN printf '#!/bin/sh\ncd /lambda\nRscript runtime.R' > /var/runtime/bootstrap \
  && chmod +x /var/runtime/bootstrap

CMD ["<function>"]
```

Full example:

```dockerfile
FROM public.ecr.aws/lambda/provided

ENV R_VERSION=4.1.2

RUN yum -y install wget git tar

RUN yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
  && wget https://cdn.rstudio.com/r/centos-7/pkgs/R-${R_VERSION}-1-1.x86_64.rpm \
  && yum -y install R-${R_VERSION}-1-1.x86_64.rpm \
  && rm R-${R_VERSION}-1-1.x86_64.rpm

ENV PATH="${PATH}:/opt/R/${R_VERSION}/bin/"

# System requirements for R packages
RUN yum -y install openssl-devel

RUN Rscript -e "install.packages(c('httr', 'jsonlite', 'logger', 'remotes'), repos = 'https://packagemanager.rstudio.com/all/__linux__/centos7/latest')"
RUN Rscript -e "remotes::install_github('mdneuzerling/lambdr')"

RUN mkdir /lambda
COPY runtime.R /lambda
RUN chmod 755 -R /lambda

RUN printf '#!/bin/sh\ncd /lambda\nRscript runtime.R' > /var/runtime/bootstrap \
  && chmod +x /var/runtime/bootstrap

CMD ["parity"]
```







##  Resources



- The [lambdr](https://github.com/mdneuzerling/lambdr) R package
- 