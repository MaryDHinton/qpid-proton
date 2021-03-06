#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

##------------------------------------------------------
## Use Swig to generate a PHP binding to the Proton API
##------------------------------------------------------

# Uses the php-config command line tool from PHP to extract the location of the PHP header
# files

execute_process(COMMAND ${PHP_CONFIG_EXE} --includes
                OUTPUT_VARIABLE PHP_INCLUDES
                RESULT_VARIABLE retval
                ERROR_VARIABLE  errmsg
                OUTPUT_STRIP_TRAILING_WHITESPACE)

set_source_files_properties(${CMAKE_CURRENT_SOURCE_DIR}/php.i PROPERTIES SWIG_FLAGS "-I${PROJECT_SOURCE_DIR}/include")
swig_add_module(cproton php ${CMAKE_CURRENT_SOURCE_DIR}/php.i)
set_target_properties(cproton PROPERTIES PREFIX "")
set_source_files_properties(${swig_generated_file_fullname} PROPERTIES COMPILE_FLAGS "${PHP_INCLUDES}")
swig_link_libraries(cproton ${BINDING_DEPS})

execute_process(COMMAND ${PHP_CONFIG_EXE} --extension-dir
                OUTPUT_VARIABLE PHP_EXT_DIR_DEFAULT
                OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND ${PHP_CONFIG_EXE} --prefix
                OUTPUT_VARIABLE PHP_PFX
                OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND ${PHP_CONFIG_EXE} --config-options
                OUTPUT_VARIABLE PHP_OPTS
                OUTPUT_STRIP_TRAILING_WHITESPACE)

set(GET_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/get_include_dir.php)
execute_process(COMMAND ${PHP_EXE} -n ${GET_INCLUDE_DIR} ${PHP_PFX}
                OUTPUT_VARIABLE PHP_INCLUDE_DIR_DEFAULT
                OUTPUT_STRIP_TRAILING_WHITESPACE)

if ("${PHP_INCLUDE_DIR_DEFAULT}" STREQUAL "")
  set(PHP_INCLUDE_DIR_DEFAULT "/usr/share/php")
endif()

string(REGEX MATCH "--with-config-file-scan-dir=([^ ]*)" PHP_OPT_MATCH ${PHP_OPTS})
set (PHP_INI_DIR_DEFAULT ${CMAKE_MATCH_1})

if ("${PHP_INI_DIR_DEFAULT}" STREQUAL "")
  set(PHP_INI_DIR_DEFAULT "/etc/php.d")
endif()

if (IS_ABSOLUTE "${PHP_INI_DIR_DEFAULT}" AND (NOT (IS_ABSOLUTE ${CMAKE_INSTALL_PREFIX})))
  string(SUBSTRING ${PHP_INI_DIR_DEFAULT} 1 -1 PHP_INI_DIR_DEFAULT)
  string(SUBSTRING ${PHP_EXT_DIR_DEFAULT} 1 -1 PHP_EXT_DIR_DEFAULT)
  string(SUBSTRING ${PHP_INCLUDE_DIR_DEFAULT} 1 -1 PHP_INCLUDE_DIR_DEFAULT)
endif ()

set(PHP_EXT_DIR ${PHP_EXT_DIR_DEFAULT} CACHE PATH "PHP extensions directory.")
set(PHP_INI_DIR ${PHP_INI_DIR_DEFAULT} CACHE PATH "Directory scanned for PHP ini files.")
set(PHP_INCLUDE_DIR ${PHP_INCLUDE_DIR_DEFAULT} CACHE PATH "PHP include directory.")

install(TARGETS cproton
        DESTINATION ${PHP_EXT_DIR}
        COMPONENT PHP)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cproton.php
        DESTINATION ${PHP_INCLUDE_DIR}
        COMPONENT PHP)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/proton.php
        DESTINATION ${PHP_INCLUDE_DIR}
        COMPONENT PHP)

if (NOT ${PHP_INI_DIR} STREQUAL "")
  install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/cproton.ini
          DESTINATION ${PHP_INI_DIR}
          COMPONENT PHP)
endif ()
