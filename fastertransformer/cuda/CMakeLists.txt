# Copyright (c) 2019-2021, NVIDIA CORPORATION.  All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
cmake_minimum_required(VERSION 3.8)

set(encoder_kernel_files
  open_attention.cu
)

set(decoder_kernel_files
  open_decoder.cu
  masked_multihead_attention.cu
)

set(online_softmax_beamsearch_kernel_files
  online_softmax_beamsearch_kernels.cu
)

set(topk_kernel_files
  topk_kernels.cu
)

set(decoding_kernel_files
  decoding_kernels.cu
)

add_library(cuda_kernels STATIC cuda_kernels.cu)
set_property(TARGET cuda_kernels PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET cuda_kernels PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(cuda_kernels PUBLIC -lcublas -lcudart -lcurand)

add_library(attention_kernels STATIC attention_kernels.cu)
set_property(TARGET attention_kernels PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET attention_kernels PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(attention_kernels PUBLIC -lcublas -lcudart -lcurand)

add_library(transformer_kernels STATIC transformer_kernels.cu)
set_property(TARGET transformer_kernels PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET transformer_kernels PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(transformer_kernels PUBLIC -lcublas -lcudart -lcurand)

add_library(cuda_int8_kernels STATIC cuda_int8_kernels.cu)
set_property(TARGET cuda_int8_kernels PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET cuda_int8_kernels PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(cuda_int8_kernels PUBLIC -lcublas -lcudart -lcurand cuda_kernels)

add_library(encoder STATIC ${encoder_kernel_files})
set_property(TARGET encoder PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET encoder PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(encoder PUBLIC -lcublas -lcublasLt -lcudart -lcurand cuda_kernels 
    cuda_int8_kernels attention_kernels transformer_kernels 
    trt_fused_multi_head_attention encoder_gemm_func encoder_igemm_func)

add_library(decoder STATIC ${decoder_kernel_files})
set_property(TARGET decoder PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET decoder PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(decoder PUBLIC -lcublas -lcudart -lcurand cuda_kernels attention_kernels transformer_kernels nccl_utils nvtx_utils)

add_library(online_softmax_beamsearch STATIC ${online_softmax_beamsearch_kernel_files})
set_property(TARGET online_softmax_beamsearch PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET online_softmax_beamsearch PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(online_softmax_beamsearch PUBLIC -lcublas -lcudart -lcurand)

add_library(topk STATIC ${topk_kernel_files})
set_property(TARGET topk PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET topk PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(topk PUBLIC -lcublas -lcudart -lcurand)

add_library(decoding STATIC ${decoding_kernel_files})
set_property(TARGET decoding PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET decoding PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(decoding PUBLIC -lcublas -lcudart -lcurand topk online_softmax_beamsearch cuda_kernels)

if (BUILD_GPT)
	target_compile_features(cuda_kernels PRIVATE cxx_std_14)
	target_compile_features(attention_kernels PRIVATE cxx_std_14)
	target_compile_features(decoder PRIVATE cxx_std_14)
	target_compile_features(online_softmax_beamsearch PRIVATE cxx_std_14)
	target_compile_features(topk PRIVATE cxx_std_14)
endif()
