# WebStrom连接Docker部署前端Vue项目



### 配置文件

##### default.conf

```nginx
server {
    listen       9528; # 监听的端口号
    server_name  localhost; # 修改为docker服务宿主机的ip

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html =404;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

##### Dockerfile

```dockerfile
# nginx作为基础镜像
FROM nginx:1.20.2

MAINTAINER kezhou  # 定义作者

#移除基础镜像内部的nginx的默认配置文件
#RUN rm /etc/nginx/conf.d/default.conf

#将自己定义的nginx文件 拷贝到原nginx文件的位置
COPY nginx.conf /etc/nginx/
ADD default.conf /etc/nginx/conf.d/

#将前端build好生成的dist文件拷贝 nginx代理的文件夹内
COPY dist/ /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Docker插件配置

我们使用Windows 11 安装的 Docker for Windows

 ![image-20221028085347882](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028085316784.png)

* 选TCP socket，既可以选本机，也可以选远程的Linux

 ![image-20221028085523990](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028085523990.png)

* 其他Linux机器

 ![image-20221028085856776](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028085856776.png)



 ![image-20221028091919455](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028085235753.png)



### 先build工程

打开 package.json 文件，在 scripts 代码段选择编译环境（向右箭头）表示可以运行，每个环境对应一个配置文件

 ![image-20221028090113453](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028090113453.png)

* build过程

```
D:\03_Dev\nodejs\npm.cmd run build:prod

> vue-element-admin@4.4.0 build:prod D:\04_GitHub\nodejs\vue-element-admin
> vue-cli-service build
...
  Images and other types of assets omitted.

 DONE  Build complete. The dist directory is ready to be deployed.
 INFO  Check out deployment instructions at https://cli.vuejs.org/guide/deployment.html


Process finished with exit code 0
```

 ![image-20221028090401167](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028090401167.png)

* build完成后会生成 dist 文件夹

 ![image-20221028090444716](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028090444716.png)

### 再 Build Docker镜像

打开DockerFile文件，选择执行【Build Image for ‘DockerFile’】，这里的 DockerFile是上面Docker插件里面配置的。

 ![image-20221028090839152](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028090839152.png)

#### 开始构建

 ![image-20221028092056640](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028091314918.png)

#### 构建过程

```
Deploying 'vue-element-admin:1.0.0 Dockerfile: Dockerfile'��
Building image��
Preparing build context archive��
[==================================================>]43862/43862 files
Done

Sending build context to Docker daemon��
[==================================================>] 78.22MB
Done

Step 1/7 : FROM nginx:1.20.2
 ---> 0584b370e957
Step 2/7 : MAINTAINER kezhou  # ��������
 ---> Using cache
 ---> b42865c7160c
Step 3/7 : COPY nginx.conf /etc/nginx/
 ---> Using cache
 ---> 37692e9a83b0
Step 4/7 : ADD default.conf /etc/nginx/conf.d/
 ---> Using cache
 ---> 076dee746de9
Step 5/7 : COPY dist/ /usr/share/nginx/html/
 ---> 00bdea6119c6
Step 6/7 : EXPOSE 80
 ---> Running in 15d7a2883073
Removing intermediate container 15d7a2883073
 ---> e694301bcf48
Step 7/7 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in 9ec25d94a2aa
Removing intermediate container 9ec25d94a2aa
 ---> 13f75b92aa3c

Successfully built 13f75b92aa3c
Successfully tagged vue-element-admin:1.0.0
'vue-element-admin:1.0.0 Dockerfile: Dockerfile' has been deployed successfully.
```

#### 构建完成

 ![image-20221028092445799](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028092445799.png)

#### 镜像信息

 ![image-20221028095704240](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028095704240.png)

### 运行镜像

#### 运行命令

```
docker run -d -p 9535:9528 --name vue-element-admin vue-element-admin:1.0.0
```

#### 运行结果

 ![image-20221028095853225](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028095853225.png)

#### 访问服务

```
http://localhost:9535
```



 ![image-20221028100126643](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028100126643.png)



 ![image-20221028100156314](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028100156314.png)



### WSL2

```
wsl -l -``v
```



 ![image-20221028094526072](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028094526072.png)



### 参考

* [WebStrom连接Docker部署前端Vue(Vue-admin-template脚手架)项目](https://blog.csdn.net/weixin_43562415/article/details/120128792)

* [WebStorm控制台中文乱码解决](https://blog.csdn.net/qq_47452289/article/details/118676221)

  在WebStorm菜单依次选择【Help】-》【Edit Custom VM Options...】点击会打开一个配置文件，在最下面加一行代码 

  -Dfile.encoding=utf-8

  再重启WebStorm就ok了

 ![image-20221028100850032](https://raw.githubusercontent.com/CoderDream/vue-element-admin/main/assets/image-20221028100850032.png)
