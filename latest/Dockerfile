FROM dceoy/r-tidyverse:latest

ADD https://s3.amazonaws.com/rstudio-server/current.ver /tmp/ver

ENV CRAN_URL https://cloud.r-project.org/

RUN set -e \
      && apt-get -y update \
      && apt-get -y dist-upgrade \
      && apt-get -y install --no-install-recommends --no-install-suggests \
        gdebi-core libapparmor1 lsb-release psmisc r-cran-* sudo \
      && apt-get -y autoremove \
      && apt-get clean \
      && rm -rf /var/lib/apt/lists/*

RUN set -e \
      && ln -s /dev/stdout /var/log/syslog \
      && curl -S -o /tmp/rstudio.deb \
        http://download2.rstudio.org/rstudio-server-$(cat /tmp/ver)-amd64.deb \
      && gdebi -n /tmp/rstudio.deb \
      && rm -rf /tmp/rstudio.deb /tmp/ver

RUN set -e \
      && clir update \
      && useradd -m -d /home/rstudio -g rstudio-server rstudio \
      && echo rstudio:rstudio | chpasswd \
      && echo "r-cran-repos=${CRAN_URL}" >> /etc/rstudio/rsession.conf
      
      
## Add LaTeX, rticles and bookdown support
## uses dummy texlive, see FAQ 8: https://yihui.name/tinytex/faq/
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
    	## for rJava
    	default-jdk \
    	## Nice Google fonts
    	fonts-roboto \
    	## used by some base R plots
    	ghostscript \
    	## used to build rJava and other packages
    	libbz2-dev \
    	libicu-dev \
    	liblzma-dev \
    	## system dependency of hunspell (devtools)
    	libhunspell-dev \
    	## system dependency of hadley/pkgdown
    	libmagick++-dev \
    	## rdf, for redland / linked data
    	librdf0-dev \
    	## for V8-based javascript wrappers
    	libv8-dev \
    	## for jq queries
    	libjq-dev \
    	## R CMD Check wants qpdf to check pdf sizes, or throws a Warning
    	qpdf \
    	## For building PDF manuals
    	texinfo \
    	## for git via ssh key
    	ssh \
   		## just because
    	less \
      software-properties-common \
      vim \
      apt-utils \
      git-all \
      && apt-get clean \
      && rm -rf /var/lib/apt/lists/ \
      && apt-get update && apt-get upgrade -y

# get gdal2.0 and spatial pkgs
RUN add-apt-repository ppa:ubuntugis/ubuntugis-unstable \
    && apt-get update && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
    libproj-dev \
    proj-data \
    proj-bin \
    && apt-get update && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
    libgeos-dev \
    libudunits2-dev \
    && apt-get install libgdal-dev gdal-bin -y \
    && apt-get autoremove \
    && apt-get clean \
    && apt-get update && apt-get upgrade -y


RUN R -e "install.packages(c('devtools'), repos='https://cloud.r-project.org/')" \
    && R -e "devtools::install_version('rgeos', version = '0.3-28')"

RUN /usr/lib/R/site-library/littler/examples/install2.r --error --deps TRUE \
    sp rgdal sf raster shinyBS shinyjs Metrics fmsb



      

EXPOSE 8787

ENTRYPOINT ["/usr/lib/rstudio-server/bin/rserver"]
CMD ["--server-daemonize=0", "--server-app-armor-enabled=0"]
