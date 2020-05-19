cmake_minimum_required (VERSION 3.5)
project(test-hfp)

# add libusb
find_path(LIBUSB_INCLUDE_DIR NAMES libusb.h PATH_SUFFIXES "include" "libusb" "libusb-1.0")
find_library(LIBUSB_LIBRARY NAMES usb-1.0 PATH_SUFFIXES "lib" "lib32" "lib64" "dylib")
get_filename_component(LIBUSB_LIBRARY_PATH ${LIBUSB_LIBRARY} DIRECTORY)
include_directories( ${LIBUSB_INCLUDE_DIR} )
link_directories( ${LIBUSB_LIBRARY_PATH} )
link_libraries( usb-1.0 )

# add CppUTest
include_directories("/usr/local/include")
link_directories("/usr/local/lib")
link_libraries( CppUTest )
link_libraries( CppUTestExt )

include_directories(.)
include_directories(..)
include_directories(../../3rd-party/micro-ecc)
include_directories(../../3rd-party/bluedroid/decoder/include)
include_directories(../../3rd-party/bluedroid/encoder/include)
include_directories(../../3rd-party/md5)
include_directories(../../3rd-party/hxcmod-player)
include_directories(../../3rd-party/hxcmod-player/mod)
include_directories(../../3rd-party/rijndael)
include_directories(../../3rd-party/yxml)
include_directories(../../3rd-party/tinydir)
include_directories(../../src)
include_directories(../../chipset/zephyr)
include_directories(../../platform/posix)
include_directories(../../platform/embedded)
include_directories(../../port/libusb)

file(GLOB SOURCES_SRC       "../../src/*.c" "../../example/sco_demo_util.c")
file(GLOB SOURCES_BLE       "../../src/ble/*.c")
file(GLOB SOURCES_GATT      "../../src/ble/gatt-service/*.c")
file(GLOB SOURCES_CLASSIC   "../../src/classic/*.c")
file(GLOB SOURCES_BLUEDROID "../../3rd-party/bluedroid/encoder/srce/*.c" "../../3rd-party/bluedroid/decoder/srce/*.c")
file(GLOB SOURCES_MD5       "../../3rd-party/md5/md5.c")
file(GLOB SOURCES_UECC      "../../3rd-party/micro-ecc/uECC.c")
file(GLOB SOURCES_YXML      "../../3rd-party/yxml/yxml.c")
file(GLOB SOURCES_HXCMOD    "../../3rd-party/hxcmod-player/*.c"  "../../3rd-party/hxcmod-player/mods/*.c")
file(GLOB SOURCES_RIJNDAEL  "../../3rd-party/rijndael/rijndael.c")
file(GLOB SOURCES_POSIX     "../../platform/posix/*.c")
file(GLOB SOURCES_LIBUSB    "../../port/libusb/*.c" "../../platform/libusb/*.c")

file(GLOB SOURCES_BLE_OFF "../../src/ble/le_device_db_memory.c")
list(REMOVE_ITEM SOURCES_BLE   ${SOURCES_BLE_OFF})

file(GLOB SOURCES_POSIX_OFF "../../platform/posix/le_device_db_fs.c")
list(REMOVE_ITEM SOURCES_POSIX ${SOURCES_POSIX_OFF})

set(SOURCES 
	${SOURCES_MD5}
	${SOURCES_YXML}
	${SOURCES_BLUEDROID}
	${SOURCES_POSIX}
	${SOURCES_RIJNDAEL}
	${SOURCES_LIBUSB}
	${SOURCES_SRC}
	${SOURCES_BLE}
	${SOURCES_GATT}
	${SOURCES_CLASSIC}
	${SOURCES_UECC}
	${SOURCES_HXCMOD}
	${SOURCES_ZEPHYR}
)
list(SORT SOURCES)

set_source_files_properties(../../src/classic/hfp_ag.c PROPERTIES LANGUAGE CXX )

list(APPEND SOURCES mock.c test_sequences.c)

# create static lib
add_library(btstack STATIC ${SOURCES})

# create targets for all examples
file(GLOB EXAMPLES_C    "*.c")
list(SORT EXAMPLES_C)

# create targets
foreach(EXAMPLE_FILE ${EXAMPLES_C})
	get_filename_component(EXAMPLE ${EXAMPLE_FILE} NAME_WE)
	# Use C++ Compiler
	set_source_files_properties(${EXAMPLE_FILE} PROPERTIES LANGUAGE CXX )
	set (SOURCE_FILES ${EXAMPLE_FILE})
	message("test/hfp tool: ${EXAMPLE}")
	add_executable(${EXAMPLE} ${SOURCE_FILES} )
	target_link_libraries(${EXAMPLE} btstack)
endforeach(EXAMPLE_FILE)