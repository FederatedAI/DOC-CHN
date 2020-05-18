此处会列出算法中DataIO模块的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q: 如何查看dtable数据**
   | A:
     可以通过dtable的collect接口把所有数据拉到本地，也可以调用first接口，直接查看第一条。详情请查看doc下的\ `session文档 <https://github.com/FederatedAI/FATE/tree/master/doc/api/session_api.md>`__.
     如果table里面是instance需要查看详细数据，需要调取.features查看，例如：table.first()[1].features.

2. | **Q: 上传数据中的列名有什么规范呢？**
   | A:
     以指定第几列是y，详情可参考dataio的配置文件，但是第一列必须是id，id可以是这些类型:[‘float’,‘float64’,‘int’,‘int64’,‘str’,‘long’]

3. | **Q:
     请问fate目前支持的最大规模是多少，使用中会有超过10万样本dataio就会报错的情况**
   | A: 这个10w应该不会报错的，数据量应该和机器的配置强相关。
     底层基于分布式的引擎eggroll 上传之后是个分布式的kv表
     Partition指分区数 数据集大的时候要适当调大partition

4. | **Q: 上传数据中的列名有什么规范呢？**
   | A:
     以指定第几列是y，详情可参考dataio的配置文件，但是第一列必须是id，id可以是这些类型:[‘float’,‘float64’,‘int’,‘int64’,‘str’,‘long’]

5. | **Q: 数据的数量有什么限制么？**
   | A:
     理论上是没有要求的，下限一般不要少于10条，上限则和机器配置有关，数据量越多，机器要求越高。

6. | **Q: 可以接入外数据源吗？**
   | A: 目前只能支持通过fate_flow上传数据。

7. | **Q:
     这个配置中的name和namespace和上传数据的时候，那2个参数是需要对应的吗？**
   | A: 是的。
