Now, check the changes made since last git commit to see wht changes are needed to build on macos-amd64. But i do not understand why these changes are needed because:

cpuinfo_default.patch

--- a/BUILD.bazel
+++ b/BUILD.bazel
@@ -132,6 +132,7 @@ cc_library(
         ":tvos_x86_64": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
         ":tvos_arm64": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
         ":emscripten_wasm": COMMON_SRCS + EMSCRIPTEN_SRCS,
+        "//conditions:default": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
     }),
     copts = select({
         ":windows_x86_64": [],

However, the version of cpuinfo used by tensorflow is shown in tensorflow-2.11.0/tensorflow/workspace2.bzl

tf_http_archive(
        name = "cpuinfo",
        strip_prefix = "cpuinfo-5e63739504f0f8e18e941bd63b2d6d42536c7d90",
        sha256 = "18eca9bc8d9c4ce5496d0d2be9f456d55cbbb5f0639a551ce9c8bac2e84d85fe",
        urls = tf_mirror_urls("https://github.com/pytorch/cpuinfo/archive/5e63739504f0f8e18e941bd63b2d6d42536c7d90.tar.gz"),
    )

However, if you look at cpuinfo-5e63739504f0f8e18e941bd63b2d6d42536c7d90/BUILD.bazel file

cc_library(
    name = "cpuinfo_impl",
    srcs = select({
        ":linux_x86_64": COMMON_SRCS + X86_SRCS + LINUX_SRCS + LINUX_X86_SRCS,
        ":linux_arm": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM32_SRCS,
        ":linux_armhf": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM32_SRCS,
        ":linux_armv7a": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM32_SRCS,
        ":linux_armeabi": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM32_SRCS,
        ":linux_aarch64": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM64_SRCS,
        ":linux_mips64": COMMON_SRCS + LINUX_SRCS,
        ":linux_riscv64": COMMON_SRCS + LINUX_SRCS,
        ":linux_s390x": COMMON_SRCS + LINUX_SRCS,
        ":macos_x86_64": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":macos_arm64": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":windows_x86_64": COMMON_SRCS + X86_SRCS + WINDOWS_X86_SRCS,
        ":android_armv7": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM32_SRCS + ANDROID_ARM_SRCS,
        ":android_arm64": COMMON_SRCS + ARM_SRCS + LINUX_SRCS + LINUX_ARM64_SRCS + ANDROID_ARM_SRCS,
        ":android_x86": COMMON_SRCS + X86_SRCS + LINUX_SRCS + LINUX_X86_SRCS,
        ":android_x86_64": COMMON_SRCS + X86_SRCS + LINUX_SRCS + LINUX_X86_SRCS,
        ":ios_x86_64": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":ios_x86": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":ios_armv7": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":ios_arm64": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":ios_arm64e": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":ios_sim_arm64": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":watchos_x86_64": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":watchos_x86": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":watchos_armv7k": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":watchos_arm64_32": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":tvos_x86_64": COMMON_SRCS + X86_SRCS + MACH_SRCS + MACH_X86_SRCS,
        ":tvos_arm64": COMMON_SRCS + MACH_SRCS + MACH_ARM_SRCS,
        ":emscripten_wasm": COMMON_SRCS + EMSCRIPTEN_SRCS,
    }),

it already has macos_x86_64 scenario

xnnpack_default.patch

--- a/build_defs.bzl
+++ b/build_defs.bzl
@@ -146,7 +146,7 @@ def xnnpack_cc_library(
             ":emscripten_wasm": wasm_srcs,
             ":emscripten_wasmsimd": wasmsimd_srcs,
             ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_srcs,
-            "//conditions:default": [],
+            "//conditions:default": x86_srcs,
         }),
         copts = [
             "-Iinclude",
@@ -179,7 +179,7 @@ def xnnpack_cc_library(
             ":emscripten_wasm": wasm_copts,
             ":emscripten_wasmsimd": wasmsimd_copts,
             ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_copts,
-            "//conditions:default": [],
+            "//conditions:default": gcc_x86_copts,
         }) + select({
             ":windows_x86_64_clang": ["/clang:" + opt for opt in gcc_copts],
             ":windows_x86_64_mingw": gcc_copts,
@@ -249,6 +249,7 @@ def xnnpack_aggregate_library(
             ":emscripten_wasmsimd": wasmsimd_deps,
             ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_deps,
             ":riscv": riscv_deps,
+            "//conditions:default": x86_deps,
         }),
         defines = defines,
         compatible_with = compatible_with,


Similarly, we can find the xnnpack version used by tensorflow in  tensorflow-2.11.0/tensorflow/workspace2.bzl

    tf_http_archive(
        name = "XNNPACK",
        sha256 = "7a16ab0d767d9f8819973dbea1dc45e4e08236f89ab702d96f389fdc78c5855c",
        strip_prefix = "XNNPACK-e8f74a9763aa36559980a0c2f37f587794995622",
        urls = tf_mirror_urls("https://github.com/google/XNNPACK/archive/e8f74a9763aa36559980a0c2f37f587794995622.zip"),
    )
    # LINT.ThenChange(//tensorflow/lite/tools/cmake/modules/xnnpack.cmake)

However, if you look at XNNPACK-e8f74a9763aa36559980a0c2f37f587794995622/build_defs.bzl you will see that all stuff necessarry for macos-amd64 is already present

    native.cc_library(
        name = name,
        srcs = srcs + select({
            ":aarch32": aarch32_srcs,
            ":aarch64": aarch64_srcs,
            ":riscv": riscv_srcs,
            ":x86": x86_srcs,
            ":emscripten_wasm": wasm_srcs,
            ":emscripten_wasmsimd": wasmsimd_srcs,
            ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_srcs,
            "//conditions:default": [],
        }),

x86 is already present here


copts = [
            "-Iinclude",
            "-Isrc",
        ] + copts + select({
            ":linux_k8": gcc_x86_copts,
            ":linux_arm": aarch32_copts,
            ":linux_armeabi": aarch32_copts,
            ":linux_armhf": aarch32_copts,
            ":linux_armv7a": aarch32_copts,
            ":linux_arm64": aarch64_copts,
            ":macos_x86_64": gcc_x86_copts,
            ":macos_arm64": aarch64_copts,
            ":windows_x86_64_clang": ["/clang:" + opt for opt in gcc_x86_copts],
            ":windows_x86_64_mingw": mingw_copts + gcc_x86_copts,
            ":windows_x86_64_msys": msys_copts + gcc_x86_copts,
            ":windows_x86_64": msvc_x86_64_copts,
            ":android_armv7": aarch32_copts,
            ":android_arm64": aarch64_copts,
            ":android_x86": gcc_x86_copts,
            ":android_x86_64": gcc_x86_copts,
            ":ios_arm64": aarch64_copts,
            ":ios_arm64e": aarch64_copts,
            ":ios_sim_arm64": aarch64_copts,
            ":ios_x86_64": gcc_x86_copts,
            ":watchos_arm64_32": aarch64_copts,
            ":watchos_x86_64": gcc_x86_copts,
            ":tvos_arm64": aarch64_copts,
            ":tvos_x86_64": gcc_x86_copts,
            ":emscripten_wasm": wasm_copts,
            ":emscripten_wasmsimd": wasmsimd_copts,
            ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_copts,
            "//conditions:default": [],
        }) + select({
            ":windows_x86_64_clang": ["/clang:" + opt for opt in gcc_copts],
            ":windows_x86_64_mingw": gcc_copts,
            ":windows_x86_64_msys": gcc_copts,
            ":windows_x86_64": msvc_copts,
            "//conditions:default": gcc_copts,
        }) + select({
            ":optimized_build": optimized_copts,
            "//conditions:default": [],
        }),

macos_x86_64 is already present



    native.cc_library(
        name = name,
        linkstatic = True,
        deps = generic_deps + select({
            ":aarch32": aarch32_deps,
            ":aarch64": aarch64_deps,
            ":x86": x86_deps,
            ":emscripten_wasm": wasm_deps,
            ":emscripten_wasmsimd": wasmsimd_deps,
            ":emscripten_wasmrelaxedsimd": wasmrelaxedsimd_deps,
            ":riscv": riscv_deps,
        }),

x86 is already present here.

But if i remove these patches, the build fails with the error shown in logs/mac.txt