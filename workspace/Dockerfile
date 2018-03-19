FROM phusion/baseimage:latest

MAINTAINER HIKARU KOBORI

RUN apt-get update && \
    apt-get install -y --allow-downgrades --allow-remove-essential \
        --allow-change-held-packages --fix-missing \
        pkg-config \
        libcurl4-openssl-dev \
        libedit-dev \
        libffi-dev \
        libssl-dev \
        libxml2-dev \
        libmysqld-dev \
        xz-utils \
        libsqlite3-dev \
        sqlite3 \
        git \
        curl \
        vim \
        nano \
    && apt-get clean


# Source the bash
RUN . ~/.bashrc


#####################################
# Non-Root User:
#####################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ARG PGID=1000

ENV PUID ${PUID}
ENV PGID ${PGID}

RUN groupadd -g ${PGID} raildock && \
    useradd -u ${PUID} -g raildock -m raildock

#####################################
# Set Timezone
#####################################

ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#####################################
# Crontab
#####################################
USER root

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

#####################################
# User Aliases
#####################################

USER raildock
COPY ./aliases.sh /home/raildock/aliases.sh
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/raildock/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
	sed -i 's/\r//' /home/raildock/aliases.sh && \
	sed -i 's/^#! \/bin\/sh/#! \/bin\/bash/' /home/raildock/aliases.sh

USER root
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/raildock/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
	sed -i 's/\r//' /home/raildock/aliases.sh && \
	sed -i 's/^#! \/bin\/sh/#! \/bin\/bash/' /home/raildock/aliases.sh


#####################################
# Ruby / rbenv:
#####################################
USER raildock

ARG RUBY_VERSION=2.4.3
ENV RUBY_VERSION ${RUBY_VERSION}
ENV RBENV_DIR /home/raildock/.rbenv
# Install rbenv (A Ruby version Manager)
RUN git clone https://github.com/rbenv/rbenv.git $RBENV_DIR && \
    cd $RBENV_DIR && src/configure && make -C src && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc && \
    echo 'eval "$(rbenv init -)"' >> ~/.bashrc && \
    git clone https://github.com/rbenv/ruby-build.git $RBENV_DIR/plugins/ruby-build && \
    $RBENV_DIR/bin/rbenv install ${RUBY_VERSION} && \
    $RBENV_DIR/bin/rbenv global ${RUBY_VERSION} && \
    $RBENV_DIR/bin/rbenv local ${RUBY_VERSION}

# Add bundler to gem
RUN $RBENV_DIR/shims/gem install bundler

# Add RBENV binaries to root's .bashrc
USER root

RUN echo "" >> ~/.bashrc && \
    echo 'export RBENV_DIR="/home/raildock/.rbenv"' >> ~/.bashrc && \
    echo 'export PATH="$RBENV_DIR/bin:$PATH"' >> ~/.bashrc && \
    echo 'eval "$(rbenv init -)"' >> ~/.bashrc

USER raildock

#####################################
# Node / NVM:
#####################################

# Check if NVM needs to be installed
ARG NODE_VERSION=stable
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ENV INSTALL_NODE ${INSTALL_NODE}
ENV NVM_DIR /home/raildock/.nvm
RUN if [ ${INSTALL_NODE} = true ]; then \
    # Install nvm (A Node Version Manager)
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash && \
        . $NVM_DIR/nvm.sh && \
        nvm install ${NODE_VERSION} && \
        nvm use ${NODE_VERSION} && \
        nvm alias ${NODE_VERSION} && \
        npm install -g gulp bower vue-cli \
;fi

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

# Add NVM binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/raildock/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

# Add PATH for node
ENV PATH $PATH:$NVM_DIR/versions/node/v${NODE_VERSION}/bin


#####################################
# YARN:
#####################################

USER raildock

ARG INSTALL_YARN=false
ENV INSTALL_YARN ${INSTALL_YARN}
ARG YARN_VERSION=latest
ENV YARN_VERSION ${YARN_VERSION}

RUN if [ ${INSTALL_YARN} = true ]; then \
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
    if [ ${YARN_VERSION} = "latest" ]; then \
        curl -o- -L https://yarnpkg.com/install.sh | bash; \
    else \
        curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version ${YARN_VERSION}; \
    fi && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.yarn/bin:$PATH"' >> ~/.bashrc \
;fi

# Add YARN binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_YARN} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export YARN_DIR="/home/raildock/.yarn"' >> ~/.bashrc && \
    echo 'export PATH="$YARN_DIR/bin:$PATH"' >> ~/.bashrc \
;fi

#####################################
# Linuxbrew:
#####################################
USER root

ARG INSTALL_LINUXBREW=false
ENV INSTALL_LINUXBREW ${INSTALL_LINUXBREW}

RUN if [ ${INSTALL_LINUXBREW} = true ]; then \

    # Preparation
    apt-get upgrade -y && \
    apt-get install -y build-essential make cmake scons curl git \
      ruby autoconf automake autoconf-archive \
      gettext libtool flex bison \
      libbz2-dev libcurl4-openssl-dev \
      libexpat-dev libncurses-dev && \

    # Install the Linuxbrew
    git clone --depth=1 https://github.com/Homebrew/linuxbrew.git ~/.linuxbrew && \

    echo "" >> ~/.bashrc && \
    echo 'export PKG_CONFIG_PATH"=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \

    # Setup linuxbrew
    echo 'export LINUXBREWHOME="$HOME/.linuxbrew"' >> ~/.bashrc && \
    echo 'export PATH="$LINUXBREWHOME/bin:$PATH"' >> ~/.bashrc && \
    echo 'export MANPATH="$LINUXBREWHOME/man:$MANPATH"' >> ~/.bashrc && \
    echo 'export PKG_CONFIG_PATH="$LINUXBREWHOME/lib64/pkgconfig:$LINUXBREWHOME/lib/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \
    echo 'export LD_LIBRARY_PATH="$LINUXBREWHOME/lib64:$LINUXBREWHOME/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc \
;fi

#####################################
# PYTHON:
#####################################

ARG INSTALL_PYTHON=false
ENV INSTALL_PYTHON ${INSTALL_PYTHON}
RUN if [ ${INSTALL_PYTHON} = true ]; then \
  apt-get update \
  && apt-get -y install python python-pip python-dev build-essential  \
  && pip install --upgrade pip  \
  && pip install --upgrade virtualenv \
;fi

#####################################
# ImageMagick:
#####################################
USER root
ARG INSTALL_IMAGEMAGICK=false
ENV INSTALL_IMAGEMAGICK ${INSTALL_IMAGEMAGICK}
RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y --force-yes imagemagick php-imagick \
;fi

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set default work directory
WORKDIR /var/www
