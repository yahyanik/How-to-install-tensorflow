#how to install tensorflow-GPU on Ubuntu

In this tutorial I will show you how to install tensorflow.
After spending a week and testing every available option to install the tensorflow platform, I realized that none are complete or time less (in the sense that after the new version is released, you cannot use them and they are considered outdated). 

## Preparation
I have tried to install tensorflow in a python Virtual Environment, but it seems that I cannot access it in there, but I have it outside of it anyways!! 

#Packages
You are going to need some packages that need installation before the actual tensorflow: (I assume the system is brand new)
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install python2.7-dev python3.5-dev
```
As mentioned it virtualization did not work for me, but I mention it anyways:
```
$ cd ~
$ wget https://bootstrap.pypa.io/get-pip.py
$ sudo python get-pip.py
$ sudo pip install virtualenv virtualenvwrapper
$ sudo rm -rf ~/get-pip.py ~/.cache/pip
$ echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.bashrc
$ echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
$ echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
$ source ~/.bashrc
```
Use python3 for another version:
```
$ mkvirtualenv cv -p python2
```
Now rest of the packages
```
$ pip install numpy
```
Use this for python2:
```
$ sudo apt-get install python-numpy python-dev python-pip python-wheel
```
Use this for python3:
```
$ sudo apt-get install python3-numpy python3-dev python3-pip python3-wheel
```
I did not see the following anywhere, but I received errors for them, so I mention them: use pip3 for python3 pip command
```
Pip install enum34
Pip install mock
```
###Nvidia Driver
Look for the “***” version of the driver on Nvidia website
```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update
$ sudo apt-get install nvidia-*** nvidia-modprobe
```
Reboot your system and run “nvidia-smi” and you should see a table with the version of the driver at top.
## CUDA
There are many methods to install CUDA, but I prefer to download the file from Nvidia and install it, to have the most updated version!
Go to https://developer.nvidia.com/cuda-zone
->download now, Linux, x86_64, Ubuntu, 16.8 (or later), deb(local)
Go to download directory and then do:
```
$ sudo dpkg -i cuda-repo-ubuntu1604-9-2-local_9.2.148-1_amd64.deb
```
If gives an error about key, just read it, tells you what to put and then run “dpkg” again.
```
$ sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get install cuda
```
Even if “nvcc –version” does not work, it should be fine now and you have CUDA if everything goes ok.
##CuDNN
I have installed it using the .deb file and during tensorflow compile got an error and did it using the .tar and could not test to see if it is ok, so I did both and everyone was happy!!!
Go to https://developer.nvidia.com/cudnn and using your account download the files for your CUDA. You should download :
cuDNN v*** Library for Linux
cuDNN v*** Runtime Library for Ubuntu16.04 (Deb)
cuDNN v*** Developer Library for Ubuntu16.04 (Deb)
cuDNN v*** Code Samples and User Guide for Ubuntu16.04 (Deb)
then go to download directory and do (based on the CUDA-**, the version of CUDA you installed) my version was cuda-9.2:
```
$ tar -xzvf cudnn-**-linux-x64-v7.tgz
$ sudo cp cuda/include/cudnn.h /usr/local/cuda-**/include
$ sudo cp cuda/lib64/libcudnn* /usr/local/cuda-**/lib64
$ sudo chmod a+r /usr/local/cuda-**/include/cudnn.h /usr/local/cuda-**/lib64/libcudnn*
```
Then: 
```
$ sudo dpkg -i libcudnn7_***-1+cuda**_amd64.deb
$ sudo dpkg -i libcudnn7-dev_***-1+cuda**_amd64.deb
$ sudo dpkg -i libcudnn7-doc_***-1+cuda**_amd64.deb
```
Then test the CuDNN to make sure:
```
$cp -r /usr/src/cudnn_samples_v7/ $HOME
$ cd  $HOME/cudnn_samples_v7/mnistCUDNN
$make clean && make
$ ./mnistCUDNN
```
If you see: “Test passed!” at the end you have it up and going.
TensorRT and NCCL are also available for installation and their documentation is valid in Nvidia website.
## installing Bazel
I found the following for it, should be a simple google search in anycase:
```
$ sudo apt-get install openjdk-8-jdk
$ echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
$ curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install bazel
```
## Tensorflow
If everything is good so far, you should be ok to install tensorflow with GPU support now:
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
$ sudo apt-get install git
$ git clone https://github.com/tensorflow/tensorflow
$ cd tensorflow
$ git checkout r1.**
```
You can put the version you want to install here, which changes with the new release, just look at their website for the latest. 
```
$ cd tensorflow
$ ./configure
```
Asks you for information before compile, The order and number of questions may change, but they should be pretty forward to answer, if you don’t know an answer, press ENTER for default.
My 1.10 version asked for (your answers may differ):
Python path
Python libraries path
Jemalloc support :yes
Google cloud support: no
Hadoop support: no
Amazon cloud support: no
XLA support: no
VERBS support: no
OpenCL support: no
CUDA support: yes
CUDA version: 9.2
CUDA path: /usr/local/cuda-9.2
Cudnn version: 7.2
Cudnn path: default 
tensorRT support: no
NCCL:1.3
Android support: no
```
$ bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
$ sudo pip install /tmp/tensorflow_pkg/tensorflow-***-py2-none-any.whl
```
##Validate Install
```
$ cd ~
$ python
>>> import tensorflow as tf
>>> sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
```
It should give you a hardware list and summery, (feels nice at this point to read that summery)

