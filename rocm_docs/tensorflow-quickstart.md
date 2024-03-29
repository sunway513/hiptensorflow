# hipTensorflow Quickstart Guide

In this quickstart guide, we'll walk through the steps for ROCm installation, run a few Tensorflow workloads, and discuss FAQs and tips.  

## Install ROCm & Tensorflow

For basic installation instructions for ROCm and Tensorflow, please see [this doc](tensorflow-install-basic.md).

## Workloads

Now that we've got ROCm and Tensorflow installed, we'll want to clone the `tensorflow/models` repo that'll provide us with numerous useful workloads:
```
cd ~
git clone https://github.com/tensorflow/models.git
```

The following sections include the instructions for running various workloads.  They also include expected results, which may vary slightly from run to run.  

### LeNet training on MNIST data

Here are the basic instructions:  
```
cd ~/models/tutorials/image/mnist

export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0
python ./convolutional.py 
```

And here is what we expect to see:  
```
Step 0 (epoch 0.00), 165.1 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 8.0 ms
Minibatch loss: 3.232, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 7.6%
Step 200 (epoch 0.23), 8.1 ms
Minibatch loss: 3.355, learning rate: 0.010000
Minibatch error: 9.4%
Validation error: 4.4%
Step 300 (epoch 0.35), 8.1 ms
Minibatch loss: 3.147, learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 2.9%

...

Step 8500 (epoch 9.89), 7.2 ms
Minibatch loss: 1.609, learning rate: 0.006302
Minibatch error: 0.0%
Validation error: 1.0%
Test error: 0.8%
```

### CifarNet training on CIFAR-10 data

Details for this workload can be found at this [link](https://www.tensorflow.org/tutorials/deep_cnn).

Here, we'll be running two simultaneous processes from different terminals:  one for training and one for evaluation.

#### Training (via terminal #1)

Run the training:  
```
cd ~/models/tutorials/image/cifar10

export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0
python ./cifar10_train.py
```

You should see output similar to this:
```
2017-10-04 17:33:39.246053: step 0, loss = 4.66 (72.3 examples/sec; 1.770 sec/batch)
2017-10-04 17:33:39.536988: step 10, loss = 4.64 (4399.5 examples/sec; 0.029 sec/batch)
2017-10-04 17:33:39.794230: step 20, loss = 4.49 (4975.8 examples/sec; 0.026 sec/batch)
2017-10-04 17:33:40.050329: step 30, loss = 4.33 (4998.1 examples/sec; 0.026 sec/batch)
2017-10-04 17:33:40.255417: step 40, loss = 4.36 (6241.7 examples/sec; 0.021 sec/batch)
2017-10-04 17:33:40.448037: step 50, loss = 4.40 (6644.5 examples/sec; 0.019 sec/batch)
2017-10-04 17:33:40.640150: step 60, loss = 4.20 (6662.7 examples/sec; 0.019 sec/batch)
2017-10-04 17:33:40.832118: step 70, loss = 4.23 (6667.8 examples/sec; 0.019 sec/batch)
2017-10-04 17:33:41.017503: step 80, loss = 4.30 (6904.7 examples/sec; 0.019 sec/batch)
2017-10-04 17:33:41.208288: step 90, loss = 4.21 (6709.0 examples/sec; 0.019 sec/batch)
```

#### Evaluation (via terminal #2)

Note: If you have a second GPU, you can run the evaluation in parallel with the training -- to do so, just change `HIP_VISIBLE_DEVICES` to your second GPU's ID.  If you only have a single GPU, it is best to wait until training is complete, otherwise you risk running out of device memory.  

To run the evaluation, follow this:  
```
cd ~/models/tutorials/image/cifar10

export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0
python ./cifar10_eval.py
```

Using the most recent training checkpoints, this script indicates how often the top prediction matches the true label of the image.  You should see periodic output similar to this:
```
2017-10-05 18:34:40.288277: precision @ 1 = 0.118
2017-10-05 18:39:45.989197: precision @ 1 = 0.118
2017-10-05 18:44:51.644702: precision @ 1 = 0.836
2017-10-05 18:49:57.354438: precision @ 1 = 0.836
2017-10-05 18:55:02.960087: precision @ 1 = 0.856
2017-10-05 19:00:08.752611: precision @ 1 = 0.856
2017-10-05 19:05:14.307137: precision @ 1 = 0.861
...
```

### ResNet training on CIFAR-10 data

Details can be found at this [link](https://github.com/tensorflow/models/tree/master/research/resnet)

Set up the CIFAR-10 dataset
```
cd ~/models/research/resnet

curl -o cifar-10-binary.tar.gz https://www.cs.toronto.edu/~kriz/cifar-10-binary.tar.gz
tar -xzf cifar-10-binary.tar.gz
ln -s ./cifar-10-batches-bin ./cifar10
```

Train ResNet:
```
export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0
python ./resnet_main.py --train_data_path=cifar10/data_batch* \
                               --log_root=/tmp/resnet_model \
                               --train_dir=/tmp/resnet_model/train \
                               --dataset='cifar10' \
                               --num_gpus=1
```

Here are the expected results (note the `precision` metric in particular):
```
INFO:tensorflow:loss = 2.53745, step = 1, precision = 0.125
INFO:tensorflow:loss = 1.9379, step = 101, precision = 0.40625
INFO:tensorflow:loss = 1.68374, step = 201, precision = 0.421875
INFO:tensorflow:loss = 1.41583, step = 301, precision = 0.554688
INFO:tensorflow:loss = 1.37645, step = 401, precision = 0.5625
...
INFO:tensorflow:loss = 0.485584, step = 4001, precision = 0.898438
...
```

### Inception classification on ImageNet data
Details can be found at this [link]( https://github.com/ROCmSoftwarePlatform/hiptensorflow/blob/hip-amd-nccl/tensorflow/g3doc/tutorials/image_recognition/index.md)

Here's how to run the classification workload:  
```
export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0

cd models/tutorials/image/imagenet
python ./classify_image.py
```
Here are the expected results:
```
giant panda, panda, panda bear, coon bear, Ailuropoda melanoleuca (score = 0.89107)
indri, indris, Indri indri, Indri brevicaudatus (score = 0.00779)
lesser panda, red panda, panda, bear cat, cat bear, Ailurus fulgens (score = 0.00296)
custard apple (score = 0.00147)
earthstar (score = 0.00117)
```

### Soumith Convnet Benchmarks
Details on the convnet benchmarks can be found at this [link](https://github.com/soumith/convnet-benchmarks).

Here are the basic instructions:  
```
# Grab the code
cd $HOME 
git clone https://github.com/soumith/convnet-benchmarks.git
BENCHDIR=$HOME/convnet-benchmarks/tensorflow
cd hiptensorflow

# Run the benchmarks
rm -f $BENCHDIR/output_*.log
MODELS="alexnet overfeat vgg googlenet"
export HIP_VISIBLE_DEVICES=0
export HSA_ENABLE_SDMA=0
for m in $MODELS
do
    python $BENCHDIR/benchmark_${m}.py 2>&1 | tee $BENCHDIR/output_${m}.log
done

# Get a quick summary
find $BENCHDIR -name "output*.log" -print -exec grep 'across 100 steps' {} \; -exec echo \;
```

Expected result:  
```
/root//convnet-benchmarks/tensorflow/output_alexnet.log
2017-09-16 20:21:03.171504: Forward across 100 steps, 0.025 +/- 0.003 sec / batch
2017-09-16 20:21:17.314033: Forward-backward across 100 steps, 0.084 +/- 0.008 sec / batch

/root//convnet-benchmarks/tensorflow/output_googlenet.log
2017-09-16 20:24:49.543523: Forward across 100 steps, 0.100 +/- 0.010 sec / batch
2017-09-16 20:27:46.143870: Forward-backward across 100 steps, 0.328 +/- 0.033 sec / batch

/root//convnet-benchmarks/tensorflow/output_overfeat.log
2017-09-16 20:21:36.272601: Forward across 100 steps, 0.096 +/- 0.010 sec / batch
2017-09-16 20:22:18.039344: Forward-backward across 100 steps, 0.340 +/- 0.034 sec / batch

/root//convnet-benchmarks/tensorflow/output_vgg.log
2017-09-16 20:22:40.406108: Forward across 100 steps, 0.132 +/- 0.001 sec / batch
2017-09-16 20:23:30.809168: Forward-backward across 100 steps, 0.384 +/- 0.002 sec / batch
```

## FAQs & tips

### Temp workaround:  Solutions when running out of memory
As a temporary workaround, if your workload runs out of device memory, you can either reduce the batch size or set `config.gpu_options.allow_growth = True`.

### Temp workaround:  Improving numerical stability
As a temporary workaround, we currently recommend that you `export HSA_ENABLE_SDMA=0` to improve numerical stability when training Tensorflow models.  
