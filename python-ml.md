- #1.机器学习三要素
  ##1.1 样本
  交叉验证
  - [Sklearn-train_test_split随机划分训练集和测试集](http://blog.csdn.net/cherdw/article/details/54881167)

  ##1.2 特征
  数据处理

  ##1.3 模型
  模型选择(训练目标、优化算法)、模型调参、模型融合
  ###1.3.1 xgboost
  - [xgboost参数](http://xgboost.readthedocs.io/en/latest/parameter.html)([汉](http://blog.sina.com.cn/s/blog_9132d85b0102w65l.html))
  - [论XGBOOST科学调参](https://zhuanlan.zhihu.com/p/25308120)
  - [xgbfi](https://github.com/limexp/xgbfir)  xgboost特征交互和重要度。里面提到用Gain（特征增益）和Cover（分裂方向的概率）构造出期望增益ExpectedGain，相比于分裂次数更能体现特征的实际作用。另外考虑了如何评估多个特征之间的影响，就是计算一条路径的增益。不过工具还不完善，对评估指标本身也没有进行详细解释。

  #2.工具箱

  ##2.1 Python基础

  - [Python重新加载模块方法](https://segmentfault.com/a/1190000004215928)

  ##2.2 Pandas数据处理

  - Pandas基础
    - [Python数据分析之pandas学习](http://www.cnblogs.com/nxld/p/6058591.html)
    - [2. pandas入门](http://pda.readthedocs.io/en/latest/chp5.html)
    - [Pandas:如何使用应用函数到多列](http://www.developerq.com/article/1499171945)
  - Pandas画图
    - [用Pandas作图](http://cloga.info/python/2014/02/23/Plotting_with_Pandas/) （注：记得调用 plt.show() 显示图）介绍了如何用pandas画图
    - [pandas 画图api](http://pandas.pydata.org/pandas-docs/stable/api.html#api-dataframe-plotting)   [Visualization](http://pandas.pydata.org/pandas-docs/stable/visualization.html) 这两个是官方对画图功能的介绍,Visualization里面的东西很多，值得学习
    - df.describe()  可以看数据的一些基础信息：数据量、最大(小)值、均值、上下四分位数、中位数
    - df.hist()  直方图，可以看数据的分布
    - df.plot(x='boys', y='girls', kind='scatter')  可以画散点图，直观的看数据之间的关系
    - df.corr()  可以计算数据之间的相关度

  ##2.3 SKI