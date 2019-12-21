###########################################################################
################################ user args ################################
###########################################################################

# docker hub tags
ARG ALPINE_TAG="3.11"

# git tags
ARG LIBTORRENT_TAG="v0.13.8"
ARG RTORRENT_TAG="v0.9.8"

#############################################################################
################################ build stage ################################
#############################################################################

# alpine must be at least version 3.3 (since it has support for "--no-cache")
FROM alpine:${ALPINE_TAG} AS builder

# move some args into scope for the builder
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

################ clone, build & install xmlrpc-c ################

WORKDIR /tmp/xmlrpc-c_src

RUN \
    svn checkout https://svn.code.sf.net/p/xmlrpc-c/code/super_stable/ .; \
    ./configure                                                           \
        --disable-abyss-server                                            \
        --disable-cgi-server                                              \
        --disable-libwww-client                                           \
        --disable-wininet-client                                          \
        --disable-libxml2-backend;                                        \
    make -j$(nproc);                                                      \
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
COPY --from=builder /usr/local/bin/rtorrent /usr/local/bin/rtorrent
COPY --from=builder /usr/local/lib/libtorrent.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_server.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_util.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_xmlparse.so.* /usr/local/lib/
COPY --from=builder /usr/local/lib/libxmlrpc_xmltok.so* /usr/local/lib/

# create some volumes
VOLUME [ "/rtorrent/.session", "/rtorrent/watch", "/rtorrent/files" ]

# start rtorrent
ENTRYPOINT [ "rtorrent", "-n", "-o", "import=/rtorrent/rtorrent.rc" ]
