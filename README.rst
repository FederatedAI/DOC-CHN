|License| |CodeStyle| |Style|

.. container::

`DOC <https://github.com/FederatedAI/FATE/tree/master/doc>`__ \| `Quick Start <https://github.com/FederatedAI/FATE/tree/master/examples/federatedml-1.x-examples>`__
\| `English <https://github.com/FederatedAI/FATE/blob/master/README.md>`__

FATE (Federated AI Technology Enabler)
是微众银行AI部门发起的开源项目，为联邦学习生态系统提供了可靠的安全计算框架。FATE项目使用多方安全计算
(MPC) 以及同态加密 (HE)
技术构建底层安全计算协议，以此支持不同种类的机器学习的安全计算，包括逻辑回归、基于树的算法、深度学习和迁移学习等。

FATE官方网站：\ https://fate.fedai.org/

FATE中的联邦学习算法
--------------------

FATE目前支持三种类型联邦学习算法：横向联邦学习、纵向联邦学习以及迁移学习。算法细节请参考文档
`federatedml <./Federatedml>`__ 。

安装教程
--------

FATE支持Linux或Mac操作系统，当前FATE支持：

-  Native部署: 单机部署和集群部署;

-  KubeFATE部署

Native部署
~~~~~~~~~~

运行环境: jdk1.8+、Python3.6、python virtualenv、mysql5.6+、redis-5.0.2

单机部署
''''''''

FATE为开发人员提供了单机部署架构版本。单机部署版本可以帮助开发人员快速开发以及测试FATE。该版本支持两种类型：1）Docker；2）手动编译。

具体细节请参阅单机部署指南：\ `standalone-deploy <./部署/FATE单机部署指南.rst/>`__\ 。

集群部署
''''''''

FATE同样为大数据场景提供了分布式运行部署架构版本。从单机部署迁移到集群部署仅需要更改配置文件，不需要更改算法。

具体细节请参阅集群部署指南：\ `cluster-deploy <./部署/FATE-Cluster-step-by-step部署指南.rst>`__\ 。

KubeFATE部署:
~~~~~~~~~~~~~

通过 KubeFATE, 我们可以使用 docker-compose或者 Kubernetes方式部署FATE:

-  如果是开发或者测试场景, 推荐使用docker-compose部署方式.
   这种模式仅仅需要 Docker 环境。 更多细节请参考 `FATE Docker
   Compose部署 <https://github.com/FederatedAI/KubeFATE/tree/master/docker-deploy>`__.

-  如果生产环境或者大规模部署, 推荐使用Kubernetes方式来管理FATE系统
   。更多细节请参考\ `FATE
   Kubernetes部署 <https://github.com/FederatedAI/KubeFATE/blob/master/k8s-deploy>`__.

更多使用说明请见\ `KubeFATE <https://github.com/FederatedAI/KubeFATE>`__\ 。

运行测试
--------

./federatedml/test 文件夹中提供了所有单元测试的脚本。

安装FATE后，可以使用以下命令运行测试：

   sh ./federatedml/test/run_test.sh

如果FATE被正确安装，那么所有单元测试都将成功通过。

示例程序
--------

快速开始
~~~~~~~~

我们提供了一个用于快速搭建训练任务的python脚本作为示例。该脚本位于：FATE/examples/federatedml-1.x-examples

获取模型并检查结果
~~~~~~~~~~~~~~~~~~

FATE提供了名为 fate-flow
的工具用来跟踪组件输出模型或日志。fate-flow的部署和使用可以在
`这里 <./FATE-Flow/README.rst>`__ 找到。

文档资料
--------

API 文档
~~~~~~~~

FATE在 `doc-api <https://github.com/FederatedAI/FATE/tree/develop-1.4/doc/api/>`__ 文件夹中提供了API文档，包括 federatedml,
eggroll, federation. 

开发者文档
~~~~~~~~

如何使用FATE开发联邦学习算法？您可以在
`开发指南 <./Federatedml/开发指南.rst>`__ 中查看FATE开发指南。

其他文档
~~~~~~~~

FATE还在 `doc <https://github.com/FederatedAI/FATE/tree/develop-1.4/doc/>`__
中提供了许多其他文档。这些文档可以帮助您更好地了解FATE。

参与到FATE开源社区
------------------

-  加入我们的邮件列表 `Fate-FedAI Group
   IO <https://groups.io/g/Fate-FedAI>`__\ ，您可以提出问题或参与讨论。

-  对于常见问题, 我们为您提供了
   `FAQ文档 <https://github.com/WeBankFinTech/FATE/wiki>`__\ 。

-  请使用 `issues <https://github.com/WeBankFinTech/FATE/issues>`__
   提交BUG。

-  请使用 `pull
   requests <https://github.com/WeBankFinTech/FATE/pulls>`__
   提交、贡献代码。

中文文档有奖征集活动
------------------
`活动详情 <./有奖征集活动/README.md>`__

License
~~~~~~~

`Apache License 2.0 <LICENSE>`__

.. |License| image:: https://img.shields.io/badge/License-Apache%202.0-blue.svg
   :target: https://opensource.org/licenses/Apache-2.0
.. |CodeStyle| image:: https://img.shields.io/badge/Check%20Style-Google-brightgreen
   :target: https://checkstyle.sourceforge.io/google_style.html
.. |Style| image:: https://img.shields.io/badge/Check%20Style-Black-black
   :target: https://checkstyle.sourceforge.io/google_style.html
