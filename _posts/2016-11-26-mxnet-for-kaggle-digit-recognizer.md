---
title: 使用mxnet进行kaggle竞赛digit recognizer
date: 2016-11-26 21:38:48
tags:
    - 深度学习
    - mxnet

categories:
    - 深度学习
---

# 0. 编译mxnet

上一篇讲了如何编译caffe，mxnet与caffe的依赖包大体相同，只需要将mxnet依赖有源码从github上全clone下来进行编译即可。

```shell
git clone --recursive https://github.com/dmlc/mxnet
cd mxnet; cp make/osx.mk ./config.mk; make -j
```

# 1. 应用

使用mxnet的python api，快速开发一个应用算法。

```python
import numpy as np
import pandas as pd
import mxnet as mx
import logging

VALIDATION_SIZE = 2000

dataset = pd.read_csv("../input/train.csv")
target = dataset[[0]].values.ravel()
train = dataset.iloc[:,1:].values


def get_lenet():
    data = mx.symbol.Variable('data')
    # first conv
    conv1 = mx.symbol.Convolution(data=data, kernel=(5,5), num_filter=20)
    tanh1 = mx.symbol.Activation(data=conv1, act_type="tanh")
    pool1 = mx.symbol.Pooling(data=tanh1, pool_type="max",
                              kernel=(2,2), stride=(2,2))
    # second conv
    conv2 = mx.symbol.Convolution(data=pool1, kernel=(5,5), num_filter=50)
    tanh2 = mx.symbol.Activation(data=conv2, act_type="tanh")
    pool2 = mx.symbol.Pooling(data=tanh2, pool_type="max",
                              kernel=(2,2), stride=(2,2))
    # first fullc
    flatten = mx.symbol.Flatten(data=pool2)
    fc1 = mx.symbol.FullyConnected(data=flatten, num_hidden=500)
    tanh3 = mx.symbol.Activation(data=fc1, act_type="tanh")
    # second fullc
    fc2 = mx.symbol.FullyConnected(data=tanh3, num_hidden=10)
    # loss
    lenet = mx.symbol.SoftmaxOutput(data=fc2, name='softmax')
    return lenet

val_data = train[:VALIDATION_SIZE].astype('float32')
val_label = target[:VALIDATION_SIZE]
train_data = train[VALIDATION_SIZE: , :].astype('float32')
train_label = target[VALIDATION_SIZE:]
train_data = np.array(train_data).reshape((-1, 1, 28, 28))
val_data = np.array(val_data).reshape((-1, 1, 28, 28))

train_data[:] /= 256.0
val_data[:]/= 256.0

batch_size = 500
train_iter = mx.io.NDArrayIter(train_data, train_label, batch_size=batch_size, shuffle=True)
val_iter = mx.io.NDArrayIter(val_data, val_label, batch_size=batch_size)

# logging
head = '%(asctime)-15s Node[0] %(message)s'
logging.basicConfig(level=logging.DEBUG, format=head)

# create model
devs = mx.gpu(0)
network=get_lenet()
model = mx.model.FeedForward(
        #ctx                = devs,
        symbol             = network,
        num_epoch          = 4,
        learning_rate      = 0.1,
        momentum           = 0.9,
        wd                 = 0.00001,
        initializer        = mx.init.Xavier(factor_type="in", magnitude=2.34)
        )

eval_metrics = ['accuracy']
model.fit(
	X=train_iter,
	eval_metric        = eval_metrics,
	eval_data	 = val_iter
	)

#predict
test = pd.read_csv("../input/test.csv").values
test_data = test.astype('float32')
test_data = np.array(test_data).reshape((-1, 1, 28, 28))
test_data[:]/= 256.0
test_iter = mx.io.NDArrayIter(test_data, batch_size=batch_size)

pred = model.predict(X = test_iter)
pred = np.argsort(pred)
np.savetxt('submission_lenet.csv', np.c_[range(1,len(test)+1),pred[:,9]], delimiter=',', header = 'ImageId,Label', comments = '', fmt='%d')
```

# 结果

```txt
2016-11-26 21:19:07,577 Node[0] Start training with [cpu(0)]
2016-11-26 21:21:21,802 Node[0] Epoch[0] Resetting Data Iterator
2016-11-26 21:21:21,802 Node[0] Epoch[0] Time cost=134.220
2016-11-26 21:21:24,613 Node[0] Epoch[0] Validation-accuracy=0.974000
2016-11-26 21:23:36,314 Node[0] Epoch[1] Resetting Data Iterator
2016-11-26 21:23:36,314 Node[0] Epoch[1] Time cost=131.700
2016-11-26 21:23:38,900 Node[0] Epoch[1] Validation-accuracy=0.988000
2016-11-26 21:25:48,668 Node[0] Epoch[2] Resetting Data Iterator
2016-11-26 21:25:48,669 Node[0] Epoch[2] Time cost=129.769
2016-11-26 21:25:51,246 Node[0] Epoch[2] Validation-accuracy=0.987500
2016-11-26 21:28:01,337 Node[0] Epoch[3] Resetting Data Iterator
2016-11-26 21:28:01,337 Node[0] Epoch[3] Time cost=130.091
2016-11-26 21:28:03,945 Node[0] Epoch[3] Validation-accuracy=0.988000
```
提交的准确率为0.98571




