file(GLOB SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp ${CMAKE_CURRENT_SOURCE_DIR}/*.h)

file(GLOB IMAGES
    ${CMAKE_CURRENT_SOURCE_DIR}/images/*.jpg
    ${CMAKE_CURRENT_SOURCE_DIR}/images/*.png
    ${CMAKE_CURRENT_SOURCE_DIR}/images/*.bmp
)

foreach(IMAGE ${IMAGES})
    get_filename_component(fname ${IMAGE} NAME)
    configure_file(${IMAGE} ${CMAKE_CURRENT_BINARY_DIR}/images/${fname} COPYONLY)
endforeach()

file(GLOB_RECURSE SHADER_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/*.vert
    ${CMAKE_CURRENT_SOURCE_DIR}/*.frag
    ${CMAKE_CURRENT_SOURCE_DIR}/*.geom
    ${CMAKE_CURRENT_SOURCE_DIR}/*.comp
    ${CMAKE_CURRENT_SOURCE_DIR}/*.tese
    ${CMAKE_CURRENT_SOURCE_DIR}/*.tesc
)

source_group("Shaders" FILES ${SHADER_SOURCES})

if(WIN32)
    add_executable(vulkan_grass_rendering WIN32 ${SOURCES} ${SHADER_SOURCES})
    target_link_libraries(vulkan_grass_rendering ${WINLIBS})
else(WIN32)
    add_executable(vulkan_grass_rendering ${SOURCES} ${SHADER_SOURCES})
    target_link_libraries(vulkan_grass_rendering ${CMAKE_THREAD_LIBS_INIT})
endif(WIN32)

set(SHADER_DIR ${CMAKE_CURRENT_BINARY_DIR}/shaders)
file(MAKE_DIRECTORY ${SHADER_DIR})
if(WIN32)
  set(GLSL_VALIDATOR_CMD $ENV{VK_SDK_PATH}/Bin/glslangValidator.exe)
else(WIN32)
  set(GLSL_VALIDATOR_CMD $ENV{VULKAN_SDK}/bin/glslangValidator)
endif(WIN32)

foreach(SHADER_SOURCE ${SHADER_SOURCES})
    get_filename_component(fname ${SHADER_SOURCE} NAME)
    add_custom_target(${fname}.spv
        COMMAND ${GLSL_VALIDATOR_CMD} -V ${SHADER_SOURCE} -o ${SHADER_DIR}/${fname}.spv
        DEPENDS ${SHADER_SOURCE}
        SOURCES ${SHADER_SOURCE}
    )
    ExternalTarget("Shaders" ${fname}.spv)
    add_dependencies(vulkan_grass_rendering ${fname}.spv)
endforeach()

target_link_libraries(vulkan_grass_rendering ${ASSIMP_LIBRARIES} Vulkan::Vulkan glfw)
target_include_directories(vulkan_grass_rendering PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${GLM_INCLUDE_DIR}
  ${STB_INCLUDE_DIR}
)

InternalTarget("" vulkan_grass_rendering)

