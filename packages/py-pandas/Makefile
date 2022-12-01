include ../build/Makefile-vars

# https://github.com/pandas-dev/pandas/releases
VERSION=1.5.1

URL = https://github.com/pandas-dev/pandas/releases/download/v${VERSION}/pandas-${VERSION}.tar.gz

TARBALL = ${UPSTREAM}/pandas-${VERSION}.tar.gz

all: wasm

include ../build/Makefile-rules

${BUILD_WASM}/.patched: ${BUILD_WASM}/.build
	cd ${BUILD_WASM} \
		&& cat ${SRC}/patches/00-no-ctypes.patch | patch -p1
	touch ${BUILD_WASM}/.patched


PYTHONPATH = $(shell cowasm-package-path @cowasm/py-cython):$(shell cowasm-package-path @cowasm/py-numpy):$(shell cowasm-package-path @cowasm/py-pip)

# Attempting to use fully wasm and new pip, etc.   It's all broken though.

# TODO: this is a horrible hack right now, but it **works somewhat.**
#  - first do the install with setup.py install
#  - then install deps by doing "pip install pandas", which installs the
#    deps because pandas is already installed.
# This would break if pypi pandas is updated, but we haven't updated our
# package here yet, etc.  Not ideal.  This is just a first step.
# ${BUILD_WASM}/.install: node_modules ${BUILD_WASM}/.patched
# 	# TODO: Now install pandas first via setup.py, then using pip to install
# 	# just the remaining dependencies.
# 	cd ${BUILD_WASM} \
# 		&& PYTHONPATH="${PYTHONPATH}" npx @cowasm/cpython setup.py build || true \
# 		&& PYTHONPATH="${PYTHONPATH}" npx @cowasm/cpython -m pip install --target build python-dateutil pytz
# 	touch ${BUILD_WASM}/.install


# Horrible old approach that works.

${BUILD_WASM}/.install: node_modules ${BUILD_WASM}/.patched
	mkdir -p ${DIST_WASM}
	# TODO: ugh
	cd ../cpython && make pip
	# TODO: This is a hack right now so that pip install pandas doesn't try to build numpy
	cd ../py-numpy && make && cd build/wasm && PYTHONPATH="${PYTHONPATH}" python-wasm setup.py install
	# TODO: Now install pandas first via setup.py, then using pip to install
	# just the remaining dependencies.
	cd ${BUILD_WASM} \
		&& PYTHONPATH="${PYTHONPATH}" python-wasm setup.py install \
		&& python-wasm -m pip install pandas || true \
		&& python-wasm -m pip install pandas
	touch ${BUILD_WASM}/.install


${DIST_WASM}/.built: node_modules ${BUILD_WASM}/.install
	rm -rf ${DIST_WASM} && mkdir -p ${DIST_WASM}
	cd ${BUILD_WASM} \
		&& npx @cowasm/cpython -m cowasm_bundler pandas \
		&& cd build \
		&& npx @cowasm/cpython -m cowasm_bundler pytz pytz/zoneinfo \
		&& npx @cowasm/cpython -m cowasm_bundler dateutil dateutil/zoneinfo \
		&& mkdir -p six && cp six.py six/__init__.py && npx @cowasm/cpython -m cowasm_bundler six

	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/pandas.tar.xz
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/pytz.tar.xz
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/dateutil.tar.xz
	cd ${DIST_WASM} && tar xf ${BUILD_WASM}/build/six.tar.xz
	touch ${DIST_WASM}/.built

# Just a trivial test for now.
test: ${DIST_WASM}/.built
	rm -rf ${BUILD_WASM}/cowasm-test
	mkdir -p ${BUILD_WASM}/cowasm-test
	@cd ${BUILD_WASM}/cowasm-test \
		&& PYTHONPATH="${PYTHONPATH}:${DIST_WASM}" python-wasm -c "import pandas" \
		&& echo "Trivial PANDAS TEST PASSED!"

