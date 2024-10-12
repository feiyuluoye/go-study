## Oracle 数据库部署与实施

[TOC]

在不同操作系统和容器中部署 Oracle 数据库有不同的步骤。以下是关于如何在 macOS、Linux、Windows 以及 Docker 上部署 Oracle 数据库的指南：

### 1. macOS 上部署 Oracle 数据库
Oracle 不直接支持在 macOS 上安装 Oracle 数据库，但是你可以使用虚拟机或者 Docker 进行部署。

#### 通过 Docker 在 macOS 上部署
1. **安装 Docker Desktop**：
   - 前往 Docker 官网下载并安装 Docker Desktop for Mac。
   - 启动 Docker Desktop。

2. **拉取 Oracle 数据库镜像**：
   Oracle 提供了官方的 Oracle Database Docker 镜像，可以在 [Oracle Container Registry](https://container-registry.oracle.com) 上找到。
   - 你需要先在 Oracle Container Registry 上注册并接受许可协议。
   - 使用以下命令拉取 Oracle 数据库镜像：
     ```bash
     docker pull container-registry.oracle.com/database/enterprise:19.3.0.0
     ```

3. **运行 Oracle 数据库容器**：
   - 使用以下命令运行容器：
     ```bash
     docker run -d --name oracle-db \
     -p 1521:1521 -p 5500:5500 \
     -e ORACLE_PWD=YourPassword \
     container-registry.oracle.com/database/enterprise:19.3.0.0
     ```
   - `-p 1521:1521` 和 `-p 5500:5500` 是将容器的端口映射到主机的端口。
   - `-e ORACLE_PWD=YourPassword` 设置 Oracle 数据库的 SYS 用户密码。

4. **连接到数据库**：
   - 使用 SQL Developer 或其他 Oracle 客户端工具，连接到 `localhost:1521`，使用用户名 `SYS` 和你设置的密码。

### 2. Linux 上部署 Oracle 数据库
Oracle 官方支持在 Linux 上部署数据库，主要在 Oracle Linux、Red Hat Enterprise Linux (RHEL)、CentOS 等发行版上。

#### 直接在 Linux 上部署
1. **下载 Oracle 数据库**：
   - 访问 [Oracle 官方下载页面](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html)。
   - 下载适用于你系统的 Oracle Database 安装文件，例如 `linuxx64_19c_database.zip`。

2. **安装必要的依赖和设置**：
   - 安装一些必需的软件包：
     ```bash
     sudo yum install -y oracle-database-preinstall-19c
     ```
   - 创建 Oracle 用户和组：
     ```bash
     sudo groupadd oinstall
     sudo groupadd dba
     sudo useradd -g oinstall -G dba oracle
     sudo passwd oracle
     ```
   - 创建 Oracle 数据库目录：
     ```bash
     sudo mkdir -p /u01/app/oracle
     sudo chown -R oracle:oinstall /u01/app/oracle
     sudo chmod -R 775 /u01/app/oracle
     ```

3. **安装 Oracle 数据库**：
   - 以 `oracle` 用户登录并解压下载的安装包：
     ```bash
     unzip linuxx64_19c_database.zip
     cd database
     ```
   - 启动安装程序：
     ```bash
     ./runInstaller
     ```
   - 选择典型安装并按照安装程序提示完成安装。

4. **配置 Oracle 数据库**：
   - 使用 `netca` 配置网络监听。
   - 使用 `dbca` 创建数据库。

#### 通过 Docker 在 Linux 上部署
与 macOS 上的 Docker 步骤相同，参考上面的 Docker 部署部分。

### 3. Windows 上部署 Oracle 数据库
Oracle 提供了直接在 Windows 上安装的可执行安装程序。

1. **下载 Oracle 数据库**：
   - 访问 [Oracle 官方下载页面](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html)。
   - 下载适用于 Windows 的 Oracle Database 安装文件，例如 `WINDOWS.X64_193000_db_home.zip`。

2. **解压和运行安装程序**：
   - 解压下载的 zip 文件。
   - 进入解压目录，运行 `setup.exe`。
   - 在安装向导中选择典型安装，设置 Oracle 主目录、数据库名称、全局数据库名称等参数，完成安装。

3. **配置 Oracle 数据库**：
   - 安装完成后，使用 SQL*Plus 或 Oracle SQL Developer 连接到数据库。

### 4. 使用 Docker 部署 Oracle 数据库
#### 前提条件
- 安装 Docker。
- 登录 Oracle Container Registry 并接受许可协议。

#### 拉取 Oracle 数据库 Docker 镜像
```bash
docker pull container-registry.oracle.com/database/enterprise:19.3.0.0
```

#### 运行 Oracle 数据库容器
```bash
docker run -d --name oracle-db \
-p 1521:1521 -p 5500:5500 \
-e ORACLE_PWD=YourPassword \
container-registry.oracle.com/database/enterprise:19.3.0.0
```

#### 连接到数据库
- 通过 SQL Developer 或其他工具连接到 `localhost:1521`。

#### 停止和启动容器
- 停止容器：
  ```bash
  docker stop oracle-db
  ```
- 启动容器：
  ```bash
  docker start oracle-db
  ```

以上是针对 macOS、Linux、Windows 以及 Docker 上部署 Oracle 数据库的基本步骤。每个环境的细节可能有所不同，请根据实际需求进行调整。