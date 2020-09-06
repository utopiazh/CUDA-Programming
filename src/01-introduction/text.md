Note: I am writing a simplified version of my Chinese CUDA book in English. This is the first chapter.

# Chapter 1: GPU and CUDA 

## Introduction to GPU 

GPU means graphics processing unit, which is usually compared to CPU (central processing unit). While a typical CPU has a few relatively fast cores, a typical GPU has hundreds or thousands of relatively slow cores. In a CPU, more transistors are devoted to cache and control; in a GPU, more transistors are devoted to data processing. 

GPU computing is heterogeneous computing, which involves both CPU and GPU, which are usually referred to as host and device, respectively. Both CPU and non-embedded GPU have their own DRAM（dynamic random-access memory), and they are usually connected by a PCIe（peripheral component interconnect express）bus.

We only consider GPUs from Nvidia, becasue CUDA programming only supports these GPUs. There are a few series of Nvidia GPUs:
* Tesla series: good for scientific computing but expensive.
* GeForce series: cheaper but less professional. 
* Quadro series: kind of between the above two.
* Jetson series: embeded device (I have never used these).

Every GPU has a version number `X.Y` to indicate its **compute capability**. Here, `X` is a major version number, and `Y` is a minor version number. A major version number corresponds to a major GPU architecture and is also named after a famous scientist. See the following table.

| Major compute capability  | architecture name |   release year  |
|:------------|:---------------|:--------------|
| `X=1` | Tesla | 2006 |
| `X=2` | Fermi | 2010 |
| `X=3` | Kepler | 2012 |
| `X=5` | Maxwell | 2014 |
| `X=6` | Pascal | 2016 |
| `X=7` | Volta | 2017 |
| `X.Y=7.5` | Turing | 2018 |
| `X=8` | Ampere | 2020 |

GPUs older than Pascal will become deprecated soon. We will focus on GPUs no older than Pascal.

The computate capability of a GPU is not directly related to its performance. The following table lists the major metrics regarding performance for a few selected GPUs.

| GPU  | compute capability |  memory capacity  |  memory bandwidth  |  double-precision peak FLOPs | single-precision peak FLOPs |
|:------------|:---------------|:--------------|:-----------------|:------------|:------------------|
| Tesla P100         | 6.0 | 16 GB | 732 GB/s | 4.7 FLOPs | 9.3 FLOPs|
| Tesla V100         | 7.0 | 32 GB | 900 GB/s | 7 FLOPs  | 14 FLOPs |
| GeForce RTX 2070   | 7.5 | 8 GB  | 448 GB/s | 0.2 FLOPs| 6.5 FLOPs|
| GeForce RTX 2080ti | 7.5 | 11 GB | 732 GB/s | 0.4 FLOPs| 13 FLOPs|

We notice that the double precision performane of a GeForce GPU is only 1/32 of its single-precision performance.


## Introduction to CUDA 

There are a few tools for GPU computing, including CUDA, OpenCL, and OpenACC, but we only consider CUDA in this book. We also only consider CUDA based on C++, which is called CUDA C++ for short. We will not consider CUDA Fortran.

CUDA provides two APIs Application Programming Interfaces) for devolopers: the CUDA driver API and the CUDA runtime API. The CUDA driver API is more fundamental (low-level) and more flexible. The CUDA runtime API is constructed based on top of the CUDA driver API and is easier to use. We only consider the CUDA runtime API.

There are also many CUDA versions, which can also be represented as `X.Y`. The following table lists the recent CUDA versions and the  the supported compute capabilites.

| CUDA versions | supported GPUs |
|:------------|:---------------|
|CUDA 11.0 |  Compute capability 3.5-8.0 (Kepler to Ampere) |
|CUDA 10.0-10.2 | Compute capability 3.0-7.5 (Kepler to Turing) |
|CUDA 9.0-9.2 | Compute capability 3.0-7.2  (Kepler to Volta) | 
|CUDA 8.0     | Compute capability 2.0-6.2  (Fermi to Pascal) | 

\section{CUDA~开发环境搭建示例}

下面叙述作者最近在装有~GeForce RTX 2070~的便携式计算机（以下简称计算机）中搭建~CUDA~开发环境的大致过程。因为作者的计算机中预装了~Windows 10~操作系统，所以我们以~Windows 10~操作系统为例进行讲解。因为~Linux~发行版有多种，故本书不列出在~Linux~中安装~CUDA~开发环境的步骤。读者可参阅~Nvidia~的官方文档：\url{https://docs.nvidia.com/cuda/cuda-installation-guide-linux}。

我们说过，GPU~计算实际上是~CPU+GPU（主机+设备）的异构计算。在~CUDA C++~程序中，既有运行于主机的代码，也有运行于设备的代码。其中，运行于主机的代码需要由主机的~C++~编译器编译和链接。所以，除了安装~CUDA~工具箱，还需要安装一个主机的~C++~编译器。在~Windows~中，最常用的~C++~编译器是~Microsoft Visual C++ （MSVC），它目前集成在~Visual Studio~中，所以我们首先安装~Visual Studio。作者安装了最高版本的~Visual Studio 2019 16.x。因为这是个人使用的，故选择了免费的~Community~版本。下载地址为
~\url{https://visualstudio.microsoft.com/free-developer-offers/}。对于~CUDA C++程序开发来说，只需要选择安装~Desktop development with C++~即可。当然，读者也可以选择安装更多的组件。

关于~CUDA，作者选择安装~2019~年8月发布的~CUDA Toolkit 10.1 update2。首先，进入网址~\url{https://developer.nvidia.com/cuda-10.1-download-archive-update2}。然后根据提示，做如下选择：Operating System~项选择~Windows；Architecture~项选择~\verb"x86_64"；Version~项选择操作系统版本，我们这里是~10；Installer Type~项可以选择~exe (network)~或者~exe (local)，分别代表一边下载一边安装和下载完毕后安装。接着，运行安装程序，根据提示一步一步安装即可。该版本的~CUDA~工具箱包含一个对应版本的~Nvidia driver，故不需要再单独安装~Nvidia driver。

安装好~Visual Studio~和~CUDA~后，进入到如下目录（读者如果找不到~C~盘下的~ProgramData~目录，可能是因为没有选择显示一些隐藏的文件）：
\begin{small}
\begin{verbatim}
    C:\ProgramData\NVIDIA Corporation\CUDA Samples\v10.1\1_Utilities\deviceQuery    
\end{verbatim}
\end{small}
然后，用~Visual Studio 2019~打开文件~\verb"deviceQuery_vs2019.sln"。接下来，编译（构建）、运行。若输出内容的最后部分为~\verb"Result = PASS"，则说明已经搭建好~Windows~中的~CUDA~开发环境。若有疑问，请参阅~Nvidia~的官方文档：\url{https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows}。

在上面的测试中，我们是直接用~Visual Studio~打开一个已有的解决方案（solution），然后直接构建并运行。本书不介绍~Visual Studio~的使用，而是选择用命令行解释器编译与运行程序。这里的命令行解释器指的是~Linux~中的~terminal~或者~Windows~中的~command prompt~程序。在~Windows~中使用~MSVC~作为~C++~程序的编译器时，需要单独设置相应的环境变量，或者从~Windows~的开始（start）菜单中找到~Visual Studio 2019~文件夹，然后单击其中的“x64 Native Tools Command Prompt for VS 2019”，而从打开一个加载了~MSVC~环境变量的命令行解释器。在本书的某些章节，需要有管理员的权限来使用~nvprof~性能分析器。此时，可以右击“x64 Native Tools Command Prompt for VS 2019”，然后选择“更多”，接着选择“以管理员身份运行”。

用命令行解释器编译与运行~CUDA~程序的方式在~Windows~和~Linux~系统几乎没有区别，但为了简洁起见，本书后面主要以~Linux~开发环境为例进行讲解。虽然如此，Windows~和~Linux~中的~CUDA~编程功能还是稍有差别。我们将在后续章节中适当的地方指出这些差别。

\section{用~nvidia-smi~检查与设置设备}

可以通过~\verb"nvidia-smi"（Nvidia's system management interface）程序检查与设置设备。它包含在~CUDA~开发工具套装内。该程序最基本的用法就是在命令行解释器中使用不带任何参数的命令~\verb"nvidia-smi"。在作者的计算机中使用该命令，得到如下文本形式的输出：
\begin{small}
\begin{verbatim}
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 426.00       Driver Version: 426.00       CUDA Version: 10.1     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce RTX 207... WDDM  | 00000000:01:00.0 Off |                  N/A |
    | N/A   38C    P8    12W /  N/A |    161MiB /  8192MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
\end{verbatim}
\end{small}

从中可以看出一些比较有用的信息：
\begin{itemize}
\item 第一行可以看到~Nvidia driver~的版本及~CUDA~工具箱的版本。
\item 作者所用计算机中有一型号为~GeForce RTX 2070~的~GPU。该~GPU~的设备号是~0。该计算机仅有一个~GPU。如果有多个~GPU，会将各个~GPU~从~0~开始编号。如果读者的系统中有多个~GPU，而且只需要使用某个特定的~GPU（比如两个之中更强大的那个），则可以通过设置环境变量~\verb"CUDA_VISIBLE_DEVICES" ~的值在运行~CUDA~程序之前选定一个~GPU。假如读者的系统中有编号为~0~和~1~的两个~GPU，而读者想在~1~号~GPU~运行~CUDA~程序，则可以用如下命令设置环境变量：
\begin{verbatim}
    $ export CUDA_VISIBLE_DEVICES=1        
\end{verbatim}
这样设置的环境变量在当前~shell session~及其子进程中有效。
\item 该~GPU~处于~WDDM（windows display driver model ）模式。另一个可能的模式是~TCC（tesla compute cluster），但它仅在~Tesla、Quadro~和~Titan~系列的~GPU~中可选。可用如下方式选择（在~Windows~中需要用管理员身份打开~Command Prompt~并去掉命令中的~sudo）：
\begin{verbatim}
    $ sudo nvidia-smi -g GPU_ID -dm 0 # 设置为 WDDM 模式
    $ sudo nvidia-smi -g GPU_ID -dm 1 # 设置为 TCC 模式
\end{verbatim}
这里，\verb"GPU_ID"~是~GPU~的编号。
\item 该~GPU~当前的温度为~38~摄氏度。GPU~在满负荷运行时，温度会高一些。
\item 这是~GeForce~系列的~GPU，没有~ECC~内存，故~Uncorr. ECC~为~N/A，代表不适用（not applicable）或者不存在（not available）。
\item Compute M.~指计算模式（compute mode）。该~GPU~的计算模式是~Default。在默认模式中，同一个~GPU~中允许存在多个计算进程，但每个计算进程对应程序的运行速度一般来说会降低。还有一种模式为~E. Process，指的是独占进程模式（exclusive process mode），但不适用于处于~WDDM~模式的~GPU。在独占进程模式下，只能运行一个计算进程独占该~GPU。可以用如下命令设置计算模式（在~Windows~中需要用管理员身份打开~Command Prompt~并去掉命令中的~sudo）：
\begin{verbatim}
    $ sudo nvidia-smi -i GPU_ID -c 0 # 默认模式
    $ sudo nvidia-smi -i GPU_ID -c 1 # 独占进程模式
\end{verbatim}
这里，\verb"-i GPU_ID" 的意思是希望该设置仅仅作用于编号为~\verb"GPU_ID"~的~GPU；如果去掉该选项，该设置将会作用于系统中所有的~GPU。
\end{itemize}

关于~\verb"nvidia-smi"~程序更多的介绍，请参考如下官方文档：
\url{https://developer.nvidia.com/nvidia-system-management-interface}。




# Other learning resources

* The official documentation: https://docs.nvidia.com/cuda
* https://devblogs.nvidia.com
* https://stackoverflow.com

