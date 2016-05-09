FROM debian:jessie

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
#RUN groupadd -r www-data && useradd -r --create-home -g www-data www-data

ENV HTTPD_PREFIX /usr/local/apache2
ENV PATH $PATH:$HTTPD_PREFIX/bin
RUN mkdir -p "$HTTPD_PREFIX" \
	&& chown www-data:www-data "$HTTPD_PREFIX"
WORKDIR $HTTPD_PREFIX

# install httpd runtime dependencies
# https://httpd.apache.org/docs/2.4/install.html#requirements
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
    wget \
		libapr1 \
		libaprutil1 \
		libapr1-dev \
		libaprutil1-dev \
		libpcre++0 \
		libssl1.0.0 \
	&& rm -r /var/lib/apt/lists/*

ENV HTTPD_VERSION 2.2.31
ENV HTTPD_BZ2_URL https://www.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2

RUN buildDeps=' \
		ca-certificates \
		curl \
		bzip2 \
		gcc \
		libpcre++-dev \
		libssl-dev \
    libperl-dev \
		make \
	' \
	set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends $buildDeps \
	&& rm -r /var/lib/apt/lists/* \
	&& curl -fSL "$HTTPD_BZ2_URL" -o httpd.tar.bz2 \
	&& curl -fSL "$HTTPD_BZ2_URL.asc" -o httpd.tar.bz2.asc \
# see https://httpd.apache.org/download.cgi#verify
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B1B96F45DFBDCCF974019235193F180AB55D9977 \
	&& gpg --batch --verify httpd.tar.bz2.asc httpd.tar.bz2 \
	&& rm -r "$GNUPGHOME" httpd.tar.bz2.asc \
	&& mkdir -p src/httpd \
	&& tar -xvf httpd.tar.bz2 -C src/httpd --strip-components=1 \
	&& rm httpd.tar.bz2


RUN cd src/httpd \
	&& ./configure --prefix=$HTTPD_PREFIX  --enable-so --enable-ssl --enable-ldap --enable-proxy --enable-proxy-http --enable-proxy-html --enable-mods-shared=all \
	&& make -j"$(nproc)" \
	&& make install \
	&& cd ../../ \
	&& rm -r src/httpd \
	&& sed -ri ' \
		s!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; \
		s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g; \
		' /usr/local/apache2/conf/httpd.conf \
	&& apt-get purge -y --auto-remove $buildDeps


RUN ln -s /usr/lib/x86_64-linux-gnu/libgdbm.so.3.0.0 /usr/lib/x86_64-linux-gnu/libgdbm.so

RUN perl -MCPAN -e 'install File::Spec::Functions'
RUN perl -MCPAN -e 'install Net::LDAP'

ENV MOD_PERL_VERSION 2.0.9
ENV MOD_PERL_URL http://mirrors.ircam.fr/pub/apache/perl/mod_perl-${MOD_PERL_VERSION}.tar.gz

RUN curl -fSL "$MOD_PERL_URL" -o mod_perl.tar.gz

RUN mkdir -p src/mod_perl \
  && tar -xvf mod_perl.tar.gz -C src/mod_perl --strip-components=1 \
  && rm mod_perl.tar.gz \
  && cd src/mod_perl \
  && perl Makefile.PL MP_APXS=/usr/local/apache2/bin/apxs \
  && make \
  && make test \
  && make install


COPY httpd-foreground /usr/local/bin/

EXPOSE 80
CMD ["httpd-foreground"]
