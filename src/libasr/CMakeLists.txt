cmake_minimum_required(VERSION 3.10)

project(libasr)

if (NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17
        CACHE STRING "C++ standard" FORCE)
endif ()

if (NOT LFORTRAN_VERSION)
    set(LFORTRAN_VERSION "0.1-git"
        CACHE STRING "LFortran version" FORCE)
endif ()

configure_file(config.h.in ${CMAKE_CURRENT_SOURCE_DIR}/config.h)

set(SRC
    codegen/asr_to_cpp.cpp
    codegen/asr_to_c.cpp
    codegen/asr_to_julia.cpp
    codegen/asr_to_python.cpp
    codegen/asr_to_fortran.cpp
    codegen/asr_to_py.cpp
    codegen/x86_assembler.cpp
    codegen/asr_to_x86.cpp
    codegen/asr_to_wasm.cpp
    codegen/wasm_to_wat.cpp
    codegen/wasm_to_x86.cpp
    codegen/wasm_to_x64.cpp
    codegen/wasm_utils.cpp

    pass/nested_vars.cpp
    pass/array_struct_temporary.cpp
    pass/where.cpp
    pass/function_call_in_declaration.cpp
    pass/array_passed_in_function_call.cpp
    pass/openmp.cpp
    pass/param_to_const.cpp
    pass/do_loops.cpp
    pass/for_all.cpp
    pass/while_else.cpp
    pass/global_stmts.cpp
    pass/select_case.cpp
    pass/init_expr.cpp
    pass/implied_do_loops.cpp
    pass/array_op.cpp
    pass/subroutine_from_function.cpp
    pass/transform_optional_argument_functions.cpp
    pass/class_constructor.cpp
    pass/print_arr.cpp
    pass/print_struct_type.cpp
    pass/print_list_tuple.cpp
    pass/pass_utils.cpp
    pass/unused_functions.cpp
    pass/flip_sign.cpp
    pass/div_to_mul.cpp
    pass/replace_symbolic.cpp
    pass/intrinsic_function.cpp
    pass/intrinsic_subroutine.cpp
    pass/fma.cpp
    pass/loop_vectorise.cpp
    pass/sign_from_value.cpp
    pass/inline_function_calls.cpp
    pass/loop_unroll.cpp
    pass/dead_code_removal.cpp
    pass/instantiate_template.cpp
    pass/update_array_dim_intrinsic_calls.cpp
    pass/pass_array_by_data.cpp
    pass/pass_list_expr.cpp
    pass/pass_compare.cpp
    pass/unique_symbols.cpp
    pass/insert_deallocate.cpp
    pass/promote_allocatable_to_nonallocatable.cpp
    pass/replace_with_compile_time_values.cpp

    asr_verify.cpp
    asr_utils.cpp
    casting_utils.cpp
    asr_scopes.cpp
    modfile.cpp
    pickle.cpp
    serialization.cpp
)
if (WITH_LLVM)
    set(SRC ${SRC}
        codegen/evaluator.cpp
        codegen/asr_to_llvm.cpp
        codegen/llvm_array_utils.cpp
        codegen/llvm_utils.cpp
    )
    if (WITH_MLIR)
        set(SRC ${SRC} codegen/asr_to_mlir.cpp)
    endif()
    # We use deprecated API in LLVM, so we disable the warning until we upgrade
    if (NOT MSVC)
        set_source_files_properties(codegen/evaluator.cpp PROPERTIES
            COMPILE_FLAGS -Wno-deprecated-declarations)
        set_source_files_properties(codegen/asr_to_llvm.cpp PROPERTIES
            COMPILE_FLAGS -Wno-deprecated-declarations)
        set_source_files_properties(codegen/llvm_array_utils.cpp PROPERTIES
            COMPILE_FLAGS -Wno-deprecated-declarations)
        set_source_files_properties(codegen/llvm_utils.cpp PROPERTIES
            COMPILE_FLAGS -Wno-deprecated-declarations)
        set_source_files_properties(stacktrace.cpp PROPERTIES
            COMPILE_FLAGS -Wno-deprecated-declarations)
    endif()
endif()

set(LFORTRAN_UTILS
  assert.h
  colors.h
  exception.h
  location.h

  diagnostics.h
  diagnostics.cpp
  stacktrace.h
  stacktrace.cpp
  string_utils.cpp
  utils.h
  utils2.cpp
)
add_library(lfortran_utils OBJECT ${LFORTRAN_UTILS})
target_include_directories(lfortran_utils BEFORE PUBLIC ${libasr_SOURCE_DIR}/..)
target_include_directories(lfortran_utils BEFORE PUBLIC ${libasr_BINARY_DIR}/..)

add_library(asr STATIC ${SRC})
target_include_directories(asr BEFORE PUBLIC ${libasr_SOURCE_DIR}/..)
target_include_directories(asr BEFORE PUBLIC ${libasr_BINARY_DIR}/..)
target_link_libraries(asr lfortran_utils)
if (WITH_LIBUNWIND)
    target_link_libraries(asr p::libunwind)
endif()
if (WITH_BFD)
    target_link_libraries(asr p::bfd)
endif()
if (WITH_LINK)
    target_link_libraries(asr p::link)
endif()
if (WITH_EXECINFO)
    target_link_libraries(asr p::execinfo)
endif()
if (WITH_LLVM)
    target_link_libraries(asr p::llvm)
    target_link_libraries(lfortran_utils p::llvm)
endif()

# Install the dwarf_convert.py and dat_convert.py
install(
    FILES dwarf_convert.py dat_convert.py
    PERMISSIONS OWNER_EXECUTE OWNER_READ
    DESTINATION ${CMAKE_INSTALL_PREFIX}/share/lfortran
)
