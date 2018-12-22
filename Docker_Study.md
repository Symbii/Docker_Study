# <center> Docker Study Note </center>

## Docker 初尝打包django项目

>	首先pip freeze > requirements.txt

	[root@izbp1278r1bks38x5f48drz django]# cat requirements.txt 
	Django==2.1.4
	django-pure-pagination==0.3.0
	mysqlclient==1.3.14
	pytz==2018.7

>  编写Dockerfile，这次使用host中的mysql，所以没有打包mysql，Dockerfile中路径只有相对路径和绝对路径，下面COPY执行时候后面的容器路径请一定要使用绝对路径，RUN的时候可以使用～这种代表家目录，COPY时候不行。RUN的时候是在容器中，不能采用RUN cp host_file xxxx这样子。因为看不到宿主机上的文件。

	FROM python:3.7
	RUN mkdir -p   ~/app
	RUN mkdir -p   ~/.pip \ 
	        && echo "[global]" > ~/.pip/pip.conf \
	        && echo "index-url = http://mirrors.aliyun.com/pypi/simple/" >> ~/.pip/pip.conf \
	        && echo "[install]">> ~/.pip/pip.conf \
	        && echo "trusted-host=mirrors.aliyun.com" >> ~/.pip/pip.conf 
	
	COPY myblog_code  /root/app/myblog_code
	COPY requirements.txt  /root/app/
	COPY env-py3 /root/app/env-py3
	
	RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
	RUN python get-pip.py
	RUN pip install -r ~/app/requirements.txt
	
	WORKDIR /root/app/myblog_code/blog
	CMD ["python", "manage.py", "runserver", "0:8001"]
	
	EXPOSE 8001
	
> 执行打包命令

	docker build -t myblog:v1.0 .

> 打包之后，执行run让docker跑起来,把host的80端口映射到docker的8001端口，这个是通过dnat实现的，通过iptables dnat -nvL 可以查到对应表项，同时ip add 可以看到服务器host上有一个docker0 网桥，该网桥是用于host与所有docker 通信的。

	docker run -p 80:8001 myblog:v1.0 

2点需要注意，由于使用宿主机的mysql，在与宿主机数据库连接的时候，首先需要开启数据库远程访问，其次需要填宿主机的同一网段ip，不能填127.0.0.1，django同时要支持外网访问，还需要allow_host=[*]


##Docker 学习网站

	docker pull dockerpracticecn/docker_practice
	docker run -it --rm -p 8080:80 dockerpracticecn/docker_practice

访问我的网页：[docker学习](http://116.62.228.221:8080)

