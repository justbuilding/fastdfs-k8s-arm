# 选择系统镜像作为基础镜像，使用 Ubuntu 26.04
FROM ubuntu:26.04

LABEL MAINTAINER liyanjing 284223249@qq.com

# 0.安装包位置，fdfs的基本目录和存储目录
ENV INSTALL_PATH=/usr/local/src \
  FASTDFS_VERSION="6.08" \
  FASTDFS_NGINX_MODULE_VERSION="1.22"

# 0.change the system source for installing libs
RUN sed -i 's|http://archive.ubuntu.com/ubuntu/|http://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list \
  && sed -i 's|http://security.ubuntu.com/ubuntu/|http://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list

# 1.复制安装包和已编译的nginx
ADD soft ${INSTALL_PATH}
COPY nginx /usr/local/nginx/sbin/

# 2.环境安装
# - 创建fdfs的存储目录
# - 安装依赖
# - 安装libfastcommon
# - 安装fastdfs
# - 配置nginx和fastdfs联合环境
RUN apt-get update && apt-get install -y --no-install-recommends \
    make \
    gcc \    
    g++ \
    unzip \
    libpcre3 \
    zlib1g-dev libpcre3-dev \
    zlib1g \
    perl \
    vim \
  && cd ${INSTALL_PATH} \
  && unzip libfastcommon-master.zip \
  && unzip fastdfs-${FASTDFS_VERSION}.zip \
  && unzip fastdfs-nginx-module-${FASTDFS_NGINX_MODULE_VERSION}.zip \
  \
  && cd ${INSTALL_PATH}/libfastcommon-master/ \
  && ./make.sh \
  && ./make.sh install \
  && cd ${INSTALL_PATH}/fastdfs-${FASTDFS_VERSION}/ \
  && ./make.sh \
  && ./make.sh install \
  \
  # 配置 fastdfs-nginx-module
  && cp ${INSTALL_PATH}/fastdfs-nginx-module-${FASTDFS_NGINX_MODULE_VERSION}/src/mod_fastdfs.conf /etc/fdfs/ \
  \
  && rm -rf ${INSTALL_PATH}/* \
  && apt-get purge -y \
    unzip \
  && apt-get autoremove -y \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# 3.添加配置文件，目标路径以/结尾，docker会把它当作目录，不存在时，会自动创建
COPY conf/*.* /etc/fdfs/
RUN rm -f /usr/local/nginx/conf && mkdir -p /usr/local/nginx/conf/ && mkdir -p /usr/local/nginx/logs
COPY nginx_conf/* /usr/local/nginx/conf/
COPY nginx_conf.d/*.conf /usr/local/nginx/conf.d/
COPY html/* /usr/local/nginx/html/
COPY start.sh /


ENV TZ=Asia/Shanghai

# 4.更改启动脚本执行权限，设置时区为中国时间
RUN chmod u+x /start.sh \
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 22122 23000 80

WORKDIR /

# 镜像启动
ENTRYPOINT ["/bin/bash","/start.sh"]
