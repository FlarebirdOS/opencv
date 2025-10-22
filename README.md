## 未知原因 编译无法通过

```
# In general, we want to list all real archs (sm_XX) and the latest virtual arch (compute_XX) for future PTX compatibility.
# Valid values can be discovered from nvcc --help
local cuda_archs="75;80;86;87;88;89;90;100;103;110;120;121;121-virtual"

# Avoid nvcc intercepting -Werror=format-security: Value 'format-security' is not defined for option 'Werror'
CUDAFLAGS="${CXXFLAGS/-Werror=format-security/-Xcompiler -Werror=format-security} -fno-lto --threads 0" \
CFLAGS="${CFLAGS} -fno-lto" CXXFLAGS="${CXXFLAGS} -fno-lto" LDFLAGS="${LDFLAGS} -fno-lto" \
cmake -B flarebird-build-cuda                       \
      -D BUILD_WITH_DEBUG_INFO=OFF                  \
      -D WITH_CUDA=ON                               \
      -D WITH_CUDNN=ON                              \
      -D ENABLE_CUDA_FIRST_CLASS_LANGUAGE=ON        \
      -D CMAKE_CUDA_ARCHITECTURES="${cuda_archs}"   \
      "${cmake_args[@]}"

cmake --build flarebird-build-cuda --verbose
```

## opencv-cuda 和 python-opencv-cuda 暂时取消编译
