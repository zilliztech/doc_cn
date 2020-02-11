---
id: "install_megawise_powerpc"
lang: "cn"
title: "安装 MegaWise (PowerPC 平台)"
---


# 安装 MegaWise (PowerPC 平台)

本文档主要介绍 PowerPC 平台上 MegaWise Docker 的安装和配置等操作。

## 安装前提

### 硬件要求

| 组件                     | 配置                    |
|--------------------------|-------------------------|
| GPU |  NVIDIA Pascal 或以上            |
| CPU                 | ppc64le 架构    |
| 内存         | 16 GB 或以上           |
| 硬盘                  | 1 TB 或以上         |

> 注意：ppc64le 架构以外的 PowerPC CPU 未经过测试，有可能不支持 MegaWise。

### 软件要求

| 组件                     | 版本                    |
|--------------------------|-------------------------|
| 操作系统                 | CentOS 7 或以上  |
| NVIDIA 驱动          | 410 或以上，推荐最新版本          |
| Docker                   | 仅支持18.03         |
| NVIDIA Container Runtime |      最新版本        |

> 注意：PowerPC 平台的软件要求与 x86 平台的软件要求不同。请检查操作系统和 Docker 版本是否满足要求。PowerPC 平台不支持 NVIDIA Container Toolkit，您需要使用 NVIDIA Container Runtime。   

### 安装 NVIDIA 驱动、Docker 和 NVIDIA Container Runtime

具体安装方法请参考这些软件的官方网站：

- Docker：[https://www.docker.com/](https://www.docker.com/)
- NVIDIA 驱动：[https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us)
- NVIDIA Container Runtime：[https://github.com/NVIDIA/nvidia-container-runtime](https://github.com/NVIDIA/nvidia-container-runtime)


> 注意：您可以使用以下命令检查 NVIDIA 驱动的安装版本：

```bash
$ sudo nvidia-smi 
```

> 注意：您可以使用以下命令检查 Docker 和 NVIDIA Container Runtime 是否安装成功：

```bash
$ docker run --runtime=nvidia --rm nvidia/cuda-ppc64le nvidia-smi
```
> 注意：如果您是非 root 用户，建议将用户加入 `docker` 用户组。否则需要在 `docker` 命令前加 `sudo`。详细信息请参考 [https://docs.docker.com/install/linux/linux-postinstall/](https://docs.docker.com/install/linux/linux-postinstall/)。

### 安装 PostgreSQL 客户端

为了验证 MegaWise 是否安装成功，您需要安装 PostgreSQL 11 客户端。具体安装方法请参考[https://www.postgresql.org/download/](https://www.postgresql.org/download/)。

## 安装步骤

> 注意：不要使用有 root 权限的用户进行安装或运行 MegaWise Docker。

> 注意：MegaWise 当前版本不提供数据持久化功能，每次重启后您必须重新进行数据导入。

1. 执行以下命令获得 0.5.0-ppc64le 版本的 MegaWise 的 Docker 镜像。

    ```bash
    $ docker pull zilliz/megawise:0.5.0-ppc64le
    ```

2. 新建一个目录作为工作目录。

    ```bash
    $ cd $WORK_DIR
    $ mkdir conf
    $ mkdir logs
    ```

3. 获取 MegaWise 配置文件。

    ```bash
    $ cd $WORK_DIR/conf
    $ wget https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/user_config.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/etcd.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/megawise_config_template.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/etcd_config_template.yaml \
    https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/db/megawise_config.yaml
    ```
    
    > 注意：万一您遇到无法通过 `wget` 命令正常下载配置文件的情况，您也可以创建上方命令中的文件，然后复制[https://github.com/zilliztech/infini/tree/v0.5.0/config/db](https://github.com/zilliztech/infini/tree/v0.5.0/config/db) 中相关文件的内容。

4. 根据 MegaWise 所在的服务器环境修改配置文件。

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

      根据服务器的硬件配置，对上述的配置项进行设置（数值单位为 GB）。

      `cpu` 部分，`physical_memory` 和 `partition_memory`分别表示 MegaWise 可用的内存总容量和数据缓存分区的内存容量。建议将 `partition_memory` 和 `physical_memory` 均设置为服务器共享内存可用容量的70%以上；
   
      `gpu` 部分，`num` 表示当前 MegaWise 使用的 GPU 数量，`physical_memory` 和 `partition_memory` 分别表示 MegaWise 可用的显存总容量和数据缓存分区的显存容量。建议预留 2GB 显存用于存储计算过程中的中间结果，即将 `partition_memory` 和 `physical_memory` 均设置为单张显卡显存容量的值减2。
        

5. 启动 MegaWise。

    ```bash
    $ docker run -d --runtime=nvidia --shm-size $SHM_SIZE \
                            -e USER=megawise -e GROUP=`id -g` \
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

### 连接 MegaWise

MegaWise Docker 启动之后，您可以选择从 Docker 内部连接 MegaWise 或者从 Docker 外部连接 MegaWise。

> 如果您要使用 Infini 可视化界面，则必须从 Docker 外部连接 MegaWise。

#### 从 Docker 内部连接 MegaWise

 1. 进入 MegaWise Docker 的 bash 命令并连接 MegaWise 数据库：
 
    ```shell
    $ docker exec -u megawise -it $CONTAINER_ID bash
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
    $ docker run -d --runtime=nvidia --shm-size $SHM_SIZE \
                            -e USER=megawise -e GROUP=`id -g` \
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

 4. 连接 MegaWise。
  
    ```bash
    $ psql -U megawise -p 5433 -h $IP_ADDR -d postgres
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

    > 注意：如果连接超时，建议检查防火墙设置是否正确。
    
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
      postgres=# \c zilliz;
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
      
> 注意：如果您需要在 Infini 可视化界面上创建图表，则需要创建扩展 `zdb_fdw`。创建扩展后需要切换到 `zilliz` 用户来建表并导入数据。使用 `copy` 导入数据时应该保证数据所在目录已经映射到 MegaWise Docker。本安装指南在之前步骤已经完成了 `tmp` 目录的映射，所以您可以直接使用 `tmp` 来存放并导入数据。

## 接下来您可以

[安装 Infini](./install_infini)
