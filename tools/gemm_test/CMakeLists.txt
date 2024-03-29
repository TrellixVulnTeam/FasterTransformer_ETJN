# Copyright (c) 2021, NVIDIA CORPORATION.  All rights reserved.
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

set(encoder_gemm_files
  encoder_gemm.cc
)

set(decoding_gemm_files
  decoding_gemm.cc
)

set(gpt_gemm_files
  gpt_gemm.cc
)

add_executable(encoder_gemm ${encoder_gemm_files})
target_link_libraries(encoder_gemm PUBLIC -lcublas -lcublasLt -lcudart encoder_gemm_func encoder_igemm_func)

add_executable(decoding_gemm ${decoding_gemm_files})
target_link_libraries(decoding_gemm PUBLIC -lcublas -lcublasLt -lcudart)

add_executable(gpt_gemm ${gpt_gemm_files})
target_link_libraries(gpt_gemm PUBLIC -lcublas -lcublasLt -lcudart)