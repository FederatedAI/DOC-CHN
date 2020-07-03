Fate DEV&DEBUG step by step MAC 开发指南
=================================


## 1 环境配置

-  安装python3  `brew install python3`  python3自带pip3

-  安装系统依赖 `brew install gmp libmpc mpfr hdf5 leveldb`

-  安装python依赖，在fate 根目录执行 `pip3 install -r requirements.txt `

-  安装PyCharm，下载网址：`https://www.jetbrains.com/pycharm/download/`

## 2 简单测试

-  安装PyCharm打开Fate,使用IDE启动fate_flow/fate_flow_server.py 脚本

-  使用IDE运行 examples/federatedml-1.x-examples/quick_run.py 脚本，控制台将输入如下信息

    >/Users/xxx/venv/bin/python /Users/xxx/FATE/examples/federatedml-1.x-examples/quick_run.py
    Upload data config json: {'file': 'examples/data/default_credit_homo_guest.csv', 'head': 1, 'partition': 10, 'work_mode': 0, 'table_name': 'default_credit_homo_guest', 'namespace': 'default_credit_homo_guest_guest'}
    stdout:{
        "data": {
            "board_url": "http://192.168.31.58:8080/index.html#/dashboard?job_id=202001181915023702281&role=local&party_id=0",
            "job_dsl_path": "/Users/xxx/FATE/jobs/202001181915023702281/job_dsl.json",
            "job_runtime_conf_path": "/Users/xxx/FATE/jobs/202001181915023702281/job_runtime_conf.json",
            "logs_directory": "/Users/xxx/FATE/logs/202001181915023702281",
            "namespace": "default_credit_homo_guest_guest",
            "table_name": "default_credit_homo_guest"
        },
        "jobId": "202001181915023702281",
        "retcode": 0,
        "retmsg": "success"
    }
    Upload data config json: {'file': 'examples/data/default_credit_homo_host.csv', 'head': 1, 'partition': 10, 'work_mode': 0, 'table_name': 'default_credit_homo_host', 'namespace': 'default_credit_homo_host_host'}
    stdout:{
        "data": {
            "board_url": "http://192.168.31.58:8080/index.html#/dashboard?job_id=202001181915058443762&role=local&party_id=0",
            "job_dsl_path": "/Users/xxx/FATE/jobs/202001181915058443762/job_dsl.json",
            "job_runtime_conf_path": "/Users/xxx/FATE/jobs/202001181915058443762/job_runtime_conf.json",
            "logs_directory": "/Users/xxx/FATE/logs/202001181915058443762",
            "namespace": "default_credit_homo_host_host",
            "table_name": "default_credit_homo_host"
        },
        "jobId": "202001181915058443762",
        "retcode": 0,
        "retmsg": "success"
    }
    dsl_path: /Users/xxx/FATE/examples/federatedml-1.x-examples/user_config/train_dsl.config_1579346108_9453, conf_path: /Users/xxx/FATE/examples/federatedml-1.x-examples/user_config/train_conf.config_1579346108_7399
    stdout:{
        "data": {
            "board_url": "http://192.168.31.58:8080/index.html#/dashboard?job_id=202001181915093328073&role=guest&party_id=10000",
            "job_dsl_path": "/Users/xxx/FATE/jobs/202001181915093328073/job_dsl.json",
            "job_runtime_conf_path": "/Users/xxx/FATE/jobs/202001181915093328073/job_runtime_conf.json",
            "logs_directory": "/Users/xxx/FATE/logs/202001181915093328073",
            "model_info": {
                "model_id": "arbiter-10000#guest-10000#host-10000#model",
                "model_version": "202001181915093328073"
            }
        },
        "jobId": "202001181915093328073",
        "retcode": 0,
        "retmsg": "success"
    }
    Please check your task in fate-board, url is : http://192.168.31.58:8080/index.html#/dashboard?job_id=202001181915093328073&role=guest&party_id=10000
    The log info is located in /Users/xxx/FATE/examples/federatedml-1.x-examples/../../logs/202001181915093328073
    Task is running, wait time: 10.55770993232727
    Task is running, wait time: 21.68661403656006
    Task is running, wait time: 32.47444009780884
    Task is running, wait time: 43.21940803527832
    Task is running, wait time: 53.8386709690094


## 3 DEBUG 

   由于Fate算法模块启动是多进程方式进行，而且进程启动时有很多参数，也无法直接对算法进行调试，因此我们需要找到算法模块进程启动的入口。可以在
   fate_flow/driver/task_scheduler.py 脚本
   > p = job_utils.run_subprocess(config_dir=task_dir, process_cmd=process_cmd, log_dir=task_log_dir)  
   print("run subprocess command:" + " ".join(process_cmd)) ###打印算法进程

   然后运行 examples/federatedml-1.x-examples/quick_run.py 在fate_flow中将会打印算法进程启动的具体参数:
   > run subprocess command:python3 /xxx/python/FATE/fate_flow/driver/task_executor.py -j 2020020822271098076518 -n dataio_0 -t 2020020822271098076518_dataio_0 -r guest -p 10000 -c /xxx/python/FATE/jobs/xxx/guest/10000/dataio_0/task_config.json --job_server 192.168.31.58:9380
   
   可以看到算法模块是通过fate_flow/driver/task_executor.py脚本文件启动的，那么我们将task_executor.py作为启动脚本，并将后面-j 2020020822271098076518 -n ... 
   的所有参数复制到DEBUG 的Paramaters，就可以进行debug了，具体PyCharm操作为Run->Edit Configuration，如下图
   ![avatar](https://pic1.zhimg.com/80/v2-47b11befb82d4384f8bd5c34f9fec72c_1440w.jpg)
   
   另外，由于PyCharm在启动子进程时会自动加入环境变量等参数，因此还有个地方需要设置，具体操作为PyCharm->Preference：
      ![avatar](https://pic4.zhimg.com/80/v2-bd39f5c04cb84b896b761f1f293e62db_1440w.jpg)
   在Python Debugger中不要勾选Attach to subprocess automatically while debugging

   
   设置之后，在debug模式下启动task_executor.py文件，并在具体的算法模块加入断点即可开始debug了
  
  
 