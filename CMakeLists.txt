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

add_custom_target(context
    COMMAND ${PANDOC_EXE} --standalone --from markdown --to context --variable papersize=A4 --output ${CMAKE_CURRENT_BINARY_DIR}/resume.tex ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md
    BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/resume.tex
    COMMENT "Generating ConTeXt file"
    SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md)
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
add_custom_target(html
    COMMAND ${PANDOC_EXE} -c ${CMAKE_CURRENT_SOURCE_DIR}/src/style.css --standalone --from markdown --to html --output ${CMAKE_CURRENT_BINARY_DIR}/resume.html ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md ${CMAKE_CURRENT_SOURCE_DIR}/src/style.css
    BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/resume.html
    COMMENT "Generating html file"
    SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md)
add_custom_target(txt
    COMMAND ${PANDOC_EXE} --standalone --from markdown --to plain+smart --output ${CMAKE_CURRENT_BINARY_DIR}/resume.txt ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md ${CMAKE_CURRENT_SOURCE_DIR}/src/style.css
    BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/resume.txt
    COMMENT "Generating html file"
    SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/README.md)

