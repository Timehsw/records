https://github.com/hanxiao/bert-as-service

看项目的README.md

在 bert-as-service 文件夹中开启 BERT 服务
```
python app.py -model_dir 模型路径 -num_worker=2
```

num_worker 是指的开启的服务进程数量，此处的 2 就表示是服务器端最高可以处理来自 2 个客户端的并发请求。但是这并不意味着同一时刻只有 2 个客户端可以连接服务，连接数量是没有限制的，只是说在同一个时刻超过 2 个的并发请求将会放在负载均衡进行排队，等待执行。

最后，将 service/client.py 文件放在客户端将要运行的文件夹中。
下面这个例子举的是在服务器上直接访问本机的例子