---
id: "install_infini"
lang: "cn"
title: "安装 Infini 可视化交互分析界面"
label1: "用户手册"
label2: "Infini"
---
# 安装 Infini 可视化交互分析界面


## 安装前提

1. 请确认已安装以下软件。
   - [Docker 19.03 or higher](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
   - [Docker Compose](https://docs.docker.com/compose/install/)

2. 请确认已安装 MegaWise，开启 MegaWise 服务并导入示例数据。
   - [安装 MegaWise](./install_megawise)




## 使用 Docker Compose 运行 Infini 界面

1. 确保 docker-compose 正在运行。 

   ```bash
   $ docker-compose --version
   ```

    如果终端能够显示 docker-compose 的版本信息，则说明系统中已经安装有对应版本的 docker-compose。在如下的示例中 docker-compose 的版本为1.24.1。

    ```bash
    docker-compose version 1.24.1, build 4667896b
    ```

2. 下载两个配置文件到同一文件目录。

   ```bash
   $ wget https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/webserver/.env \
   https://raw.githubusercontent.com/zilliztech/infini/v0.5.0/config/webserver/docker-compose.yml

   ```

3. 修改 `.env` 文件。

   > 注意：请把 `192.168.1.60` 改成当前运行 Infini docker 的服务器 IP 地址。请把 `192.168.1.106` 改成当前运行 MegaWise docker 的服务器 IP 地址。

   ```yml
   # 默认API服务地址
   API_URL=http://192.168.1.60:9000
   # 默认web服务端口
   LOCAL_PORT=80
   # megawise ip
   MEGAWISE_HOST=192.168.1.106
   # megawise 用户名
   MEGAWISE_USER=zilliz
   # megawise 密码
   MEGAWISE_PWD=zilliz
   # megawise 数据库
   MEGAWISE_DB=postgres
   # megawise 端口
   MEGAWISE_PORT=5433
   ```

4. 启动 Infini web server。

   ```shell
   # start Infini
   $ docker-compose -f docker-compose.yml up
   ```

5. 修改 host，打开 `/etc/hosts` 文件，添加以下一条
   > 注意: 请把 `192.168.1.60` 改成当前运行Infini docker 的服务器 IP 地址。若使用 Windows 系统查看 Infini 界面，则在`C:\Windows\System32\drivers\etc\hosts` 文件中添加。

   ```shell
    #/etc/hosts
    192.168.1.60 infini
   ```


6. 打开任意浏览器，推荐 Chrome 和 Firefox。

   ```shell
   # 如果修改了80端口，请加上端口号
   http://192.168.1.60
   ```


## 设置 Infini 界面


1. 在登录界面上，输入用户名和密码进行登录：

   - 默认用户名: zilliz
   - 默认密码: zilliz

2. 登录后，输入 MegaWise 数据库的相关信息，点击保存，界面就会跳转到仪表盘页面。


3. 单击 **New York Taxi Boards**，出现以下界面：

![New York Taxi data](../assets/nyc-demo.png)

如果您可以看到以上界面，说明 Infini 界面已经成功启动了。


4. 使用以下命令关闭 Infini 界面。

```bash
# Stop Infini
$ docker-compose -f docker-compose.yml down
```
