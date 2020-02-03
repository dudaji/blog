---
layout: post
title: Nvidia TensorRT
author: godaji
categories: general
comments: true
---

## 기본 정보

https://github.com/NVIDIA/tensorrt-inference-server r19.12 branch 


## Installing the Server


```bash
# sudo apt install software-properties-common autoconf \
#  automake build-essential cmake git libgoogle-glog0v5 libre2-dev \
#  libssl-dev libtool libboost-dev libcurl4-openssl-dev zlib1g-dev

cmake \
-DTRTIS_ENABLE_TENSORRT=OFF \
-DTRTIS_ENABLE_TENSORFLOW=ON \
-DTRTIS_ENABLE_ONNXRUNTIME=OFF \
-DTRTIS_ENABLE_PYTORCH=OFF \
-DTRTIS_ENABLE_CAFFE2=OFF \
-DTRTIS_ENABLE_CUSTOM=OFF \
-DTRTIS_ENABLE_GPU=OFF \
../build

make -j16 trtis
```

<Code 1> Install CMake script

#ifdef를 이용해서 GPU, Backend를 ON/OFF할 수 있게 되어 있습니다.  
예를 들어 DTRTIS_ENABLE_TENSORFLOW와 DTRTIS_ENABLE_GPU가 ON이 되어 있으면  
Backend로 TF를 사용하면서 GPU를 이용하는 버전으로 컴파일한다는 의미가 됩니다.

## Model Repository

모델 저장소는 아래와 같습니다.  
trtserver를 실행시킬 때, model-repository-path를 지정하게 되는데  
 폴더 아래에 존재하는 모델들을 실행할 때 읽어오는 형태로 되어 있습니다.

```bash
<model-repository-path>/
  model_0/
    config.pbtxt
    output0_labels.txt
    1/
      model.plan
    2/
      model.plan
  inception_graphdef/
    config.pbtxt
    inception_labels.txt
    1/
      model.graphdef
```

<Code 2> model-repository-path

```protobuf
name: "inception_graphdef"
platform: "tensorflow_graphdef"
max_batch_size: 128
input [
 {
   name: "input"
   data_type: TYPE_FP32
   format: FORMAT_NHWC
   dims: [ 299, 299, 3 ]
 }
]
output [
 {
   name: "InceptionV3/Predictions/Softmax"
   data_type: TYPE_FP32
   dims: [ 1001 ]
   label_filename: "inception_labels.txt"
 }
]
```

<Code 3> config.pbtxt

model_0, inception_graphdef 두개의 모델을 로드합니다.  
config.pbtxt는 필수 입니다, config.pbtxt안에 platform에 따라서 읽어오는 모델이 달라집니다.  
즉 platform 마다 넣어주어야 하는 파일이 달라집니다.  
예를 들어서 platform이 tensorflow_graphdef라면 model.graphdef가 필요합니다.  
만약 platform이 onnx라면 model.onnx가 있어야 합니다.  
config.pbtxt안에 name과 모델 폴더 이름은 같아야 합니다.  
model.graphdef이름은 정해져 있는 이름입니다.  
config.pbtxt안에 버전 정보를 넣으면 원하는 버전의 모델을 읽을 수 있습니다.  
특별히 없으면 1 폴더 아래의 model.graphdef를 읽게 됨

## BackendContext의 의미

위 코드의 예로 보면 model_1, inception_graphdef 두개의 모델이 로딩됩니다.  
model_1은 tensorRT backend의 context로 뜨게 되고 inception_graphdef는  
tensorflow backend의 context로 뜨게 됩니다.  
만약 같은 TF 플랫폼에 모델이 N개라면 그 TF Backend에 N개의 BackendContext가 생기게 됩니다.

## Input converting
HTTP / gRPC로 오는 것과 상관없이 스케줄러에서 BackendContext::Run을 호출하여 추론을 할 때에는  
vector<Scheduler::Payload>로 변환됩니다.  

## 추론 핵심 함수

```c++
// src/backends/tensorflow/base_backend.cc:763

Status
BaseBackend::Context::Run(
    const InferenceBackend* base, 
    std::vector<Scheduler::Payload>* payloads)
{
  //...
  {
    TRTISTF_TensorList* rtl;
    RETURN_IF_TRTISTF_ERROR(TRTISTF_ModelRun(
        trtistf_model_.get(), *(input_tensors.release()),
        required_outputs.size(), output_names_cstr, &rtl));
    output_tensors.reset(rtl);
  }
  //...
}
```

core의 BackendContext::Run을 구현.

## GPU vs CPU

BackendContext::cudaStream_t stream_가 enable / disable되느냐에 따라서 CPU / GPU 사용 여부를 결정하고 있음.  
(nvidia에서 제공하는 cuda_runtime_api.h)  
Run 함수 내에서 아래와 같이 cuda를 쓸것인지 아닌지에 대해서 의사결정하고 있습니다.  

```c++
// src/backends/tensorflow/base_backend.cc:859


#ifdef TRTIS_ENABLE_GPU
  if (cuda_copy) {
    cudaStreamSynchronize(stream_);
  }
#endif  // TRTIS_ENABLE_GPU
```
