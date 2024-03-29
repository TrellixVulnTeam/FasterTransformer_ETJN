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
add_subdirectory(cuda)
add_subdirectory(utils)
add_subdirectory(gemm_test)
if(BUILD_TF)
  add_subdirectory(tf_op)
endif()

if(BUILD_PYT)
  add_subdirectory(th_op)
endif()

add_subdirectory(trt_fused_multihead_attention)
add_subdirectory(triton_backend)