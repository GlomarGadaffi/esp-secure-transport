# GloSSH example project (ESP-IDF)
#
# Builds the `main` example: glo_mtls crypto self-test + TinySSH server.
# Both the SSH server (tinyssh) and the TLS client (glo_mtls) are built on
# TinySSH's tinynacl crypto.
cmake_minimum_required(VERSION 3.16)

set(COMPONENTS "main" "tinyssh" "glo_mtls" "esptool_py")

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(glossh-example)
