include ../build/Makefile-vars

all: wasm

# See https://github.com/pypa/setuptools/releases
VERSION = 65.6.3

URL = https://github.com/pypa/setuptools/archive/refs/tags/v65.6.3.tar.gz
TARBALL = ${UPSTREAM}/setuptools-${VERSION}.tar.gz

include ../build/Makefile-rules

${BUILD_WASM}/.built: node_modules ${BUILD_WASM}/.build
	cd ${BUILD_WASM} \
		&& npx @cowasm/cpython setup.py build
	touch ${BUILD_WASM}/.built

${DIST_WASM}/.built: ${BUILD_WASM}/.built
	cd ${BUILD_WASM}/build/lib \
		&& npx @cowasm/cpython -m cowasm_bundler setuptools \
		&& npx @cowasm/cpython -m cowasm_bundler _distutils_hack \
		&& npx @cowasm/cpython -m cowasm_bundler pkg_resources
	rm -rf ${DIST_WASM}
	mkdir -p ${DIST_WASM}
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/lib/setuptools.tar.xz
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/lib/_distutils_hack.tar.xz
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/lib/pkg_resources.tar.xz
	touch ${DIST_WASM}/.built

test: ${DIST_WASM}/.built
	# Test that importing from the bundle works:
	PYTHONPATH=${DIST_WASM} python-wasm -c 'import setuptools'
