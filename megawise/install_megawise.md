---
id: "install_megawise"
lang: "cn"
title: "安装 MegaWise（x86 平台）"
---


# 安装 MegaWise（x86 平台）

本文档主要介绍 x86 平台的 MegaWise Docker 的安装和配置等操作。

> 注意：目前版本的 MegaWise 不具备数据持久化功能。如果您重启了 MegaWise，则必须重新导入数据。

## 安装前提

### 硬件要求

| 组件                     | 配置                    |
|--------------------------|-------------------------|
| GPU |  NVIDIA Pascal 或以上            |
| CPU                 |Intel CPU Sandy Bridge 或以上|
| 内存         | 16 GB 或以上           |
| 硬盘                  | 1 TB 或以上         |


### 软件要求

| 组件                     | 版本                    |
|--------------------------|-------------------------|
| 操作系统                 | Ubuntu 16.04 或以上  |
| NVIDIA 驱动          | 410 或以上，推荐最新版本          |
| Docker                   | 19.03 或以上         |
| NVIDIA Container Toolkit |  1.0.5-1 或以上            |


### 安装 NVIDIA 驱动


1. 禁用 Nouveau 驱动。

   安装 NVIDIA 驱动之前必须先禁用 Nouveau 驱动。请通过以下命令检查是否已启用 Nouveau 驱动：

   ```bash
   $ lsmod | grep nouveau  
   ```

   该命令执行后，如果终端打印了相关信息则说明 Nouveau 驱动已经被启用。如果启用了 Nouveau 驱动，则需执行后续的步骤将其禁用，否则请跳过以下步骤，开始安装 NVIDIA 驱动。

   1. 在以下路径创建文件 `/etc/modprobe.d/blacklist-nouveau.conf` 并在文件中写入如下内容：

      ```
      blacklist nouveau
      options nouveau modeset=0  
      ```

   2. 执行以下命令并重启系统：

      ```bash
      $ sudo update-initramfs -u
      $ sudo reboot  
      ```

   3. 确认禁用 Nouveau 驱动，执行该命令将不输出任何信息。

      ```bash
      $ lsmod | grep nouveau
      ```
   
      如果系统中未安装 lsmod 工具，则先安装 lsmod, 然后执行上述命令。

      ```bash
      $ sudo apt-get install lsmod
      ```

2. 从 [NVIDIA官方驱动下载链接](https://www.nvidia.com/Download/index.aspx?lang=en-us) 下载最新版本的驱动安装文件。

   > 注意：安装或更新 NVIDIA 驱动存在一定风险，有可能导致显示系统崩溃。在安装或更新 NVIDIA 驱动前，请在[NVIDIA官方驱动下载链接](https://www.nvidia.com/Download/index.aspx?lang=en-us)检查您的显卡是否适用最新版本的 NVIDIA 驱动。

3. 安装NVIDIA驱动需要先关闭图形界面， 按 Ctrl+Alt+F1 进入命令行界面，并执行以下命令关闭图形界面。

   ```bash
   $ sudo service lightdm stop
   ```
   
4. 如果您已经安装 NVIDIA 驱动软件，请删除旧的驱动软件。

   ```bash
   $ sudo apt-get remove nvidia-*
   ```
   
5. 赋予安装文件执行权限并安装驱动软件。下面的示例假设安装文件下载在`/home`目录下。

   ```bash
   $ sudo chmod a+x NVIDIA-Linux-x86_64-430.50.run
   $ sudo ./NVIDIA-Linux-x86_64-430.50.run
   ```

6. 重启系统。

   ```bash
   $ sudo reboot  
   ```

7. 测试是否安装成功。

   ```bash
   $ sudo nvidia-smi  
   ```

   如果安装正确，终端打印的内容将包含类似如下信息：

   ```
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 430.34       Driver Version: 430.34       CUDA Version: 10.1     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0  GeForce GTX 1660    Off  | 00000000:01:00.0  On |                  N/A |
   | 28%   49C    P0    24W / 130W |   2731MiB /  5941MiB |      1%      Default |
   +-------------------------------+----------------------+----------------------+
   ```

### 安装 Docker

1. 更新源。

   ```bash
   $ sudo apt-get update
   ```

2. 使用 curl 工具下载最新版本的 Docker。

   ```bash
   $ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   $ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```
   
   如果系统中未安装 curl 工具，则先安装 curl, 然后执行上述命令。
   
   ```bash
   $ sudo apt-get install curl
   ```
   
3. 更新 apt-get 仓库。

   ```bash
   $ sudo apt-get update
   ```

4. 安装 Docker 及其相关的命令行接口和 runtime 环境。

   ```bash
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. 重新执行以下命令验证 Docker 是否安装成功。如果能够打印 Docker 的版本信息，则说明已成功安装 Docker。

   ```bash
   $ docker -v
   ```
   > 注意：如果您是非 root 用户，建议将用户加入 `docker` 用户组。否则需要在 `docker` 命令前加 `sudo`。详细信息请参考 [https://docs.docker.com/install/linux/linux-postinstall/](https://docs.docker.com/install/linux/linux-postinstall/)。

### 安装 NVIDIA container toolkit

1. 使用 curl 添加 gpg key。

   ```bash
   $ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
   sudo apt-key add -
   $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   ```

2. 更新下载源。

   ```bash
   $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
   sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   ```

3. 安装 NVIDIA runtime。

   ```bash
   $ sudo apt-get update
   $ sudo apt-get install -y nvidia-container-toolkit
   ```

4. 重启 Docker daemon。

   ```bash
   $ sudo systemctl restart docker
   ```

5. 验证 NVIDIA container toolkit 是否安装成功。

   ```bash
   $ docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
   ```


如果能够成功打印服务器 GPU 信息，则表示安装成功。


## 自动安装 MegaWise 并导入示例数据

> 注意：自动安装仅用于功能展示。如果您需要部署到生产环境，请使用[手动安装](#手动安装-MegaWise)。

1. 下载脚本 `install_megawise.sh` 和 `data_import.sh` 至同一目录，并确保当前用户对两个脚本有可执行权限。

   ```bash
   $ wget https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/script/data_import.sh \
   https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/script/install_megawise.sh
   $ chmod a+x *.sh
   ```
   
2. 安装 MegaWise 并导入示例数据。

   ```bash
   $ ./install_megawise.sh [参数1，必选] [参数2，可选]
   ```

   > 参数1：MegaWise 安装目录的绝对路径或相对路径，请确保该目录不存在，并且当前用户对该目录有读写权限。您不能通过加 `sudo` 的方式将当前用户没有读写权限的目录设为安装路径。
   
   > 参数2：MegaWise 镜像id，可选，默认'0.5.0'。
   
   示例：
   
   ```bash
   $ ./install_megawise.sh  /home/$USER/megawise '0.5.0'
   ```
   > 注意：如果您是非 root 用户，您必须将用户加入 docker 用户组才能成功执行该脚本。详细信息请参考 [https://docs.docker.com/install/linux/linux-postinstall/](https://docs.docker.com/install/linux/linux-postinstall/)。
   
   该语句所执行的操作如下：

     1. 拉取 MegaWise Docker 镜像。
     2. 下载配置文件和示例数据。
     3. 启动 MegaWise。
     4. 准备示例数据并导入 MegaWise。
     5. 修改相关配置参数。

若出现 `Successfully installed MegaWise and imported test data` 则表示 MegaWise 成功安装且示例数据已导入。自动安装后，MegaWise 的 docker 启动后会内置一个默认数据库 postgres。默认用户名 `zilliz`，密码 `zilliz`。

## 手动安装 MegaWise

### 安装并启动 MegaWise


1. 执行以下命令获得 0.5.0 版本的 MegaWise 的 docker 镜像。

    ```bash
    $ docker pull zilliz/megawise:0.5.0
    ```

2. 安装 PostgreSQL 客户端。

    ```bash
    $ sudo apt-get install curl ca-certificates
    $ curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    $ sudo apt-get update
    $ sudo apt-get install postgresql-client-11
    ```

    PostgreSQL 客户端默认安装路径是`/usr/lib/postgresql/11/bin/`，安装完成后执行 `which psql` 命令，如果没有输出 PostgreSQL 客户端程序的正确路径，需要手动将安装路径加到环境变量中。

    ```bash
    $ export PATH=/usr/lib/postgresql/11/bin:$PATH
    ```

3. 新建一个目录作为工作目录。

    ```bash
    $ cd $WORK_DIR
    $ mkdir conf
    $ mkdir logs
    ```

4. 获取 MegaWise 配置文件。

    ```bash
    $ cd $WORK_DIR/conf
    $ wget https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/user_config.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/etcd.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/megawise_config_template.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/etcd_config_template.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/megawise_config.yaml
    ```

5. 根据 MegaWise 所在的服务器环境修改配置文件。

   1. 打开 `conf` 目录下面的 `user_config.yaml` 配置文件。定位到如下片段：

      ```yaml
      memory:  
        cpu:
          physical_memory: 16 # size in GB
          partition_memory: 16 # size in GB
        
        gpu:
          num: 2
          physical_memory: 2  # size in GB
          partition_memory: 2 # size in GB
      ```

      根据服务器的硬件配置，对上述的配置项进行设置（数值单位为 GB ）。

      `cpu` 部分，`physical_memory` 和 `partition_memory`分别表示 MegaWise 可用的内存总容量和数据缓存分区的内存容量。建议将 `partition_memory` 和 `physical_memory` 均设置为服务器共享内存可用容量的70%以上；
   
      `gpu` 部分，`num` 表示当前 MegaWise 使用的 GPU 数量，`physical_memory` 和 `partition_memory` 分别表示 MegaWise 可用的显存总容量和数据缓存分区的显存容量。建议预留 2GB 显存用于存储计算过程中的中间结果，即将 `partition_memory` 和 `physical_memory` 均设置为单张显卡显存容量的值减2。
        

6. 启动 MegaWise。

    ```bash
    $ docker run --gpus all --shm-size $SHM_SIZE \
                            -e USER=`id -u` -e GROUP=`id -g` \
                            -v $WORK_DIR/conf:/megawise/conf \
                            -v $WORK_DIR/data:/megawise/data \
                            -v $WORK_DIR/logs:/megawise/logs \
                            -v $WORK_DIR/server_data:/megawise/server_data \
                            -v /home/$USER/.nv:/home/megawise/.nv \
                            -v /tmp:/tmp \
                            -p 5433:5432 \
                            $IMAGE_ID
    ```
    > 注意：`$IMAGE_ID` 指 MegaWise Docker 镜像的 image ID，可以通过以下命令查看：

      ```bash
        $ docker image ls
      ```
    > 注意：`-v /tmp:/tmp` 表示对 `tmp` 目录的映射，在本指南中用于存放示例数据。您可以根据实际情况设置映射目录。
    
    参数说明

    **参数说明**

    - `--shm-size`

      Docker image 运行时系统分配的共享内存大小，单位为字节。建议取值为 `/dev/shm` 目录的可用存储的 70%。您可以通过 `df -h` 命令查看`/dev/shm` 目录的可用存储。注意将可用存储的单位转换为字节再计算 `--shm-size` 的值。

    - `-v`

      宿主机和 image 之间的目录映射，用 `:` 隔开，前面是宿主机的目录，后面是 Docker image 的目录。

      在启动容器时可以通过 `-v` 将本地存储的数据文件映射到容器内，以实现本地文件导入 MegaWise 数据库。
      
      > 注意：`-v /tmp:/tmp` 表示对 `tmp` 目录的映射，在本指南中用于存放示例数据。您可以根据实际情况设置映射目录。
    
    - `-e`

      宿主机和 image 之间的用户和组映射。

      在启动容器时可以通过 `-e` 将本地用户和组信息映射到容器内，以实现本地和容器中运行时的权限统一。

    - `-p`

      宿主机和 image 之间的端口映射，用 `:` 隔开，前面是宿主机的端口，后面是 Docker image 的端口，宿主机的端口可以随意设置未被占用的端口，本指南设置为5433。
      
    > 注意：`$IMAGE_ID` 指 MegaWise Docker 镜像的 image ID，可以通过以下命令查看：

      ```bash
        $ docker image ls
      ```
     

    容器启动后，将会启动日志，您可以通过以下命令查看日志：
    
    ```bash
    $ docker logs $CONTAINER_ID
    ```
    
    如果能找到如下日志内容，则说明 MegaWise server 已经启动成功。

    ```bash
    MegaWise server is running...
    ```

### 连接 MegaWise

MegaWise Docker 启动之后，您可以选择从 Docker 内部连接 MegaWise 或者从 Docker 外部连接 MegaWise。

> 如果您要使用 Infini 可视化界面，则必须从 Docker 外部连接 MegaWise。

#### 从 Docker 内部连接 MegaWise

 1. 进入 MegaWise Docker 的 bash 命令并连接 MegaWise 数据库：
 
    ```shell
    $ docker exec -u `id -u` -it $CONTAINER_ID bash
    $ cd script && ./connect.sh
    ```   
    如果出现以下信息：

    ```bash
    psql (11.6 (Ubuntu 11.6-1.pgdg18.04+1), server 11.1)
    Type "help" for help.

    postgres=#
    ```

    就说明成功连接上 MegaWise 了。
    
#### 从 Docker 外部连接 MegaWise 

 1. 关闭 MegaWise。

    ```bash
    $ docker stop $CONTAINER_ID
    ```

 2. 进入 MegaWise 的工作目录并进行以下修改： 

    1. 打开 `data` 目录下面的 `postgresql.conf` 配置文件。将 `listen_addresses` 参数的值设置为 `'*'`（注意引号）并取消此行的注释。

    2. 打开 `data` 目录下面的 `pg_hba.conf` 配置文件。在 `# IPv4 local connections` 下方添加如下行：

        ```bash
        host   all      all     0.0.0.0/0      trust
        ```
 3. 重新启动 MegaWise。

    > 注意：您不能使用 `docker start $CONTAINER_ID` 的方式来重新启动 MegaWise。

    ```bash
    $ docker run --gpus all --shm-size $SHM_SIZE \
                            -e USER=`id -u` -e GROUP=`id -g` \
                            -v $WORK_DIR/conf:/megawise/conf \
                            -v $WORK_DIR/data:/megawise/data \
                            -v $WORK_DIR/logs:/megawise/logs \
                            -v $WORK_DIR/server_data:/megawise/server_data \
                            -v /home/$USER/.nv:/home/megawise/.nv \
                            -v /tmp:/tmp \
                            -p 5433:5432 \
                            $IMAGE_ID
    ```
    
    **参数说明**

    - `--shm-size`

      Docker image 运行时系统分配的共享内存大小，单位为字节。建议取值为 `/dev/shm` 目录的可用存储的 70%。您可以通过 `df -h` 命令查看`/dev/shm` 目录的可用存储。注意将可用存储的单位转换为字节再计算 `--shm-size` 的值。

    - `-v`

      宿主机和 image 之间的目录映射，用 `:` 隔开，前面是宿主机的目录，后面是 Docker image 的目录。

      在启动容器时可以通过 `-v` 将本地存储的数据文件映射到容器内，以实现本地文件导入 MegaWise 数据库。
      
      > 注意：`-v /tmp:/tmp` 表示对 `tmp` 目录的映射，在本指南中用于存放示例数据。您可以根据实际情况设置映射目录。
    
    - `-e`

      宿主机和 image 之间的用户和组映射。

      在启动容器时可以通过 `-e` 将本地用户和组信息映射到容器内，以实现本地和容器中运行时的权限统一。

    - `-p`

      宿主机和 image 之间的端口映射，用 `:` 隔开，前面是宿主机的端口，后面是 Docker image 的端口，宿主机的端口可以随意设置未被占用的端口，本指南设置为5433。
      
    > 注意：`$IMAGE_ID` 指 MegaWise Docker 镜像的 image ID，可以通过以下命令查看：

      ```bash
        $ docker image ls
      ```
     

    容器启动后，将会启动日志，您可以通过以下命令查看日志：
    
    ```bash
    $ docker logs $CONTAINER_ID
    ```
    
    如果能找到如下日志内容，则说明 MegaWise server 已经启动成功。

    ```bash
    MegaWise server is running...
    ```

 4. 操作 MegaWise。
  
    ```bash
    $ psql -U $USER_ID -p 5433 -h $IP_ADDR -d postgres
    ```

    > `$USER_ID` 可以通过以下命令得到：

    ```bash
    $ id -u
    ```
    > `$IP_ADDR` 可以通过以下命令得到：

    ```bash
    $ ifconfig
    ```

    如果出现以下信息：

    ```bash
    psql (11.6 (Ubuntu 11.6-1.pgdg18.04+1), server 11.1)
    Type "help" for help.

    postgres=#
    ```

    就说明成功连接上 MegaWise 了。MegaWise 的 docker 启动后会内置一个默认数据库 postgres。

    > 注意：如果连接超时，建议检查防火墙设置是否正确。MegaWise 当前版本不提供数据持久化功能，建议每次重启后重新进行数据导入。
    
### 创建 MegaWise 用户并导入数据
    
1. 在 postgres 数据库中创建一个用户。用户名为 `zilliz`，密码为 `zilliz`。
    
   ```bash
   postgres=# CREATE USER zilliz WITH PASSWORD 'zilliz';
   postgres=# grant all privileges on database postgres to zilliz;
   ```
2. 用户创建完成后，您可以使用创建好的用户向 postgres 数据库中导入示例数据。

   获取示例数据：

   ```bash
   $ wget -P /tmp https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/sample_data/nyc_taxi_data.csv
   ```

   创建扩展、建表、并导入示例数据：
   
   ```bash
      postgres=# create extension zdb_fdw;
      postgres=# \c - zilliz；
      postgres=> create table nyc_taxi(
       vendor_id text,
       tpep_pickup_datetime timestamp,
       tpep_dropoff_datetime timestamp,
       passenger_count int,
       trip_distance float,
       pickup_longitute float,
       pickup_latitute float,
       dropoff_longitute float,
       dropoff_latitute float,
       fare_amount float,
       tip_amount float,
       total_amount float
       );
      postgres=> copy nyc_taxi from '/tmp/nyc_taxi_data.csv'
       WITH DELIMITER ',' csv header;
    ```
      
> 注意：如果您需要在 Infini 可视化界面上创建图表，则需要创建扩展 `zdb_fdw`。创建扩展后需要切换到 “zilliz” 用户来建表并导入数据。使用 copy 导入数据时应该保证数据所在目录已经映射到 MegaWise Docker。本安装指南在之前步骤已经完成了 `tmp` 目录的映射，所以您可以直接使用 `tmp` 来存放并导入数据。

## 接下来您可以

[安装 Infini](./install_infini)
