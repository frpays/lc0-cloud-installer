# Lc0 Google Cloud installer

[Lc0](https://github.com/LeelaChessZero/lc0) will be installed on Ubuntu and take advantage of up to 8 GPUs. This installer is derived/upgraded from the older [LC0 Google Cloud guide](http://lczero.org/dev/wiki/google-cloud-guide-lc0/) to support Leela 0.25.

## 1. Setup a Google cloud account 

Visit [Google Cloud](https://console.cloud.google.com/) and setup an account.
Note that free trial accounts are offered free credits ($300 at the time of writing).

## 2. Create a project and a new instance

Create a new project and [install a project-wide ssh key in the metadata](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys?hl=en#project-wide).

Create a new compute engine instance that will host the chess engine.

Keep it small during the installation:

- 1 vcpu, 3.75Gb memory, (we will bump this later),
- 20 Gb of disk,
- Ubuntu 18.04LTS,
- No GPU (for the moment).

Start you instance and login.

*Note:* You may be billed for this instance (about $0.034 hourly).


## 3. Launch the provided installer

The installation will take about 15 minutes. 

```
$ git clone https://github.com/frpays/lc0-cloud-installer.git

$ cd lc0-cloud-installer

$ ./installer

Installing CUDA
--2020-06-06 16:58:00--  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.2.89-1_amd64.deb
Resolving developer.download.nvidia.com (developer.download.nvidia.com)... 152.195.19.142
Connecting to developer.download.nvidia.com (developer.download.nvidia.com)|152.195.19.142|:443... connected.
(...)
Saving to: ‘256x20-t40-1541.pb.gz’

256x20-t40-1541.pb.gz           100%[=====================================================>]  42.24M   483KB/s    in 92s     

2020-06-06 17:35:33 (471 KB/s) - ‘256x20-t40-1541.pb.gz’ saved [44289015/44289015]
````
CUDA and CUDNN are installed and Lc0 is built from the latest tag.
The installer will also download a 256x20b network (256x20-t40-1541.pb.gz from https://lczero.org/play/networks/bestnets/).

When the installer completed, you can optionaly test your engine.
Without GPU, the engine will fallback to CPU.

```
~$ ./engine 
       _
|   _ | |
|_ |_ |_| v0.25.1+git.69105b4 built Jun  6 2020
go
Loading weights file from: /home/frpays/weights/default
Creating backend [eigen]...
Using Eigen version 3.3.5
Eigen max batch size is 256.
info depth 1 seldepth 2 time 1956 nodes 3 score cp 15 nps 3 tbhits 0 pv e2e4 e7e5
info depth 2 seldepth 3 time 2879 nodes 5 score cp 14 nps 2 tbhits 0 pv e2e4 e7e5 g1f3
(...)
```


## 4. Add GPU devices to your instance

Stop you instance and edit the characteritics.

* Add as many GPU you want, up to 8 V100. You will need to increase quotas.
* You will have to increase the number of vcpus. Make sure you have 2 vcpus per GPU.
* You will need about 20Gb of memory. Make it 32gb to be confortable.

*Caution*: every GPU makes the instance *considerably more expensive.*
Each V100 is billed about $2.5 per hour at the time of writing.

See [the Google Gloud GPU pricing](https://cloud.google.com/compute/gpus-pricing) for details.
Preemptive instances are less expensive, but are not suitable for the task as they can be stopped at any moment.

**Don't forget to stop your instance when you finished!**


## 5. Test your instance

The `engine` script will automatically use all your GPUs, (or fallback to CPU if none).

```
$ ~/engine

|   _ | |
|_ |_ |_| v0.25.1+git.69105b4 built Jun  6 2020
go
Loading weights file from: /home/frpays/weights/default
Creating backend [cudnn-fp16]...
CUDA Runtime version: 10.2.0
Cudnn version: 7.6.5
Latest version of CUDA supported by the driver: 11.0.0
GPU: Tesla V100-SXM2-16GB
GPU memory: 15.7817 Gb
GPU clock frequency: 1530 MHz
GPU compute capability: 7.0
info depth 1 seldepth 2 time 2208 nodes 3 score cp 15 nps 272 tbhits 0 pv e2e4 e7e5
(...)
info depth 22 seldepth 62 time 29845 nodes 1485568 score cp 11 nps 53922 tbhits 0 pv e2e4 e7e5 g1f3 b8c6 f1b5 g8f6 e1g1 f6e4 d2d4 e4d6 b5c6 d7c6 d4e5 d6f5 d1d8 e8d8 b1c3 f8e7 h2h3 f5h4 f1d1 d8e8 f3h4 e7h4 g2g4 h7h5 f2f3 e8e7 c1f4 b7b6 g1g2 e7e6 a2a4 a7a5 c3e2 c6c5 f4g3 h4g3 g2g3 h5h4 g3f4 f7f6 e5f6 e6f6 e2c3
```

## 6. Connect remotely to your engine from your laptop

Although this is not required, it's highly recommended to [setup a static address](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address?hl=en) on the instance.
The static address is free, as long as it is attached to an instance. 
The instance is allowed to keep its static address freely, even stopped.

### 6.1 MacOS and Linux

On you Mac or Linux box, create a forwarding script to your cloud instance.

Create a new file script `remove_engine`. You will need:
- your (possibly static) IP address,
- your account name,
- your ssh identity.

``` 
#!/bin/bash
ssh -i ~/.ssh/google-compute $frpays@35.221.231.255 /home/frpays/engine
```

Make you script executable and test it.

```
$ chmod ug+rx remote_engine
$ ./remote_engine

       _
|   _ | |
|_ |_ |_| v0.25.1+git.69105b4 built Jun  6 2020
go
Loading weights file from: /home/frpays/weights/default
Creating backend [multiplexing]...
Creating backend [cudnn-fp16]...
CUDA Runtime version: 10.2.0
Cudnn version: 7.6.5
Latest version of CUDA supported by the driver: 11.0.0
GPU: Tesla V100-SXM2-16GB
GPU memory: 15.7817 Gb
GPU clock frequency: 1530 MHz
GPU compute capability: 7.0
Creating backend [cudnn-fp16]...
CUDA Runtime version: 10.2.0
Cudnn version: 7.6.5
Latest version of CUDA supported by the driver: 11.0.0
GPU: Tesla V100-SXM2-16GB
GPU memory: 15.7817 Gb
GPU clock frequency: 1530 MHz
GPU compute capability: 7.0
info depth 1 seldepth 2 time 20285 nodes 3 score cp 15 nps 375 tbhits 0 pv e2e4 e7e5
info depth 2 seldepth 3 time 20289 nodes 5 score cp 14 nps 416 tbhits 0 pv e2e4 e7e5 g1f3
(...)
info depth 18 seldepth 56 time 7671 nodes 250858 score cp 11 nps 62155 tbhits 0 pv e2e4 e7e5 g1f3 b8c6 f1b5 g8f6 e1g1 f6e4 d2d4 e4d6 b5c6 d7c6 d4e5 d6f5 d1d8 e8d8 h2h3 f8e7 b1c3 f5h4 f3h4 e7h4 f1d1 d8e8 g2g4 a7a5 a2a4 b7b6 c1f4 e8e7 g1g2 h7h5 f2f3 e7e6
```

### 6.2 Windows

Use PuTTY to create you script, as explained here:

http://komodochess.com/remote-engine.htm


### 7. Setup you remote engine in your favorite UCI client

Select the `remove_engine` script that you created.
The UCI engine will recognize it as a chess engine.


### Appendix:  Installed software versions

* Ubuntu 18.04LTS
* CUDA 10.2
* Cudnn 7.6.5
* Leela v0.25.1


