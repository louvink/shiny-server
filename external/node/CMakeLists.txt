project(node)

set(NODEJS_VERSION 0.10.17)
# See e.g. http://nodejs.org/dist/v0.10.15/SHASUMS.txt to find the appropriate
# hash value for the node-v0.10.15.tar.gz file.
set(NODEJS_SHA1 c9d31d5415d2cf7a09fd7abebf9f01259e9dd93b)

include(ExternalProject)

# Allow overriding of Python command used to build node.js (pass
# -DPYTHON=<path> to cmake).
if(NOT PYTHON)
  if(EXISTS /usr/bin/python26)
    # RedHat/CentOS 5 requires python26, because python means Python 2.4
    set(PYTHON python26)
  else()
    set(PYTHON python)
  endif()
endif()

get_filename_component(NODE_PREFIX 
                       "${CMAKE_CURRENT_SOURCE_DIR}/../../ext/node"
                       REALPATH)

# Download and build Node.js.
# We copy bin/node to bin/shiny-server in order to make the process table
# show "shiny-server" for our process.
ExternalProject_Add(
  node
  URL http://nodejs.org/dist/v${NODEJS_VERSION}/node-v${NODEJS_VERSION}.tar.gz
  URL_HASH SHA1=${NODEJS_SHA1}
  DOWNLOAD_DIR download
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/node-v${NODEJS_VERSION}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ${PYTHON} ${CMAKE_CURRENT_SOURCE_DIR}/node-v${NODEJS_VERSION}/configure
                    "--prefix=${NODE_PREFIX}"
  BUILD_COMMAND make -j4 PYTHON=${PYTHON}
  INSTALL_COMMAND make install &&
                  cp "${NODE_PREFIX}/bin/node" "${NODE_PREFIX}/bin/shiny-server" &&
                  rm "${NODE_PREFIX}/bin/npm" &&
                  cd "${NODE_PREFIX}/lib/node_modules/npm" &&
                  ./scripts/relocate.sh
)
