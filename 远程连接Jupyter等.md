## 连接到服务器上的jupyter notebook
在shell中连接到远程服务器之后：

安装
```
pip intall jupyter
```

打开python：
```
ipython
```

在python中生成密码对应的加密字符串：
```
In [1]: from notebook.auth import passwd
In [2]: passwd()  回车后自动出现设置密码的地方
Enter password: 在这里设置密码，密码不会显示在屏幕上
Verify password: 重复刚才的密码
Out[2]: 'sha1:一堆看起来像乱码的字母和数字后面要用所以找个地方存下来'    
In [3]: quit()
```

生成和修改配置文件：
```
jupyter notebook --generate-config -y
cat >>~/.jupyter/jupyter_notebook_config.py <<EOF
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.password = u'sha1:粘贴上一步得到的那一长串东西'
c.NotebookApp.port = 8888
c.NotebookApp.allow_remote_access = True
EOF
```

打开jupyter notebook：
```
jupyter notebook
```

显示如下：
```
WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
Serving notebooks from local directory: /home/ubuntu/data
The Jupyter Notebook is running at:
http://(ip-xxx-xx-xx-x or 127.0.0.1):8888/
Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

此时打开浏览器，在地址栏输入自己的公有(或公有DNA):8888，回车。公有ip在EC2的控制台中查看，就是远程连接时的ip地址。  
这时候会出现不安全提醒，忽略，出现密码框，输入前面设置的密码就可以了。 

退出的时候敲ctrl+c，提示是否关闭敲y(yes)。


