FROM python:2.7.11
MAINTAINER Katharine Berry <katharine@pebble.com>

ENV NPM_CONFIG_LOGLEVEL=info NODE_VERSION=9.11.1 DJANGO_VERSION=1.6

# Node stuff.

# Disable IPv6 in gnupg - can cause key retrieval to fail
RUN mkdir ~/.gnupg && \
  echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf && \
  chmod 700 ~/.gnupg && \
  chmod 600 ~/.gnupg/dirmngr.conf

# gpg keys listed at https://github.com/nodejs/node
RUN set -ex \
  && for key in \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    56730D5401028683275BD23C23EFEFE93C4CFFFE \
    77984A986EBC2AA786BC0F66B01FBB92821C587A \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --verify SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.gz\$" SHASUMS256.txt.asc | sha256sum -c - \
  && tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.gz" SHASUMS256.txt.asc

RUN npm install -g npm jshint

# Django stuff

# Fix broken sources.list (jessie repositories have been archived)
RUN printf "deb http://archive.debian.org/debian/ jessie main\ndeb-src http://archive.debian.org/debian/ jessie main\ndeb http://security.debian.org jessie/updates main\ndeb-src http://security.debian.org jessie/updates main" > /etc/apt/sources.list

RUN apt-get update && apt-get install -y \
    gettext \
    postgresql-client libpq-dev \
  --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN pip install psycopg2 django=="$DJANGO_VERSION"

EXPOSE 8000

# CloudPebble stuff
RUN npm install -g bower && echo '{"allow_root": true}' > ~/.bowerrc

# Grab the toolchain
RUN curl -o /tmp/arm-cs-tools.tar https://cloudpebble-vagrant.s3.amazonaws.com/arm-cs-tools-stripped.tar && \
  tar -xf /tmp/arm-cs-tools.tar -C / && rm /tmp/arm-cs-tools.tar

ADD requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

ENV SDK_TWO_VERSION=3.0

# Install SDK 2
RUN mkdir /sdk2 && \
  curl -L "https://github.com/aveao/PebbleArchive/raw/master/SDKCores/sdk-core-$SDK_TWO_VERSION.tar.bz2"  | \
  tar --strip-components=1 -xj -C /sdk2

ENV SDK_THREE_CHANNEL=beta
ENV SDK_THREE_VERSION=4.3

# Install SDK 3
RUN mkdir /sdk3 && \
  curl -L "https://github.com/aveao/PebbleArchive/raw/master/SDKCores/sdk-core-$SDK_THREE_VERSION.tar.bz2" | \
  tar --strip-components=1 -xj -C /sdk3

COPY . /code
WORKDIR /code

# Bower is awful.
RUN rm -rf bower_components && cd /tmp && python /code/manage.py bower install && mv bower_components /code/

RUN python manage.py compilemessages

RUN make -C /code/c-preload

CMD ["sh", "docker_start.sh"]
