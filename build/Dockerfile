FROM emscripten/emsdk:5.0.7 AS builder

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
      autotools-dev automake libtool \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ARG VERSION=1.16
ENV VERSION=${VERSION}
ENV DIRECTORY=potrace-${VERSION}
ENV OUT_DIR=/out

WORKDIR /src

ADD https://potrace.sourceforge.net/download/${VERSION}/${DIRECTORY}.tar.gz potrace.tar.gz
RUN echo "be8248a17dedd6ccbaab2fcc45835bb0502d062e40fbded3bc56028ce5eb7acc  potrace.tar.gz" | sha256sum -c - \
    && tar -xzf potrace.tar.gz && rm potrace.tar.gz

WORKDIR /src/${DIRECTORY}
RUN mkdir -p $OUT_DIR

ENV CFLAGS="-O2"
ENV CXXFLAGS="${CFLAGS}"

RUN emconfigure ./configure \
    --host=wasm32-unknown-linux \
    --prefix="$OUT_DIR" \
    --disable-shared \
    --enable-static \
    --disable-zlib

COPY js /src/js

ENV POTRACE_LDFLAGS="\
    -lnodefs.js -lworkerfs.js \
    --post-js /src/js/post.js \
    -s WASM_BIGINT=1 \
    -s ALLOW_MEMORY_GROWTH=1 \
    -s EXIT_RUNTIME=1 \
    -s FORCE_FILESYSTEM=1 \
    -s EXPORTED_RUNTIME_METHODS='[\"callMain\",\"FS\",\"NODEFS\",\"WORKERFS\",\"ENV\"]' \
    -s INCOMING_MODULE_JS_API='[\"noInitialRun\",\"noFSInit\",\"wasmBinary\",\"locateFile\",\"preRun\",\"postRun\",\"instantiateWasm\",\"print\",\"printErr\",\"arguments\"]' \
    -s INVOKE_RUN=0 \
    -s MODULARIZE=1 \
    -s EXPORT_ES6=1 \
"

RUN emmake make \
    EXEEXT=".mjs" \
    LDFLAGS="${POTRACE_LDFLAGS}" \
    -j$(nproc)

RUN mkdir -p /dist \
    && for t in potrace mkbitmap; do \
         cp src/$t.mjs /dist/$t.mjs \
         && cp src/$t.wasm /dist/$t.wasm \
         && [ "$(head -c4 /dist/$t.wasm | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
         && [ "$(stat -c%s /dist/$t.wasm)" -gt 4096 ] \
         && grep -q 'export default' /dist/$t.mjs \
         || { echo "FATAL: $t artifact failed sanity check"; exit 1; }; \
       done

FROM scratch AS exportor
COPY --from=builder /dist /dist
