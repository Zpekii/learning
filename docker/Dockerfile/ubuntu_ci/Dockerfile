# 使用官方ubuntu作为父镜像
FROM ubuntu:latest

# 设置镜像维护者信息
LABEL maintainer="Zpekii" description="预安装各种工具，用于做集成工作的ubuntu镜像"

# 预安装工具
RUN apt update && apt install -y \
	vim \
	nano \
	curl	\
	git	\
	rsync \
	&& rm -rf /var/lib/apt/lists/*

# 指定挂载卷
VOLUME ["/data"]

# 默认执行命令
CMD ["/bin/bash"]