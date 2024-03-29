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

set(tf_weight_quantize_files
  weight_quantize_op.cc
)

set(tf_bert_transformer_files
  bert_transformer_op.cc
)

set(tf_fused_multihead_attention_files
  fused_multihead_attention_op.cc
)

set(tf_add_bias_transpose_files
  add_bias_transpose_op.cc
)

set(tf_decoder_files
  decoder_op.cc
)

set(tf_decoding_beamsearch_files
  decoding_beamsearch_op.cc
)

set(tf_decoding_sampling_files
  decoding_sampling_op.cc
)

set(tf_gpt_files
  gpt_op.cc
)

if(EXISTS ${TF_PATH}/libtensorflow_framework.so)
  set(tf_link
    -ltensorflow_framework
  )
elseif(EXISTS ${TF_PATH}/libtensorflow_framework.so.1)
  set(tf_link
    -l:libtensorflow_framework.so.1
  )
endif()

add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)
add_definitions(-DGOOGLE_CUDA=1)

if(EXISTS ${TF_PATH}libtensorflow_framework.so)
  set(tf_link
    -ltensorflow_framework
  )
elseif(EXISTS ${TF_PATH}libtensorflow_framework.so.1)
  set(tf_link
    -l:libtensorflow_framework.so.1
  )
endif()

add_library(tf_weight_quantize SHARED ${tf_weight_quantize_files})
target_link_libraries(tf_weight_quantize PRIVATE ${tf_link} encoder)

add_library(tf_fastertransformer SHARED ${tf_bert_transformer_files})
target_link_libraries(tf_fastertransformer PRIVATE -lcublas -lcudart ${tf_link} encoder)

add_library(tf_fused_multihead_attention SHARED ${tf_fused_multihead_attention_files})
target_link_libraries(tf_fused_multihead_attention PRIVATE -lcublas -lcudart ${tf_link} trt_fused_multi_head_attention decoder)

add_library(tf_add_bias_transpose SHARED ${tf_add_bias_transpose_files})
target_link_libraries(tf_add_bias_transpose PRIVATE -lcublas -lcudart ${tf_link} encoder)

add_library(tf_decoder SHARED ${tf_decoder_files})
target_link_libraries(tf_decoder PUBLIC -lcublas -lcudart ${tf_link} decoder encoder)

add_library(tf_decoding_beamsearch SHARED ${tf_decoding_beamsearch_files})
target_link_libraries(tf_decoding_beamsearch PRIVATE -lcublas -lcudart ${tf_link} decoder decoding)

add_library(tf_decoding_sampling SHARED ${tf_decoding_sampling_files})
target_link_libraries(tf_decoding_sampling PRIVATE -lcublas -lcudart ${tf_link} decoder decoding)

if(BUILD_GPT)
  add_library(tf_gpt SHARED ${tf_gpt_files})
  target_link_libraries(tf_gpt PRIVATE -lcublas -lcudart ${tf_link} decoder decoding)
endif()