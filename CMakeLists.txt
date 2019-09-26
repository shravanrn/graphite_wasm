cmake_minimum_required(VERSION 3.11)

project(libgraphite
        VERSION 1
        DESCRIPTION "libgraphite included in Firefox")

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(wasiclang_SOURCE_DIR "${CMAKE_SOURCE_DIR}/../rlbox_lucet_sandbox/build/_deps/wasiclang-src")
set(LUCET_WASI_DIR "${CMAKE_SOURCE_DIR}/../rlbox_lucet_sandbox/build/_deps/mod_lucet-src/lucet-wasi")
set(LUCET_DIR "${CMAKE_SOURCE_DIR}/../rlbox_lucet_sandbox/build/cargo/release")

set(CMAKE_C_COMPILER ${wasiclang_SOURCE_DIR}/opt/wasi-sdk/bin/clang)
set(CMAKE_CXX_COMPILER ${wasiclang_SOURCE_DIR}/opt/wasi-sdk/bin/clang++)
set(CMAKE_BUILD_TYPE Release)

# Apply settings suitable for wasm module compilation
set(CMAKE_C_FLAGS
    "--sysroot ${wasiclang_SOURCE_DIR}/opt/wasi-sdk/share/wasi-sysroot/")
set(CMAKE_CXX_FLAGS
    "--sysroot ${wasiclang_SOURCE_DIR}/opt/wasi-sdk/share/wasi-sysroot/")
set(CMAKE_EXE_LINKER_FLAGS "-Wl,--export-all")

add_executable(libgraphite.wasm
            "${CMAKE_SOURCE_DIR}/../rlbox_lucet_sandbox/c_src/lucet_sandbox_wrapper.c"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/call_machine.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/CmapCache.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Code.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Collider.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Decompressor.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Face.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/FeatureMap.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/FileFace.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Font.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/GlyphCache.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/GlyphFace.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_char_info.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_face.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_features.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_font.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_logging.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_segment.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/gr_slot.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/GraphiteExtra.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Intervals.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/json.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Justifier.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/NameTable.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Pass.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Position.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Segment.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Silf.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Slot.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/Sparse.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/TtfUtil.cpp"
            "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/UtfCodec.cpp"
)

target_include_directories(libgraphite.wasm PUBLIC "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/include/")
target_include_directories(libgraphite.wasm PUBLIC "${CMAKE_SOURCE_DIR}/../mozilla-latest/src_git/gfx/graphite2/src/")
target_compile_definitions(libgraphite.wasm PRIVATE GRAPHITE2_STATIC=True)
target_compile_definitions(libgraphite.wasm PRIVATE GRAPHITE2_NFILEFACE=True)
target_compile_definitions(libgraphite.wasm PRIVATE GRAPHITE2_NTRACING=True)
target_compile_definitions(libgraphite.wasm PRIVATE GRAPHITE2_NSEGCACHE=True)

set(GRAPHITE_OUT_PATH "${CMAKE_BINARY_DIR}/libgraphite_wasm.so")

add_custom_command(OUTPUT ${GRAPHITE_OUT_PATH}
                   DEPENDS libgraphite.wasm
                   COMMAND ${LUCET_DIR}/lucetc
                           --bindings
                           ${LUCET_WASI_DIR}/bindings.json
                           --guard-size
                           "4GiB"
                           --min-reserved-size
                           "4GiB"
                           --max-reserved-size
                           "4GiB"
                           libgraphite.wasm
                           --opt-level
                           2
                           -o
                           ${GRAPHITE_OUT_PATH}
                   COMMENT "Compiling wasm file to native")

add_custom_target(libgraphite_so ALL DEPENDS ${GRAPHITE_OUT_PATH})