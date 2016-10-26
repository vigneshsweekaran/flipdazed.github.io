---
layout: post
title: "Theano on Amazon Web Services for Deep Learning"
menutitle: "Theano on Amazon Web Services (EC2) for Deep Learning"
date: 2016-10-25 22:31:00 +0100
tags: Theano Amazon Web Services Deep Learning
category: Deep Learning
author: am
published: true
redirect_from: "/2016-10-25-theano-on-amazon-web-services-for-deep-learning/"
language: EN
comments: true
---

This is the second part of a two-part guide

1. [Set Up Amazon Elastic Compute Cloud (EC2)]({% post_url 2016-10-25-set-up-amazon-ec2 %})
2.  [Theano on Amazon Web Services for Deep Learning]({%post_url 2016-10-25-theano-on-amazon-web-services-for-deep-learning %})

This entry demonstrates how you can offload computational tasks to an Amazon Elastic 
Compute Cloud (EC2) instance through [Amazon Web Services (AWS)][1]. The guide focuses on
 CUDA support for [`Theano`][2].

# Requirements

 - Can set up an EC2 Instance - [see part one]({% post_url 2016-10-25-theano-on-amazon-web-services-for-deep-learning %})
 - Familiarity with Linux and Bash e.g. `sudo`, `wget`, `export`
 - Familiarity with [Ubuntu][3] for `apt-get`

# Contents
 1. [Connect](#connect)
 2. [Load Software](#load)
 3. [Run Code](#run)
 4. [Close Instance](#close)

<a id="connect"></a>

# Connect

Connect to the Instance through SSH. Assuming you followed part 1 this is just
<pre class="line-numbers language-bash"><code>ssh ubuntu@[DNS]</code></pre>

<a id="load"></a>

# Load Software
See the references for the sources of these instructions. This code is almost identical with a few 
tweaks.

**Note** you will have to do this each time you start an Instance

<pre class="line-numbers language-bash"><code># update software
sudo apt-get update
sudo apt-get -y dist-upgrade

# install dependencies
sudo apt-get install -y gcc g++ gfortran build-essential \
    git wget linux-image-generic libopenblas-dev \
    python-dev python-pip ipython python-nose\
    python-numpy python-scipy

# install bleeding edge theano
sudo pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git

# get CUDA
sudo wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.0-28_amd64.deb

# depackage and install CUDA
sudo dpkg -i cuda-repo-ubuntu1404_7.0-28_amd64.deb
sudo apt-get update
sudo apt-get install -y cuda

# update PATH variables
{
    echo -e "\nexport PATH=/usr/local/cuda/bin:$PATH"
    echo -e "\nexport LD_LIBRARY_PATH=/usr/local/cuda/lib64"
} >> .bashrc

# reboot for CUDA
sudo reboot
</code></pre>

After waiting about a minute for the reboot, `ssh` back into the Instance

<pre class="line-numbers language-bash"><code># install included samples and test cuda
ver=$(echo ${f%%.sh} | tail -c -4) # get version number
echo "CUDA version: ${ver}"
/usr/local/cuda/bin/cuda-install-samples-${ver}.sh ~/
cd NVIDIA\_CUDA-${ver}\_Samples/1\_Utilities/deviceQuery
make
./deviceQuery
</code></pre>

Make sure the test shows that a GPU exists. If none exists you will receive the following output

<pre class="language-terminal"><code>./deviceQuery Starting...

CUDA Device Query (Runtime API) version (CUDART static linking)

NVIDIA: no NVIDIA devices found
cudaGetDeviceCount returned 30
-> unknown error
Result = FAIL
ubuntu@ip-172-31-36-215:~/NVIDIA_CUDA-8.0_Samples/1_Utilities/deviceQuery$ ./deviceQuery 
deviceQuery        deviceQuery.cpp    deviceQuery.o      Makefile           NsightEclipse.xml  readme.txt         
ubuntu@ip-172-31-36-215:~/NVIDIA_CUDA-8.0_Samples/1_Utilities/deviceQuery$ ./deviceQuery 
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)
 
NVIDIA: no NVIDIA devices found
cudaGetDeviceCount returned 30
-> unknown error
Result = FAIL</code></pre>

If you don't have a GPU then skip the next step or use a GPU EC2 Instance

<pre class="line-numbers language-bash"><code>#  set up the theano config file to use gpu by default
{
    echo -e "\n[global]\nfloatX=float32\ndevice=gpu"
    echo -e "[mode]=FAST_RUN"
    echo -e "\n[nvcc]"
    echo -e "fastmath=True"
    echo -e "\n[cuda]"
    echo -e "root=/usr/local/cuda" 
}>> ~/.theanorc</code></pre>

Install any other dependencies you may require

<a id="run">

# Run Code
Transfer the relevant code across the the Cloud e.g.

 - Pull from an existing `git` repository
 - `scp` files across
 
If you are running code in a Spot Instance, I would recommend saving results at runtime
and passing them back to your local machine. It is sensible to `pickle` the state of the neural net
at runtime so that you can easily continue the training process from a saved state rather than
having to run again from scratch.

<a id="close"></a>

# Close
Don't forget to Stop or Close the instance once it has completed the task! 
This can be automated by running the following as root after code has been executed
<pre class="line-numbers language-bash"><code>shutdown -h now</code></pre>

# References
 - [Amazon Web Services][1]
 - [`theano` Docs][2]
 - [Ubuntu][3]
 - [Installing CUDA, OpenCL, and PyOpenCL on AWS EC2][4]
 - [How to install Theano on Amazon EC2 GPU instances for deep learning][5]
- [StackOverflow: Self-Terminating AWS EC2 Instance][6]

[1]: https://aws.amazon.com
[2]: http://theano.readthedocs.io
[3]: https://www.ubuntu.com
[4]: http://vasir.net/blog/opencl/installing-cuda-opencl-pyopencl-on-aws-ec2
[5]: http://markus.com/install-theano-on-aws/
[6]: http://stackoverflow.com/a/10542456/4013571