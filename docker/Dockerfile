FROM ubuntu:16.04
MAINTAINER annawoodard@uchicago.edu
# adapted from https://github.com/trinityrnaseq/trinityrnaseq/blob/master/Docker/Dockerfile

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y gcc g++ perl python automake make \
       wget git curl libdb-dev \
       zlib1g-dev bzip2 libncurses5-dev \
       texlive-latex-base \
       default-jre \
       python-pip python-dev \
       vim \
       bc \
       gfortran \
       build-essential libghc-zlib-dev libncurses-dev libbz2-dev liblzma-dev libpcre3-dev libxml2-dev \
       libblas-dev gfortran unzip ftp libzmq3-dev ftp fort77 libreadline-dev \
       libcurl4-openssl-dev libx11-dev libxt-dev \
       x11-common libcairo2-dev libpng-dev libreadline-dev libjpeg-dev pkg-config libtbb-dev \
       cmake rsync libssl-dev tzdata \
       bedtools \
       locales-all \
       gawk \
       && apt-get clean



## perl
RUN curl -L https://cpanmin.us | perl - App::cpanminus && \
    cpanm install DB_File && \
    cpanm install DB_File && \
    cpanm install URI::Escape && \
    cpanm install PerlIO::gzip


## set up tool config and deployment area
ENV SRC /usr/local/src
ENV BIN /usr/local/bin


## arriba
WORKDIR $SRC
RUN wget -q -O - "https://github.com/suhrig/arriba/releases/download/v1.2.0/arriba_v1.2.0.tar.gz" | tar -xzf -


## bowtie
WORKDIR $SRC
RUN wget https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.2.1.1/bowtie-1.2.1.1-linux-x86_64.zip/download -O bowtie-1.2.1.1-linux-x86_64.zip && \
        unzip bowtie-1.2.1.1-linux-x86_64.zip && \
	mv bowtie-1.2.1.1/bowtie* $BIN


# blast
WORKDIR $SRC
RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.5.0/ncbi-blast-2.5.0+-x64-linux.tar.gz && \
    tar xvf ncbi-blast-2.5.0+-x64-linux.tar.gz && \
    cp ncbi-blast-2.5.0+/bin/* $BIN && \
    rm -r ncbi-blast-2.5.0+


## bowtie2
WORKDIR $SRC
RUN wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.3.4.1/bowtie2-2.3.4.1-linux-x86_64.zip/download -O bowtie2-2.3.4.1-linux-x86_64.zip && \
    unzip bowtie2-2.3.4.1-linux-x86_64.zip && \
    mv bowtie2-2.3.4.1-linux-x86_64/bowtie2* $BIN && \
    rm *.zip && \
    rm -r bowtie2-2.3.4.1-linux-x86_64


## ChimPipe
## ChimPipe expects executables to be in bin dir relative to source,
## so only copy gemtools to bin
WORKDIR $SRC
RUN wget https://github.com/Chimera-tools/ChimPipe/releases/download/v0.9.6/ChimPipe-0.9.6.tar.gz && \
    tar xzf ChimPipe* && \
    cp $SRC/ChimPipe*/bin/gemtools*/* $BIN && \
    rm $SRC/*tar.gz

## needed by gemtools
ENV LC_ALL=en_US.utf8
RUN dpkg-reconfigure locales-all

## samtools
RUN wget https://github.com/samtools/samtools/releases/download/1.7/samtools-1.7.tar.bz2 && \
    tar xvf samtools-1.7.tar.bz2 && \
    cd samtools-1.7/ && \
    ./configure && make && make install


# ## jellyfish
# RUN wget https://github.com/gmarcais/Jellyfish/releases/download/v2.3.0/jellyfish-2.3.0.tar.gz && \
#     tar xvf jellyfish-2.3.0.tar.gz && \
#     cd jellyfish-2.3.0/ && \
#     ./configure && make && make install


## RSEM
# RUN mkdir /usr/local/lib/site_perl
# WORKDIR $SRC
# RUN wget https://github.com/deweylab/RSEM/archive/v1.3.3.tar.gz && \
# 	 tar xvf v1.3.3.tar.gz && \
#      cd RSEM-1.3.3 && \
#      make && \
#      cp rsem-* $BIN && \
#      cp convert-sam-for-rsem $BIN && \
#      cp rsem_perl_utils.pm /usr/local/lib/site_perl/ && \
#      cd ../ && rm -r RSEM-1.3.3


## salmon
WORKDIR $SRC
ENV SALMON_VERSION=1.0.0
RUN wget https://github.com/COMBINE-lab/salmon/releases/download/v${SALMON_VERSION}/Salmon-${SALMON_VERSION}_linux_x86_64.tar.gz && \
    tar xvf Salmon-${SALMON_VERSION}_linux_x86_64.tar.gz && \
    ln -s $SRC/Salmon-latest_linux_x86_64/bin/salmon $BIN/.

WORKDIR $SRC
RUN ln -sf /usr/local/src/salmon-latest_linux_x86_64/bin/salmon $BIN/.


## STAR
ENV STAR_VERSION=2.7.3a
RUN STAR_URL="https://github.com/alexdobin/STAR/archive/${STAR_VERSION}.tar.gz" &&\
    wget -P $SRC $STAR_URL && \
    tar -xvf $SRC/${STAR_VERSION}.tar.gz -C $SRC && \
    mv $SRC/STAR-${STAR_VERSION}/bin/Linux_x86_64_static/STAR /usr/local/bin


## blat
RUN wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/blat/blat -P $BIN && \
    chmod a+x $BIN/blat


## Trinity
ENV TRINITY_VERSION="2.9.1"
ENV TRINITY_CO="e806a0f341e4d3917c2b181c4ecd4a4c3f4367fc"

WORKDIR $SRC
RUN git clone --recursive https://github.com/trinityrnaseq/trinityrnaseq.git && \
    cd trinityrnaseq && \
    git checkout ${TRINITY_CO} && \
    git submodule init && git submodule update && \
    rm -rf ./trinity_ext_sample_data && \
    make && make plugins && \
    make install && \
    cd ../ && rm -r trinityrnaseq

ENV TRINITY_HOME /usr/local/bin/trinityrnaseq
ENV PATH=${TRINITY_HOME}:${PATH}

## GMAP  (build / install using --prefix, or else this one will clobber important header files)
ENV GMAP_VERSION=2017-11-15
WORKDIR $SRC
RUN GMAP_URL="http://research-pub.gene.com/gmap/src/gmap-gsnap-$GMAP_VERSION.tar.gz" && \
    wget $GMAP_URL && \
    tar xvf gmap-gsnap-$GMAP_VERSION.tar.gz && \
    cd gmap-$GMAP_VERSION && ./configure --prefix=`pwd` && make && make install && cp ./bin/* $BIN/


## STAR-Fusion
WORKDIR $SRC

ENV STAR_FUSION_VERSION=1.9.0
ENV STARF_CHECKOUT=36ed7468942d84baa44e1b973bf354d3d653d344
ENV STAR_FUSION_HOME $SRC/STAR-Fusion

RUN git clone https://github.com/STAR-Fusion/STAR-Fusion.git && \
     cd STAR-Fusion && \
     git checkout $STARF_CHECKOUT && \
     git submodule init && git submodule update && \
     cd FusionInspector && \
     git submodule init && git submodule update

## Kallisto
WORKDIR $SRC
RUN git clone https://github.com/pachterlab/kallisto.git && \
    cd kallisto && \
    git checkout ae81a86 && \
    mkdir build && cd build && \
    cmake .. && make && make install

## Pizzly
WORKDIR $SRC
RUN apt update && apt install -y libz-dev python3 python3-pip; \
	# ln -s /usr/bin/python3 /usr/local/bin/python; \
	pip3 install h5py numpy; \
	git clone https://github.com/pmelsted/pizzly.git; \
	cd pizzly; \
	git checkout 8617a24; \
	mkdir build && cd build && cmake .. && make && make install; \
	apt remove -y git  && apt autoclean -y  && apt autoremove -y


# ## MapSplice
# ENV MAPSPLICE_VERSION  2.2.1
# WORKDIR $SRC
# RUN wget http://protocols.netlab.uky.edu/~zeng/MapSplice-v$MAPSPLICE_VERSION.zip \
# 	 && unzip MapSplice-v$MAPSPLICE_VERSION.zip \
# 	 && cd MapSplice-v$MAPSPLICE_VERSION \
# 	 && make

# COPY run.sh /usr/local/bin
# RUN chmod a+x /usr/local/bin/run.sh

# cleanup
WORKDIR $SRC
RUN rm -r *.tar.gz
RUN apt-get clean
