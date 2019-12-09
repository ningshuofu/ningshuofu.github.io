---
title: Dockerfile
categories: docker
tags: docker
date: 2019-12-05 11:03:56
---
# 1.Dockerfile使用文档

**1. MAINTAINER**：设置该镜像的作者。语法如下：

MAINTAINER <author name>


**2. RUN**：在shell或者exec的环境下执行的命令。RUN指令会在新创建的镜像上添加新的层面，接下来提交的结果用在Dockerfile的下一条指令中。语法如下：

RUN 《command》


**3. ADD**：复制文件指令。它有两个参数<source>和<destination>。destination是容器内的路径。source可以是URL或者是启动配置上下文中的一个文件。语法如下：

ADD 《src》 《destination》


**4. CMD**：提供了容器默认的执行命令。 Dockerfile只允许使用一次CMD指令。 使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。 CMD有三种形式：

CMD ["executable","param1","param2"]

CMD ["param1","param2"]

CMD command param1 param2

注：cmd需要配置绝对路径


**5. EXPOSE**：指定容器在运行时监听的端口。语法如下：

EXPOSE <port>;


**6. ENTRYPOINT**：配置给容器一个可执行的命令，这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。同时也意味着该镜像每次被调用时仅能运行指定的应用。类似于CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。语法如下：

ENTRYPOINT ["executable", "param1","param2"]

ENTRYPOINT command param1 param2


**7. WORKDIR**：指定RUN、CMD与ENTRYPOINT命令的工作目录。语法如下：

WORKDIR /path/to/workdir


**8. ENV**：设置环境变量。它们使用键值对，增加运行程序的灵活性。语法如下：

ENV <key> <value>


**9. USER**：镜像正在运行时设置一个UID。语法如下：

USER <uid>


**10. VOLUME**：授权访问从容器内到主机上的目录。语法如下：

VOLUME ["/data"]

在c7-novnc基础上安装tkj，

RUN pip install tensorflow-gpu &&\

\# Install keras

​	pip install keras &&\

\# Install jupyter

​	yum install -y python-devel &&\

​	pip install jupyter

刚开始简单这样安装三个软件，网络墙了所以网上找了可以指定镜像，变成

RUN pip install -i https://pypi.doubanio.com/simple/ tensorflow-gpu &&\

\# Install keras

​	pip install -i https://pypi.doubanio.com/simple/ keras &&\

\# Install jupyter

​	yum install -y python-devel &&\

​	pip install -i https://pypi.doubanio.com/simple/ jupyter

后面难点是安装jupyter需要配置，开始是

EXPOSE 22  

CMD ["/run.sh"]

里面的run.sh中写一个python脚本然后python xxx.py，后面经过指导直接将配置好的文件copy就行，于是变成

ADD run.sh /run.sh

RUN chmod 755 /*.sh

 

\# Configure Jupyter

RUN jupyter notebook --generate-config

ADD jupyter_notebook_config.py /root/.jupyter/jupyter_notebook_config.py

中间发现通过yml创建容器已知失败，

原因是润。run.sh使用相对路径，改成/run.sh就能通过yml创建容器。最后的坑是自己配置文件格式有问题导致一直失败，只要安装规范就不会错

vi /root/.jupyter/jupyter_notebook_config.py 

修改 

\#c.NotebookApp.ip = 'localhost'

c.NotebookApp.ip = '0.0.0.0'

修改

\#c.NotebookApp.notebook_dir = u''

c.NotebookApp.notebook_dir = u'/home'

修改

\#c.NotebookApp.password = u''

c.NotebookApp.password = u'sha1:c1ebae9ce1b5:0c25328cd843b9f57b2043de1bb55f464d023939'

 