cmake_minimum_required(VERSION 4.3)
project(URHI LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_library(URHI SHARED
        ICommandList.hpp
        ICommandQueue.hpp
        IDeviceContext.hpp
        IDeviceFactory.hpp
        Base.cpp)

include(GenerateExportHeader)
generate_export_header(URHI)

target_include_directories(URHI PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)

find_package(VulkanHeaders CONFIG)
find_package(vk-bootstrap CONFIG REQUIRED)
find_package(volk CONFIG REQUIRED)
find_package(unofficial-vulkan-memory-allocator-hpp CONFIG REQUIRED)

target_link_libraries(URHI PRIVATE Vulkan::Headers
        vk-bootstrap::vk-bootstrap
        vk-bootstrap::vk-bootstrap-compiler-warnings
        volk::volk volk::volk_headers
        unofficial::VulkanMemoryAllocator-Hpp::VulkanMemoryAllocator-Hpp)

if(WIN32)
    find_package(D3D12MemoryAllocator CONFIG REQUIRED)
    find_package(directx-headers CONFIG REQUIRED)
    find_package(directx12-agility CONFIG REQUIRED)

    target_link_libraries(URHI PRIVATE
            Microsoft::DirectX-Headers
            Microsoft::DirectX-Guids
            Microsoft::DirectX12-Agility
            GPUOpen::D3D12MemoryAllocator)

    if(TARGET Microsoft::DirectX12-Agility)
        file(MAKE_DIRECTORY "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/D3D12")
        add_custom_command(TARGET URHI POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DirectX12-Core,IMPORTED_LOCATION_RELEASE> $<TARGET_FILE_DIR:URHI>/D3D12
                COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DirectX12-Layers,IMPORTED_LOCATION_DEBUG> $<TARGET_FILE_DIR:URHI>/D3D12
                COMMAND_EXPAND_LISTS
        )
    endif()
endif()