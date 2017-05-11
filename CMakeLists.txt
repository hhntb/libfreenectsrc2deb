cmake_minimum_required(VERSION 3.7)
project(libfreenectsrc2deb)

set(CMAKE_CXX_STANDARD 14)

include(ExternalProject)

set(PACKAGE_VERSION 0.2.0)
set(PACKAGE_NAME libfreenect2)
set(SOURCE_TMP_DIRECTORY tmp)
set(PACKAGE_BUILD_DIRECTORY ${PROJECT_SOURCE_DIR}/${SOURCE_TMP_DIRECTORY}/${PACKAGE_NAME}_${PACKAGE_VERSION})
set(PACKAGE_RELATIVE_INSTALL_DIRECTORY /usr/local)

ExternalProject_Add(
        freenect2_build
        GIT_REPOSITORY https://github.com/hhntb/libfreenect2.git
        GIT_TAG v${PACKAGE_VERSION}
        CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${PACKAGE_BUILD_DIRECTORY}/${PACKAGE_RELATIVE_INSTALL_DIRECTORY}
)
ExternalProject_Get_Property(freenect2_build SOURCE_DIR)


message(${CMAKE_COMMAND})

configure_file(control ${PACKAGE_BUILD_DIRECTORY}/DEBIAN/control)

add_custom_command(TARGET freenect2_build POST_BUILD COMMAND ${CMAKE_COMMAND}
        -DSRC_FILE="${SOURCE_DIR}/platform/linux/udev/90-kinect2.rules"
        -DDST_FILE="${PACKAGE_BUILD_DIRECTORY}/etc/udev/rules.d/90-kinect2.rules"
        -DCPY_MODE=COPYONLY
        -P configure_file.cmake WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})

add_custom_command(TARGET freenect2_build POST_BUILD COMMAND ${CMAKE_COMMAND}
        -DSRC_FILE="${SOURCE_DIR}/freenect2.pc.in"
        -DDST_FILE="${PACKAGE_BUILD_DIRECTORY}/${PACKAGE_RELATIVE_INSTALL_DIRECTORY}/lib/pkgconfig/freenect2.pc"
        -DCMAKE_INSTALL_PREFIX="${PACKAGE_RELATIVE_INSTALL_DIRECTORY}"
        -DCPY_MODE=@ONLY
        -P configure_file.cmake WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})

add_custom_command(TARGET freenect2_build POST_BUILD COMMAND ${CMAKE_COMMAND}
        -DSRC_FILE="${SOURCE_DIR}/freenect2.cmake.in"
        -DDST_FILE="${PACKAGE_BUILD_DIRECTORY}/${PACKAGE_RELATIVE_INSTALL_DIRECTORY}/lib/cmake/freenect2/freenect2Config.cmake"
        -DCMAKE_INSTALL_PREFIX="${PACKAGE_RELATIVE_INSTALL_DIRECTORY}"
        -DPROJECT_VER="${PACKAGE_VERSION}"
        -DPROJECT_NAME="${PACKAGE_NAME}"
        -DLIBFREENECT2_THREADING_LIBRARIES="pthread"
        -DCPY_MODE=@ONLY
        -P configure_file.cmake WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})

add_custom_command(TARGET freenect2_build POST_BUILD COMMAND fakeroot dpkg-deb --build ${PACKAGE_NAME}_${PACKAGE_VERSION} WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/${SOURCE_TMP_DIRECTORY}/)
