---
layout: post
title: "Elastic stack"
author: bubuta
categories: general
comments: true
---

## On Kubernetes

helm chart 중에 elastic stack 이 있어서 한번 설치해 본 적이 있습니다.  
설치 후 kibana에서 모든 Container 로그들을 볼 수 있었습니다.  
namespace, label, pod 이름등 kubernetes concept에 있는 것들로 filtering도 가능 했습니다.  

## On mesos marathon

사내 프로젝트 중에 [mesos](https://github.com/apache/mesos) 및 [marathon](https://github.com/mesosphere/marathon) 을 사용 하는 프로젝트가 있었는데  
여기에는 helm으로 elastic stack을 설치할 수가 없어서 어떻게 작동하는지 살펴보았습니다.  

elastic stack 중에
1. elastic search
2. kibana

두가지는 marathon에 띄워도 크게 변경하는 사항이 없었습니다.

fluentd의 경우는 달랐습니다.

[fluentd configuration line #150](https://github.com/helm/charts/blob/c143a86211533e432716e1b4fb1e73cce306857a/stable/fluentd-elasticsearch/values.yaml#L150) fluentd-elasticsearch chart 에서 comment 부분을 보시면  

> The Kubernetes kubelet makes a symbolic link to this file on the host machine  
> in the /var/log/containers directory which includes the pod name and the Kubernetes  
> container name:  
> synthetic-logger-0.25lps-pod_default_synth-lgr-997599971ee6366d4a5920d25b79286ad45ff37a74494f262e3bc98d909d0a7b.log  
> ->
> /var/lib/docker/containers/997599971ee6366d4a5920d25b79286ad45ff37a74494f262e3bc98d909d0a7b/997599971ee6366d4a5920d25b79286ad45ff37a74494f262e3bc98d909d0a7b-json.log

kubelet이 /var/lib/docker/containers 에 있는 log file에 대한 symbolic link를 /var/log/containers directory에  
만든다고 나와있습니다. 예제를 보시면 pod 이름 같은 metadata 정보도 symbolic link에 추가해 주고 있습니다.  

marathon에 fluentd 들 띄울때는 configuration 을 아래와 같이 했습니다.
```
@type tail
path /var/lib/docker/containers/*/*-json.log
```

이렇게 했을때 log file 에는 log, stream, time 정도 정보만 있어서 수집된 log들은 있어도 이게 어떤 container의 로그인지 알기 어려웠습니다.

```json
{"log":"Traceback (most recent call last):\n","stream":"stderr","time":"2019-10-25T02:14:51.347722716Z"}
```

그래서 marathon에 container를 띄울때 아래와 같은 docker run option을 주었고
```bash
--log-opt labels=elastic-stack --label elastic-stack=fluentd
```

log file에 attrs 라는 field가 생겨서 이 field로 filtering이 가능해졌습니다.
```json
{"log":"  2020-01-31 00:55:37 +0000 [warn]: suppressed same stacktrace\n","stream":"stdout","attrs":{"elastic-stack":"fluentd"},"time":"2020-01-31T00:55:37.720439474Z"}
```
