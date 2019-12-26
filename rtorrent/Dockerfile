###########################################################################
################################ user args ################################
###########################################################################

# docker hub tags
# alpine must be at least version 3.3 (since it has support for "--no-cache")
ARG ALPINE_TAG="3.11.2"

# git tags
ARG LIBXML2_TAG="v2.9.10"
ARG LIBTORRENT_TAG="v0.13.8"
ARG RTORRENT_TAG="v0.9.8"

#############################################################################
################################ build stage ################################
#############################################################################

FROM alpine:${ALPINE_TAG} AS builder

# move some args into scope for the builder
ARG LIBXML2_TAG
ARG LIBTORRENT_TAG
ARG RTORRENT_TAG

# install all build dependencies
RUN apk add --no-cache \
        autoconf       \
        automake       \
        build-base     \
        curl-dev       \
        git            \
        libtool        \
        linux-headers  \
        ncurses-dev    \
        openssl-dev    \
        pkgconfig      \
        subversion

############### clone, build & install libxml2 ################

WORKDIR /tmp/libxml2_src

RUN \
    git clone https://gitlab.gnome.org/GNOME/libxml2.git --branch ${LIBXML2_TAG} .; \
    ./autogen.sh;                                                                   \
    ./configure;                                                                    \
    make -j$(nproc);                                                                \
    make install

################ clone, build & install xmlrpc-c ################

WORKDIR /tmp/xmlrpc-c_src

RUN \
    svn checkout https://svn.code.sf.net/p/xmlrpc-c/code/stable/ .; \
    ./configure                                                     \
        --disable-abyss-server                                      \
        --disable-cgi-server                                        \
        --disable-libwww-client                                     \
        --disable-wininet-client                                    \
        --enable-libxml2-backend;                                   \
    make -j$(nproc);                                                \
    make install

################ clone, build & install libTorrent ################

WORKDIR /tmp/libtorrent_src

RUN git clone https://github.com/rakshasa/libtorrent.git --branch ${LIBTORRENT_TAG} .; \
    ./autogen.sh;                                                                      \
    ./configure                                                                        \
        --disable-debug;                                                               \
    make -j$(nproc);                                                                   \
    make install

################ clone, build & install rTorrent ################

WORKDIR /tmp/rtorrent_src

RUN git clone https://github.com/rakshasa/rtorrent.git --branch ${RTORRENT_TAG} .; \
    ./autogen.sh;                                                                  \
    ./configure                                                                    \
        --disable-debug                                                            \
        --with-xmlrpc-c;                                                           \
    make -j$(nproc);                                                               \
    make install

################################################################################
################################ run-time stage ################################
################################################################################

FROM alpine:${ALPINE_TAG}

# install all run-time dependencies
RUN apk add --no-cache \
        curl           \
        libstdc++      \
        ncurses-dev

# copy built libs & binaries from the builder stage (only copy the ones rTorrent needs)
COPY --from=builder /usr/local/lib/libxml2.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_server.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_util.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libtorrent.so* /usr/local/lib/
COPY --from=builder /usr/local/bin/rtorrent /usr/local/bin/rtorrent

# create some volumes
VOLUME [ "/rtorrent", "/rtorrent/.session", "/rtorrent/watch", "/rtorrent/files" ]

# start rtorrent
ENTRYPOINT [ "rtorrent", "-n", "-o", "import=/rtorrent/rtorrent.rc" ]
