include ../build/Makefile-vars

all: wasm

# See https://github.com/pypa/pip/tags
#
VERSION = 22.3.1

URL = https://github.com/pypa/pip/archive/refs/tags/${VERSION}.tar.gz
TARBALL = ${UPSTREAM}/pip-${VERSION}.tar.gz

include ../build/Makefile-rules

SETUPTOOLS = $(shell cowasm-package-path @cowasm/python-setuptools)

${BUILD_WASM}/.built: node_modules ${BUILD_WASM}/.build
	cd ${BUILD_WASM} \
		&& PYTHONPATH=${SETUPTOOLS} npx @cowasm/cpython setup.py build
	touch ${BUILD_WASM}/.built

${DIST_WASM}/.built: ${BUILD_WASM}/.built
	cd ${BUILD_WASM}/build/lib \
		&& npx @cowasm/cpython -m cowasm_bundler pip
	rm -rf ${DIST_WASM}
	mkdir -p ${DIST_WASM}
	cp -r ${SETUPTOOLS}/* ${DIST_WASM}
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/lib/pip.tar.xz
	touch ${DIST_WASM}/.built

test: ${DIST_WASM}/.built
	# Importing from the bundle works:
	PYTHONPATH=${DIST_WASM} npx @cowasm/cpython -c 'import pip'
	# The command line script works.
	PYTHONPATH=${DIST_WASM} npx @cowasm/cpython -m pip > /dev/null
