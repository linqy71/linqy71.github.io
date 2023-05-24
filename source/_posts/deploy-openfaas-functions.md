---
title: k8s+openfaas运行自定义函数--tfserving prediction
date: 2023-05-23 14:59:12
tags: k8s openfaas tf-serving
categories: 操作记录
copyright: true
---

> 基于k8s部署openfaas,并启动tensorflow-serving函数作为服务响应请求

# 部署k8s

[用kubeadm部署k8s并检测GPU](https://linqy71.github.io/2023/03/26/deploy-k8s-with-kubeadm/)

上一篇博客记录了部署过程，在其基础上进行下一步。

# 部署openfaas

参考[openfaas部署](https://coldtea214.gitbook.io/cncf-serverless/installableplatform/openfaas)

```
#安装client
$ curl -Lo faas-cli https://github.com/openfaas/faas-cli/releases/download/0.12.8/faas-cli \
    && chmod +x faas-cli && sudo mv faas-cli /usr/local/bin/

#安装server
$ cd faas-netes  #进入faas-netes 仓库
$ kubectl apply -f namespaces.yml  #创建2个namespace
$ kubectl -n openfaas create secret generic basic-auth \
    --from-literal=basic-auth-user=admin \
    --from-literal=basic-auth-password="admin"
$ kubectl apply -f ./yaml/


#确认所有组件正常运行
$ kubectl get pods -n openfaas

NAME                            READY   STATUS    RESTARTS        AGE
alertmanager-67f5ff98d4-lpr5f   1/1     Running   0               2d21h
gateway-6f47b5c9b4-5szwn        2/2     Running   1 (2d21h ago)   2d21h
nats-85dc667985-drw9x           1/1     Running   0               2d21h
prometheus-6b76879cd6-x5z7m     1/1     Running   0               2d21h
queue-worker-648c8c8c6f-vvj6x   1/1     Running   0               2d21h

```

成功运行后可继续参考[openfaas部署](https://coldtea214.gitbook.io/cncf-serverless/installableplatform/openfaas)写一个基础go函数或参考[First Python Function](https://docs.openfaas.com/tutorials/first-python-function/)写一个基础python函数


# 编写函数模板

由于faas-cli提供的模板库有限，需要自行编写运行tfserving镜像的模板。
1. 仿照python模板，创建tfserving模板文件夹
```
.
├── template
│   ├── python
│   │   ├── Dockerfile
│   │   ├── function
│   │   ├── index.py
│   │   ├── requirements.txt
│   │   └── template.yml
│   └── tfserving
│       ├── Dockerfile
│       ├── entrypoint.sh
│       ├── function
│       ├── index.py
│       ├── requirements.txt
│       ├── serving_models
│       └── template.yml
```
2. template/tfserving/Dockerfile内容如下
```
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/of-watchdog:0.9.10 as watchdog
FROM --platform=${TARGETPLATFORM:-linux/amd64} tensorflow/serving:latest

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

###安装python3###
RUN apt-get -qy update
RUN apt-get install -y libgl1-mesa-dev libglib2.0-dev wget
RUN yes | apt-get install python3  python3-pip
RUN ln -s /usr/bin/python3 /usr/bin/python
###安装python3###

# Add non root user
RUN addgroup --system app && adduser app --system --ingroup app
RUN chown app /home/app

USER app

ENV PATH=$PATH:/home/app/.local/bin

WORKDIR /home/app/

COPY --chown=app:app index.py           .
COPY --chown=app:app requirements.txt   .

USER root
RUN pip3 install --upgrade pip
RUN pip3 install --no-cache-dir -r requirements.txt  -i https://pypi.tuna.tsinghua.edu.cn/simple/

# Build the function directory and install any user-specified components
USER app

RUN mkdir -p function
RUN touch ./function/__init__.py

USER root
WORKDIR /home/app/function/
COPY --chown=app:app function/requirements.txt  .
RUN pip3 install --no-cache-dir --verbose -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/

#下载一张图片作为模型的input
RUN wget https://img-blog.csdnimg.cn/575dd33e3cc74ad7bd4ebed9d4f68362.png#pic_center
RUN mv *.png test.png
RUN cp ./test.png ../

#install function code
USER root
COPY --chown=app:app function/   .
ARG MODEL

# FROM build as ship
WORKDIR /home/app/

###复制模型镜像###
COPY ./serving_models/$MODEL /models/$MODEL
ENV MODEL_NAME=$MODEL
###复制模型镜像###

###复制entrypoint脚本，启动tfserving的关键###
COPY ./entrypoint.sh ./entrypoint.sh
RUN chmod +x ./entrypoint.sh
###复制entrypoint脚本###

#configure WSGI server and healthcheck
# USER app
ENV fprocess="python index.py"

ENV cgi_headers="true"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:5000"

ENTRYPOINT ["./entrypoint.sh"]
```
3. entrypoint.sh内容如下：
```
#!/bin/bash
#启动tensorflow-serving
/usr/bin/tf_serving_entrypoint.sh & 
 
fwatchdog
```
其中/usr/bin/tf_serving_entrypoint.sh脚本是拉取的tensorflow/serving:latest镜像自带的，具体内容为启动tensorflow_model_server监听：
```
tensorflow_model_server --port=8600 --rest_api_port=8601 --model_name=${MODEL_NAME} --model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME} "$@"
```

4. requirements.txt
```
flask
waitress
tox==3.*
```
5. template.yml
```
language: tfserving #定义language类型，用于创建函数
fprocess: python3 index.py
```
6. index.py
```
# Copyright (c) Alex Ellis 2017. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for full license information.

from flask import Flask, request
from function import handler
from waitress import serve
import os

app = Flask(__name__)

# or set a default timeout
app.config['TIMEOUT'] = 60

# distutils.util.strtobool() can throw an exception
def is_true(val):
    return len(val) > 0 and val.lower() == "true" or val == "1"

@app.before_request
def fix_transfer_encoding():
    """
    Sets the "wsgi.input_terminated" environment flag, thus enabling
    Werkzeug to pass chunked requests as streams.  The gunicorn server
    should set this, but it's not yet been implemented.
    """

    transfer_encoding = request.headers.get("Transfer-Encoding", None)
    if transfer_encoding == u"chunked":
        request.environ["wsgi.input_terminated"] = True

@app.route("/", defaults={"path": ""}, methods=["POST", "GET"])
@app.route("/<path:path>", methods=["POST", "GET"])
def main_route(path):
    raw_body = os.getenv("RAW_BODY", "false")

    as_text = True

    if is_true(raw_body):
        as_text = False

    ret = handler.handle(request.get_data(as_text=as_text))
    return ret

if __name__ == '__main__':
    serve(app, host='0.0.0.0', port=5000)
```

7. function文件夹下的内容与python模板中的function文件夹相同
8. serving_models文件夹用于存放后续tensorflow_model_server要运行的模型

# 根据模板创建函数并调用

1. 创建新函数resnet152
```
faas-cli new --lang tfserving resnet152
```
2. 确保resnet152模型存放在serving_models文件夹中
```
cp -r ./serving_models/resnet152 ./template/tfserving/serving_models/
```
3. 修改resnet152/handler.py
```
import time
def handle(req):
  \"\"\"handle a request to the function
  Args:
      req (str): request body
  \"\"\"

  err = \"$func_name: error nont found\"
  a = \"\"
  import os
  import cv2
  import numpy as np
  import  requests
  import json
  url = 'http://127.0.0.1:8501/v1/models/$model:predict'
  image = cv2.imread(\"test.png\")
  image = cv2.resize(image, (1024, 1024))
  image = image.astype(np.uint8) / 255
  image = np.uint8(image)
  image = image.tolist()
  headers = {\"content-type\": \"application/json\"}
  body = {
          \"signature_name\": \"serving_default\",
          \"inputs\": [
          image
          ]
          }
  beginTime = time.perf_counter()
  r = requests.post(url, data = json.dumps(body), headers = headers)
  endTime = time.perf_counter()
  totalTime = (endTime-beginTime)*1000
  req += \"耗时%f毫秒\"%totalTime+\"\n\"
  print(req)
  return req
```

4. 添加环境依赖
```
echo "opencv-python
requests" > ./resnet152/requirements.txt
```

5. 修改yml文件
```
echo "version: 1.0
provider:
  name: openfaas
  gateway: http://127.0.0.1:31112
functions:
  resnet152:
    lang: resnet152
    handler: ./resnet152
    image: username/resnet152:latest #此处username是Dockerhub用户名
" > ./resnet152.yml
```

6. build函数
```
faas-cli build  -f ./resnet152.yml --build-arg MODEL=resnet152
```

7. push镜像到dockerhub
```
#确保docker login过了
sudo docker push username/resnet152:latest
```

8. 部署函数（这一步会从dockerhub拉取镜像）
```
faas-cli deploy -f ./resnet152.yml
```

9. 等待容器成功运行
```
kubectl get pods -n openfaas-fn

NAME                            READY   STATUS    RESTARTS   AGE
resnet152-c6b687d55-hlkp2       1/1     Running   0          4h41m
```

10. 调用函数
```
faas-cli invoke resnet152 -g 127.0.0.1:31112 < empty_input.txt 
#由于input使用的是提前下载好的图片，所以这里采用了empty input，直接执行模型推理
```

函数调用没有成功返回时，可以进入容器排查错误：
```
kubectl exec -it resnet152-c6b687d55-hlkp2 -n openfaas-fn /bin/bash
```


