cmake_minimum_required(VERSION 3.4)
project(winbagility)

set(CMAKE_CXX_STANDARD 17)

include(common.cmake)
get_filename_component(root_dir     ${CMAKE_CURRENT_SOURCE_DIR}/.. ABSOLUTE)
get_filename_component(bin_d_dir    ${root_dir}/bin/fdp_d_${ARCH}  ABSOLUTE)
get_filename_component(bin_dir      ${root_dir}/bin/fdp_${ARCH}    ABSOLUTE)

# fdp
function(add_fdp target options)
    add_target(${target} libs "${root_dir}/src/FDP" OPTIONS recurse warnings ${options})
    set_target_output_directory(${target} "")
    target_include_directories(${target} PUBLIC "${root_dir}/src/FDP/include")
endfunction()
add_fdp(fdp_static "")
add_fdp(fdp shared)

# winbagility
add_target(winbagility apps "${root_dir}/src/Winbagility" OPTIONS executable recurse warnings)
set_target_output_directory(winbagility "")
target_include_directories(winbagility PRIVATE
    "${root_dir}/src/Winbagility"
    "${root_dir}/src/Winbagility/STUB"
)
target_link_libraries(winbagility PRIVATE
    dbghelp
    fdp
    ws2_32
)

# test_fdp
add_target(test_fdp tests "${root_dir}/src/TestFDP" OPTIONS executable warnings)
set_target_output_directory(test_fdp "")
target_link_libraries(test_fdp PRIVATE
    fdp_static
)
target_include_directories(test_fdp PRIVATE
    "${root_dir}/src/Winbagility"
)

# hbp
add_target(hbp libs "${root_dir}/src/HDBPExtension" OPTIONS shared warnings)
set_target_output_directory(hbp "")
target_link_libraries(hbp PRIVATE
    fdp
)

# loguru
set(loguru_dir "${root_dir}/deps/loguru-2.0.0")
add_target(loguru deps "${loguru_dir}" OPTIONS external)
target_compile_definitions(loguru PRIVATE
    _CRT_SECURE_NO_WARNINGS
)
target_include_directories(loguru PUBLIC
    "${loguru_dir}"
)

# pdbparser
set(retdec_dir "${root_dir}/deps/retdec-3.2")
add_target(pdbparser deps "${retdec_dir}/src/pdbparser" "${retdec_dir}/include/retdec/pdbparser" OPTIONS external)
target_include_directories(pdbparser PUBLIC
    "${retdec_dir}/include"
)

# fdp_exec
set(fdp_exec_dir "${root_dir}/src/fdp_exec")
add_target(fdp_exec apps ${fdp_exec_dir} OPTIONS recurse executable)
set_target_output_directory(fdp_exec "")
target_include_directories(fdp_exec PRIVATE
    "${fdp_exec_dir}"
)
target_compile_definitions(fdp_exec PRIVATE
    _CRT_SECURE_NO_WARNINGS
    _SILENCE_CXX17_CODECVT_HEADER_DEPRECATION_WARNING
)
target_link_libraries(fdp_exec PRIVATE
    fdp_static
    loguru
    pdbparser
)