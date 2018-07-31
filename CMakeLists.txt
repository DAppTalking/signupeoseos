cmake_minimum_required(VERSION 3.9)
project(signupeoseos CXX)

set(EOSIO_INSTALL_DIR /usr/local/eosio)
set(BOOST_INCLUDE_DIR /usr/local/include)
set(EOSIOCPP ${EOSIO_INSTALL_DIR}/bin/eosiocpp)
set(CLEOS ${EOSIO_INSTALL_DIR}/bin/cleos)
set(WASM_INSTALL_DIR /usr/local/wasm)

# config cmake cxx
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -emit-llvm")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++14")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --target=wasm32")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdinc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlib")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlibinc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffreestanding")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-threadsafe-statics")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
set(CMAKE_CXX_COMPILER "${WASM_INSTALL_DIR}/bin/clang++")
set(CMAKE_CXX_CREATE_SHARED_LIBRARY "${WASM_INSTALL_DIR}/bin/llvm-link -only-needed -o <TARGET> <OBJECTS> <LINK_LIBRARIES>")

include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${EOSIO_INSTALL_DIR}/include
        ${EOSIO_INSTALL_DIR}/include/libc++/upstream/include
        ${EOSIO_INSTALL_DIR}/include/musl/upstream/include
        ${BOOST_INCLUDE_DIR}
)

file(GLOB_RECURSE RESOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR} *.c *.h *.cpp *.hpp)
add_library(link SHARED ${RESOURCE_FILES})

set(CONTRACT_WAST_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.wast)
set(CONTRACT_ABI_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.abi)
set(CONTRACT_CPP_FILE ${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_PROJECT_NAME}.cpp)

add_custom_target(abi ALL)
add_custom_command(
        TARGET abi
        COMMAND ${EOSIOCPP} -g ${CONTRACT_ABI_FILE} ${CONTRACT_CPP_FILE}
)

add_custom_target(wast ALL)
add_custom_command(
        TARGET wast
        COMMAND ${EOSIOCPP} -o ${CONTRACT_WAST_FILE} ${CONTRACT_CPP_FILE}
)

add_custom_target(deploy)
add_dependencies(deploy abi wast)
add_custom_command(
        TARGET deploy
        COMMAND ${CLEOS} set contract ${CMAKE_PROJECT_NAME} ${CONTRACT_WAST_FILE} ${CMAKE_PROJECT_NAME}.wast ${CMAKE_PROJECT_NAME}.abi -p signupeoseos
)

