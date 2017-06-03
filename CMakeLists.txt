
cmake_minimum_required( VERSION 3.2 ) 
set( CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Supported configuration types" FORCE )
        
project( date-adapter LANGUAGES CXX )
get_directory_property( is_subproject PARENT_DIRECTORY )
include( CMakeDependentOption REQUIRED )

if ( NOT GIT_EXECUTABLE )
    find_package( Git )
    if ( NOT GIT_FOUND )
        message( FATAL_ERROR "git installation was not found." )
    endif()
endif()

execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse --abbrev-ref HEAD
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_BRANCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_HASH
    OUTPUT_STRIP_TRAILING_WHITESPACE
) 

message( STATUS "" )
message( STATUS "-----------------------------------------------------------" )
message( STATUS "" )
message( STATUS "date-adapter" )
message( STATUS "Git current branch: ${GIT_BRANCH}" )
message( STATUS "Git commit hash: ${GIT_HASH}" )
message( STATUS "" )
message( STATUS "-----------------------------------------------------------" )

add_library( date-adapter INTERFACE )
target_include_directories( date-adapter INTERFACE src )
target_sources( date-adapter INTERFACE
                "${CMAKE_CURRENT_SOURCE_DIR}/src/chrono_io.h"
                "${CMAKE_CURRENT_SOURCE_DIR}/src/date.h"
                "${CMAKE_CURRENT_SOURCE_DIR}/src/ios.h" 
                "${CMAKE_CURRENT_SOURCE_DIR}/src/iso_week.h"
                "${CMAKE_CURRENT_SOURCE_DIR}/src/islamic.h"
                "${CMAKE_CURRENT_SOURCE_DIR}/src/julian.h" )

if ( time-zone_support )

  find_package( CURL REQUIRED )
  # general properties
  option( strict "Compile time warnings are converted to errors" ON )

  # binary instrumentation
  option( coverage "Enable binary instrumentation to collect test coverage information in the DEBUG configuration" )
  option( profile_generate "Enable binary instrumentation to generation execution profiles in the RELEASE configuration which may be used to guide later optimization" )

  # additional optimizations
  option( link_time_optimization "Enable link time optimization in the RELEASE configuration" )
  option( profile_use "In the RELEASE configuration, leverage previously generated exeution profile to inform optimization decisions" )
  option( nonportable_optimization "Enable optimizations which compromise portability of resulting binary in the RELEASE configuration" )

  # libraries and linking
  option( static "Statically link component and environment libraries" OFF )
  if ( static AND ( "${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin" ) )
    message( FATAL_ERROR "Static binaries not supported on OSX" )
  endif()
  CMAKE_DEPENDENT_OPTION( static_libraries "Statically link component libraries" OFF "NOT static" ON )
  CMAKE_DEPENDENT_OPTION( static_time-zone "Statically link the time-sone component library" OFF "NOT static;NOT static_libraries" ON ) 

  option( unit_tests "Compile the interpolation unit tests and integrate with ctest" ON ) 

  if ( profile_generate AND profile_use )
    message( FATAL_ERROR "Cannot both generate and use execution profile in the same configuration" )
  endif()

  if ( profile_generate )
    file( MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/profiling" )
  endif()

  set( time-zone_Clang_Windows_common_flags "-stdlib=libc++" "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_Clang_Windows_DEBUG_flags "-O0" "-g" "-gdwarf-3" )
  set( time-zone_Clang_Windows_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_Clang_Windows_strict_flags "-Werror" )
  set( time-zone_Clang_Windows_coverage_flags "--coverage" )
  set( time-zone_Clang_Windows_subproject_flags  )
  set( time-zone_Clang_Windows_base_project_flags  )
  set( time-zone_Clang_Windows_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Windows_link_time_optimization_flags "-flto" )
  set( time-zone_Clang_Windows_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Windows_nonportable_optimization_flags "-march=native" )
  set( time-zone_Clang_Windows_static_flags "-static" )
  set( time-zone_Clang_Darwin_common_flags "-stdlib=libc++" "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_Clang_Darwin_DEBUG_flags "-O0" "-g" "-gdwarf-3" )
  set( time-zone_Clang_Darwin_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_Clang_Darwin_strict_flags "-Werror" )
  set( time-zone_Clang_Darwin_coverage_flags "--coverage" )
  set( time-zone_Clang_Darwin_subproject_flags  )
  set( time-zone_Clang_Darwin_base_project_flags  )
  set( time-zone_Clang_Darwin_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Darwin_link_time_optimization_flags "-flto" )
  set( time-zone_Clang_Darwin_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Darwin_nonportable_optimization_flags "-march=native" )
  set( time-zone_Clang_Darwin_static_flags "-static" )
  set( time-zone_Clang_Linux_common_flags "-stdlib=libstdc++" "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_Clang_Linux_DEBUG_flags "-O0" "-g" "-gdwarf-3" )
  set( time-zone_Clang_Linux_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_Clang_Linux_strict_flags "-Werror" )
  set( time-zone_Clang_Linux_coverage_flags "--coverage" )
  set( time-zone_Clang_Linux_subproject_flags  )
  set( time-zone_Clang_Linux_base_project_flags  )
  set( time-zone_Clang_Linux_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Linux_link_time_optimization_flags "-flto" )
  set( time-zone_Clang_Linux_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_Clang_Linux_nonportable_optimization_flags "-march=native" )
  set( time-zone_Clang_Linux_static_flags "-static" )
  set( time-zone_AppleClang_Darwin_common_flags "-stdlib=libc++" "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_AppleClang_Darwin_DEBUG_flags "-O0" "-g" "-gdwarf-3" )
  set( time-zone_AppleClang_Darwin_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_AppleClang_Darwin_strict_flags "-Werror" )
  set( time-zone_AppleClang_Darwin_coverage_flags "--coverage" )
  set( time-zone_AppleClang_Darwin_subproject_flags  )
  set( time-zone_AppleClang_Darwin_base_project_flags  )
  set( time-zone_AppleClang_Darwin_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_AppleClang_Darwin_link_time_optimization_flags "-flto" )
  set( time-zone_AppleClang_Darwin_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_AppleClang_Darwin_nonportable_optimization_flags "-march=native" )
  set( time-zone_AppleClang_Darwin_static_flags "-static" )
  set( time-zone_GNU_Windows_common_flags "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_GNU_Windows_DEBUG_flags "-O0" "-g" "-gdwarf-3" "-fsignaling-nans" )
  set( time-zone_GNU_Windows_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_GNU_Windows_strict_flags "-Werror" )
  set( time-zone_GNU_Windows_coverage_flags "--coverage" )
  set( time-zone_GNU_Windows_subproject_flags  )
  set( time-zone_GNU_Windows_base_project_flags  )
  set( time-zone_GNU_Windows_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Windows_link_time_optimization_flags "-flto" )
  set( time-zone_GNU_Windows_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Windows_nonportable_optimization_flags "-march=native" )
  set( time-zone_GNU_Windows_static_flags "-static" )
  set( time-zone_GNU_Darwin_common_flags "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_GNU_Darwin_DEBUG_flags "-O0" "-g" "-gdwarf-3" "-fsignaling-nans" )
  set( time-zone_GNU_Darwin_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_GNU_Darwin_strict_flags "-Werror" )
  set( time-zone_GNU_Darwin_coverage_flags "--coverage" )
  set( time-zone_GNU_Darwin_subproject_flags  )
  set( time-zone_GNU_Darwin_base_project_flags  )
  set( time-zone_GNU_Darwin_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Darwin_link_time_optimization_flags "-flto" )
  set( time-zone_GNU_Darwin_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Darwin_nonportable_optimization_flags "-march=native" )
  set( time-zone_GNU_Darwin_static_flags "-static" )
  set( time-zone_GNU_Linux_common_flags "-Wall" "-Wextra" "-Wpedantic" "-std=c++14" )
  set( time-zone_GNU_Linux_DEBUG_flags "-O0" "-g" "-gdwarf-3" "-fsignaling-nans" )
  set( time-zone_GNU_Linux_RELEASE_flags "-O3" "-DNDEBUG" )
  set( time-zone_GNU_Linux_strict_flags "-Werror" )
  set( time-zone_GNU_Linux_coverage_flags "--coverage" )
  set( time-zone_GNU_Linux_subproject_flags  )
  set( time-zone_GNU_Linux_base_project_flags  )
  set( time-zone_GNU_Linux_profile_generate_flags "-fprofile-generate='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Linux_link_time_optimization_flags "-flto" )
  set( time-zone_GNU_Linux_profile_use_flags "-fprofile-use='${CMAKE_BINARY_DIR}/profiling'" )
  set( time-zone_GNU_Linux_nonportable_optimization_flags "-march=native" )
  set( time-zone_GNU_Linux_static_flags "-static" )
  get_directory_property( is_subproject PARENT_DIRECTORY )

  if ( static_time-zone )
    set( time-zone_library_linkage STATIC )
  else ()
    set( time-zone_library_linkage SHARED )
  endif () 

  add_library( time-zone ${time-zone_library_linkage}
               "${CMAKE_CURRENT_SOURCE_DIR}/src/tz.h"
               "${CMAKE_CURRENT_SOURCE_DIR}/src/tz_private.h"
               "${CMAKE_CURRENT_SOURCE_DIR}/src/tz.cpp" )

  target_include_directories( time-zone PUBLIC src PRIVATE ${CURL_INCLUDE_DIRS} )

  set( PREFIX time-zone_${CMAKE_CXX_COMPILER_ID}_${CMAKE_SYSTEM_NAME} )           

  target_compile_options( time-zone PRIVATE
    ${${PREFIX}_common_flags}
    $<$<BOOL:${strict}>:${${PREFIX}_strict_flags}>$<$<CONFIG:DEBUG>:
    ${${PREFIX}_DEBUG_flags}
    $<$<BOOL:${coverage}>:${${PREFIX}_coverage_flags}>>
    $<$<CONFIG:RELEASE>:
    ${${PREFIX}_RELEASE_flags}
    $<$<BOOL:${link_time_optimization}>:${${PREFIX}_link_time_optimization_flags}>
    $<$<BOOL:${nonportable_optimization}>:${${PREFIX}_nonportable_optimization_flags}>>
    ${CXX_appended_flags} ${time-zone_appended_flags} )

  target_link_libraries( time-zone PRIVATE "$<$<AND:$<CONFIG:RELEASE>,$<BOOL:${link_time_optimization}>>:${${PREFIX}_RELEASE_flags};${${PREFIX}_link_time_optimization_flags}$<$<BOOL:${profile_generate}>:${${PREFIX}_profile_generate_flags};>$<$<BOOL:${profile_use}>:${${PREFIX}_profile_use_flags};>$<$<BOOL:${nonportable_optimization}>:${${PREFIX}_nonportable_optimization_flags};>>$<$<CONFIG:DEBUG>:$<$<BOOL:${coverage}>:${${PREFIX}_coverage_flags};>>$<$<BOOL:CXX_appended_flags>:${CXX_appended_flags};>$<$<BOOL:time-zone_appended_flags>:${time-zone_appended_flags};>" )
   
  target_link_libraries( time-zone PUBLIC date-adapter PUBLIC ${CURL_LIBRARIES} )
endif()
