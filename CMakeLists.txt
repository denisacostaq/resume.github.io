cmake_minimum_required(VERSION 3.10)
project(Resume VERSION 0.1)

find_program(PANDOC_EXE pandoc)
message("PANDOC_EXE ${PANDOC_EXE}")
if(NOT PANDOC_EXE)
    message(FATAL_ERROR "pandoc not found (https://pandoc.org/installing.html)!")
endif()
find_program(MTXRUN_EXE mtxrun)
if(NOT MTXRUN_EXE)
    message(FATAL_ERROR "ConTeXt TDS Runner Tool not found (https://wiki.contextgarden.net/Installation)")
endif()

#FIXME(denisacostaq@gmail.com): setting LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 in
# the cmd line is not portable, #use set(ENV{LC_ALL} en_US.UTF-8)
# set(ENV{LANG} en_US.UTF-8)
add_custom_target(pdf ALL
    COMMAND LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 ${MTXRUN_EXE} --path=${CMAKE_CURRENT_BINARY_DIR} --result=${CMAKE_CURRENT_BINARY_DIR}/resume.pdf --script context ${CMAKE_CURRENT_BINARY_DIR}/resume.tex > ${CMAKE_CURRENT_BINARY_DIR}/context.log 2>&1
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/resume.tex
    BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/resume.pdf
    COMMENT "Generating pdf file"
    SOURCES ${CMAKE_CURRENT_BINARY_DIR}/resume.tex)
set(CMAKE_VERBOSE_MAKEFILE ON)
add_custom_target(latext
    COMMAND ${PANDOC_EXE} --standalone --from markdown --to context --variable papersize=A4 --output ${CMAKE_CURRENT_BINARY_DIR}/resume.tex ${CMAKE_CURRENT_SOURCE_DIR}/README.md
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/README.md
    BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/resume.tex
    COMMENT "Generating latext file"
    SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/README.md)
