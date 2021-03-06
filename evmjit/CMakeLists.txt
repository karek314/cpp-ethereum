cmake_minimum_required(VERSION 3.0.0)

cmake_policy(SET CMP0042 OLD) 	# fix MACOSX_RPATH
cmake_policy(SET CMP0048 NEW) 	# allow VERSION argument in project()

project(EVMJIT VERSION 0.9.0.1 LANGUAGES CXX)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_AUTOMOC OFF)

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC")
else()
	set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Wextra -Wconversion -Wno-sign-conversion -Wno-unknown-pragmas ${CMAKE_CXX_FLAGS}")
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND NOT ${CMAKE_BUILD_TYPE} STREQUAL "DebugSan")
	# Do not allow unresovled symbols in shared library (default on linux)
	set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined")
endif()

# LLVM
if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND NOT LLVM_DIR)
	# Workaround for Ubuntu broken LLVM package
	find_program(LLVM3_7_CONFIG llvm-config-3.7)
	find_program(LLVM3_8_CONFIG llvm-config-3.8)
	if (LLVM3_7_CONFIG)
		message(STATUS "Using llvm-3.7-dev package from Ubuntu. If does not work, build LLVM and set -DLLVM_DIR=llvm-build/share/llvm/cmake")
		set(LLVM_CONFIG_EXEC llvm-config-3.7)
		set(LLVM_LIB_DIR /usr/lib/llvm-3.7/lib)
	elseif(LLVM3_8_CONFIG)
			message(STATUS "Using llvm-3.8-dev package from Ubuntu. If does not work, build LLVM and set -DLLVM_DIR=llvm-build/share/llvm/cmake")
		set(LLVM_CONFIG_EXEC llvm-config-3.8)
		set(LLVM_LIB_DIR /usr/lib/llvm-3.8/lib)
	else()
		message(FATAL_ERROR "No LLVM package found!")
	endif()

	execute_process(COMMAND ${LLVM_CONFIG_EXEC} --includedir OUTPUT_VARIABLE LLVM_INCLUDE_DIRS OUTPUT_STRIP_TRAILING_WHITESPACE)
	message(STATUS "LLVM include dirs: ${LLVM_INCLUDE_DIRS}")
	if (LLVM3_8_CONFIG)
		set(LLVM_LIBS "-lLLVMipo -lLLVMVectorize -lLLVMX86AsmParser -lLLVMX86CodeGen -lLLVMX86Desc -lLLVMX86Info -lLLVMMCDisassembler -lLLVMX86AsmPrinter -lLLVMX86Utils -lLLVMSelectionDAG -lLLVMAsmPrinter -lLLVMCodeGen -lLLVMScalarOpts -lLLVMProfileData -lLLVMInstCombine -lLLVMInstrumentation -lLLVMTransformUtils -lLLVMMCJIT -lLLVMExecutionEngine -lLLVMTarget -lLLVMAnalysis -lLLVMRuntimeDyld -lLLVMObject -lLLVMMCParser -lLLVMBitReader -lLLVMMC -lLLVMCore -lLLVMSupport -lz -lpthread -lffi -ltinfo -ldl -lm")
	else()
		set(LLVM_LIBS "-lLLVMipo -lLLVMVectorize -lLLVMX86AsmParser -lLLVMX86CodeGen -lLLVMX86Desc -lLLVMX86Info -lLLVMMCDisassembler -lLLVMX86AsmPrinter -lLLVMX86Utils -lLLVMSelectionDAG -lLLVMAsmPrinter -lLLVMCodeGen -lLLVMScalarOpts -lLLVMProfileData -lLLVMInstCombine -lLLVMInstrumentation -lLLVMTransformUtils -lLLVMipa -lLLVMMCJIT -lLLVMExecutionEngine -lLLVMTarget -lLLVMAnalysis -lLLVMRuntimeDyld -lLLVMObject -lLLVMMCParser -lLLVMBitReader -lLLVMMC -lLLVMCore -lLLVMSupport -lz -lpthread -lffi -ltinfo -ldl -lm")
	endif()
	set(LLVM_DEFINITIONS "-D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS")
	link_directories(${LLVM_LIB_DIR})
else()
	find_package(LLVM REQUIRED CONFIG)
	if (${LLVM_VERSION} VERSION_LESS 3.7)
		message(FATAL_ERROR "Incompatible LLVM version ${LLVM_VERSION}")
	endif()
	message(STATUS "Found LLVM ${LLVM_VERSION}")
	message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")
	llvm_map_components_to_libnames(LLVM_LIBS core support mcjit x86asmparser x86codegen ipo)
endif()

add_subdirectory(libevmjit)

if(EVMJIT_CPP)
	add_subdirectory(libevmjit-cpp)
endif()
