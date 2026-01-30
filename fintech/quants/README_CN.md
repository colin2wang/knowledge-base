# Awesome Quants 中文版

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 中文量化金融相关资源精选列表

**来源：** [thuquant/awesome-quant](https://github.com/thuquant/awesome-quant)

**语言：** 中文 | [English Version](README.md)

---

## 目录

- [数据源](#数据源)
- [数据库](#数据库)
- [量化交易平台](#量化交易平台)
- [策略](#策略)
- [回测](#回测)
- [交易API](#交易api)
- [编程](#编程)
  - [Python](#python)
  - [R](#r)
  - [C++](#c)
  - [Julia](#julia)
- [论坛](#论坛)
- [书籍](#书籍)
- [论文](#论文)
- [政策](#政策)
- [值得关注的信息源](#值得关注的信息源)
- [其他Quant资源索引](#其他quant资源索引)

## 数据源

- [TuShare](http://tushare.org/) - 中文财经数据接口包
- [Quandl](https://www.quandl.com/) - 国际金融和经济数据
- [Wind资讯-经济数据库](http://www.wind.com.cn/NewSite/edb.html) - 收费
- [锐思数据](http://www.resset.cn/) - 收费
- [国泰安数据服务中心](http://www.gtarsc.com/Home) - 收费
- [恒生API](https://open.hscloud.cn/cloud/open/apilibrary/queryLibraryMenu.html?parent_id=100313&menu_id=100307) - 收费
- [Bloomberg API](https://www.bloomberglabs.com/api/libraries/) - 收费
- [数库金融数据API](http://developer.chinascope.com/) - 收费
- [Historical Data Sources](http://quantpedia.com/Links/HistoricalData) - 数据源索引
- [Python通达信数据接口](https://github.com/rainx/pytdx) - 免费通达信数据源
- [fooltrader](https://github.com/foolcage/fooltrader) - 大数据开源量化项目，全市场数据源
- [zvt](https://github.com/zvtvz/zvt) - 中低频多级别全市场分析和交易框架
- [JoinQuant/jqdatasdk](https://github.com/JoinQuant/jqdatasdk) - 聚宽金融数据SDK
- [米筐RQData](https://www.ricequant.com/introduce_rqdata) - 收费
- [AkShare](https://github.com/jindaxiang/akshare) - 免费开源财经数据接口库

## 数据库

- [manahl/arctic](https://github.com/manahl/arctic) - 基于MongoDB和Python的高性能时间序列和tick数据存储
- [kdb](https://kx.com/) - 高性能金融序列数据库解决方案
- [MongoDB时间序列](http://blog.mongodb.org/post/65517193370/schema-design-for-time-series-data-in-mongodb) - MongoDB时间序列数据存储方案
- [InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) - Go编写的分布式时间序列数据库
- [OpenTSDB](https://github.com/OpenTSDB/opentsdb) - 基于HBase的时间序列数据库
- [kairosdb](https://github.com/kairosdb/kairosdb) - 基于Cassandra的时间序列数据库
- [timescaledb](https://github.com/timescale/timescaledb) - 基于PostgreSQL的时间序列数据库

## 量化交易平台

- [JoinQuant聚宽](https://www.joinquant.com/) - 基于Python的在线量化交易平台
- [优矿](https://uqer.io/home/) - 基于Python的在线量化交易平台
- [Ricequant](https://www.ricequant.com/) - 支持Python和Java的在线量化交易平台
- [掘金量化](http://www.myquant.cn/) - 支持多语言的量化交易平台
- [Auto-Trader](http://www.atrader.com.cn/portal.php) - 基于MATLAB的量化交易平台
- [MultiCharts中国版](https://www.multicharts.cn/) - 程序化交易软件
- [BotVS](https://www.botvs.com/) - 支持期货、股票、数字货币的量化平台
- [Tradeblazer](http://www.tradeblazer.net/) - 期货程序化交易软件平台
- [MetaTrader 5](https://www.metatrader5.com/en) - 多资产交易平台
- [BigQuant](https://bigquant.com) - 人工智能/机器学习量化投资平台
- [天勤量化TqSdk](https://github.com/shinnytech/tqsdk-python) - 免费期货、期权、股票数据，支持实盘交易

## 策略

- [JoinQuant聚宽学习资料](https://xueqiu.com/8287840120/65009358) - 量化学习资料、经典交易策略、Python入门
- [掘金策略集锦](https://github.com/myquant/strategy) - 掘金策略集锦
- [优矿社区内容索引](https://uqer.io/community/share/58243e7d228e5b91df6d5d19) - 优矿社区内容索引
- [RiceQuant优秀策略汇总](https://www.ricequant.com/community/topic/1863//3) - 2016年4月以来优秀策略与研究汇总
- [雪球选股](https://xueqiu.com/9796081404) - 雪球选股
- [botvs策略](https://github.com/botvs/strategies) - JavaScript和Python量化交易策略

## 回测

- [Zipline](https://github.com/quantopian/zipline) - Python回测框架
- [pyalgotrade](https://github.com/gbeced/pyalgotrade) - Python事件驱动回测框架
- [pyalgotrade-cn](https://github.com/Yam-cn/pyalgotrade-cn) - 支持A股历史行情回测的Pyalgotrade
- [rqalpha](https://github.com/ricequant/rqalpha) - Ricequant开源的Python回测引擎
- [quantdigger](https://github.com/QuantFans/quantdigger) - 基于Python的量化回测框架
- [pyktrader](https://github.com/harveywwu/pyktrader) - 基于pyctp和vnpy的Python交易平台
- [QuantConnect/Lean](https://github.com/QuantConnect/Lean) - QuantConnect算法交易引擎
- [QUANTAXIS](https://github.com/yutiansut/QUANTAXIS) - 中小型策略团队解决方案
- [Hikyuu](http://hikyuu.org) - 基于Python/C++的开源量化交易研究框架
- [StarQuant](https://github.com/physercoe/starquant) - 基于Python/C++的综合量化交易回测系统

## 交易API

- [上海期货信息技术CTP API](http://www.sfit.com.cn/5_2_DocumentDown.htm) - 期货交易所API
- [飞马快速交易平台](http://www.cffexit.com.cn/static/3000201.html) - 飞马
- [大连飞创信息技术](http://www.dfitc.com.cn/portal/cate?cid=1364967839100#1) - 飞创
- [vnpy](https://github.com/vnpy/vnpy) - 基于Python的开源交易平台开发框架
- [QuantBox/XAPI2](https://github.com/QuantBox/XAPI2) - 统一行情交易接口第2版
- [easytrader](https://github.com/shidenggui/easytrader) - 券商自动程序化交易组件
- [策略易](http://www.iguuu.com/e) - 交易客户端管理，HTTP RESTful API
- [IB API](https://www.interactivebrokers.com.hk/cn/index.php?f=5234&ns=T) - 盈透证券交易API
- [富途量化API](https://github.com/FutunnOpen/futuquant) - 富途量化平台API


## 编程

### Python

#### 安装

- [Anaconda](https://www.continuum.io/downloads) - 推荐通过[清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)下载安装
- [Christoph Gohlke预编译包](http://www.lfd.uci.edu/~gohlke/pythonlibs/) - Windows用户Python库预编译包

#### 教程

- [Codecademy Python](https://www.codecademy.com/learn/python) - Codecademy Python课程
- [用Python玩转数据](https://www.coursera.org/learn/hipython) - 南京大学Coursera课程
- [Python数据分析入门](https://www.coursera.org/learn/python-data-analysis) - 密歇根大学Coursera课程
- [Python官方教程](https://docs.python.org/3/tutorial/) - Python 3.5.2官方文档
- [Python for Finance](https://book.douban.com/subject/25921015/) - 金融Python编程书籍
- [Algorithmic Thinking](https://www.coursera.org/learn/algorithmic-thinking-1) - Python算法思维训练

#### 库

- [awesome-python](https://github.com/vinta/awesome-python) - 精选Python框架、库和资源
- [pandas](http://pandas.pydata.org) - Python数据分析基础
- [pyql](https://github.com/enthought/pyql) - Cython QuantLib封装
- [ffn](http://pmorissette.github.io/ffn/quick.html) - 绩效评估
- [TA-Lib](https://github.com/mrjbq7/ta-lib) - 技术指标
- [StatsModels](http://statsmodels.sourceforge.net/) - 常用统计模型
- [arch](https://github.com/bashtage/arch) - 时间序列
- [pyfolio](https://github.com/quantopian/pyfolio) - 组合风险评估
- [flint](https://github.com/twosigma/flint) - Apache Spark时间序列库
- [PyFlux](https://github.com/RJT1990/pyflux) - Python时间序列建模  

### R

#### 安装

- [CRAN镜像](https://mirrors.tuna.tsinghua.edu.cn/CRAN/) - 国内清华镜像下载安装
- [RStudio](https://www.rstudio.com/products/rstudio/download/) - R常用开发平台

#### 教程

- [R编程入门](https://www.datacamp.com/courses/free-introduction-to-r) - DataCamp在线学习
- [R Programming](https://www.coursera.org/learn/r-programming) - 约翰霍普金斯大学Coursera课程
- [计算金融与R](https://www.datacamp.com/community/open-courses/computational-finance-and-financial-econometrics-with-r) - 用R进行计算金融分析

#### 库

- [CRAN金融任务视图](https://cran.r-project.org/web/views/Finance.html) - CRAN官方R金融相关包整理
- [awesome-R](https://github.com/qinwf/awesome-R) - 精选R包、框架和软件

### C++

#### 教程

- [C++程序设计](http://www.xuetangx.com/courses/course-v1:PekingX+04831750.1x+2015T1/about) - 北京大学郭炜
- [基于Linux的C++](http://www.xuetangx.com/courses/course-v1:TsinghuaX+20740084X+sp/about) - 清华大学乔林
- [面向对象程序设计C++](http://www.xuetangx.com/courses/course-v1:TsinghuaX+30240532X+sp/about) - 清华大学徐明星
- [C++设计模式与衍生品定价](https://book.douban.com/subject/1485468/) - C++设计模式
- [C++参考文档](http://en.cppreference.com/w/cpp) - 在线文档

#### 库

- [awesome-cpp](https://github.com/fffaraz/awesome-cpp) - 精选C/C++框架、库和资源
- [awesome-modern-cpp](https://github.com/rigtorp/awesome-modern-cpp) - 现代C++资源集合
- [QuantLib](http://quantlib.org/index.shtml) - 免费开源量化金融库
- [libtrading](https://github.com/libtrading/libtrading) - 超低延迟交易连接库

### Julia

#### 教程

- [Learning Julia](http://julialang.org/learning/) - 官方学习资源
- [量化经济学与Julia](http://quant-econ.net/_static/pdfs/jl-quant-econ.pdf) - 诺贝尔经济学奖得主Thomas Sargent的Julia应用

#### 库

- [Quantitative Finance in Julia](https://github.com/JuliaQuant) - Julia量化金融库集合

### 编程论坛

- [Stack Overflow](http://stackoverflow.com/) - 对应语言标签
- [SegmentFault](https://segmentfault.com/) - 对应语言标签

### 编程能力在线训练

- [HackerRank](https://www.hackerrank.com/domains) - 包含多种语言和技术的编程挑战
- [LeetCode](https://leetcode.com/) - 多种编程语言在线编程训练

## 论坛

- [Quantitative Finance StackExchange](http://quant.stackexchange.com/) - StackExchange量化金融论坛
- [JoinQuant社区](https://www.joinquant.com/community) - JoinQuant社区
- [优矿社区](https://uqer.io/community/list) - 优矿社区
- [RiceQuant社区](https://www.ricequant.com/community/) - RiceQuant量化社区
- [掘金量化社区](http://forum.myquant.cn/) - 掘金量化社区
- [清华大学金融论坛](http://forum.thuquant.com/) - 清华大学学生金融数据与量化投资协会

## 书籍
* [My Life as a Quant: Reflections on Physics and Finance](http://www.amazon.com/My-Life-Quant-Reflections-Physics/dp/0470192739) - In My Life as a Quant, Emanuel Derman relives his exciting journey as one of the first high-energy particle physicists to migrate to Wall Street.
* [量化交易](https://book.douban.com/subject/25878150/) - Ernest P. Chan撰写的量化投资理论
* [量化投资与对冲基金丛书：波动率交易](https://book.douban.com/subject/25711100/)
* [Following the Trend](https://book.douban.com/subject/19990593/)
* [Statistical Inference](https://book.douban.com/subject/1464795/) - 统计推断入门
* [All of Nonparametric Statistics](https://book.douban.com/subject/4251603/) - 非参统计入门
* [The Elements of Statistical Learning](https://book.douban.com/subject/3294335/) -  Data Mining, Inference, and Prediction
* [Analysis of Financial Time Series](https://book.douban.com/subject/4719140/) - Ruey S. Tsay  的时间序列分析
* [Options, Futures, and Other Derivatives](https://book.douban.com/subject/6127888/) - 期权期货等衍生品



## 论文
* [awesome-quant/papers.md](https://github.com/thuquant/awesome-quant/blob/master/papers.md)

## 值得关注的信息源
* [Quantitative Finance arxiv](https://arxiv.org/archive/q-fin)
* [雪球工程师1号](http://xueqiu.com/engineer) - 财经社交网络雪球的量化相关账号。
* [Ricequant量化](http://xueqiu.com/ricequant) - Ricequant量化平台的雪球账号。
* [量化哥-优矿Uqer](http://xueqiu.com/4105947155) - 优矿Uqer量化平台的雪球账号。
* [宽客 (Quant) - 索引 - 知乎](https://www.zhihu.com/topic/19557481)
* 量化投资与机器学习 - 微信公众号
* THU量协 - 微信公众号
* 优矿量化实验室  - 微信公众号
* Ricequant   - 微信公众号
* 鲁明量化全视角 - 微信公众号


## 政策
* [中国证券监督管理委员会](http://www.csrc.gov.cn/pub/newsite/)
* [考试报名-中国证券业协会](http://www.sac.net.cn/cyry/kspt/ksbm/) - 证券从业资格报名
* [中国证券投资基金业协会](http://www.amac.org.cn/) - 内有相关法规教育和从业资格报名入口
* [大连商品交易所](http://www.dce.com.cn/)
* [上海期货交易所首页](http://www.shfe.com.cn/)
* [郑州商品交易所网站](http://www.czce.com.cn/portal/index.htm)
* [上海证券交易所](http://www.sse.com.cn/)
* [深圳证券交易所](http://www.szse.cn/)

# 其他Quant资源索引

* [Quantitative Finance Reading List - QuantStart](https://www.quantstart.com/articles/Quantitative-Finance-Reading-List#general-quant-finance-reading)
* [Master reading list for Quants, MFE (Financial Engineering) students | QuantNet Community](https://www.quantnet.com/threads/master-reading-list-for-quants-mfe-financial-engineering-students.535/)

# 其他 Awesome 列表
* 英文版 awesome-quant [wilsonfreitas/awesome-quant: A curated list of insanely awesome libraries, packages and resources for Quants (Quantitative Finance)](https://github.com/wilsonfreitas/awesome-quant)
* Other awesome lists [awesome-awesomeness](https://github.com/bayandin/awesome-awesomeness).
* Even more lists [awesome](https://github.com/sindresorhus/awesome).
* Another list? [list](https://github.com/jnv/lists).
* WTF! [awesome-awesome-awesome](https://github.com/t3chnoboy/awesome-awesome-awesome).
* Analytics [awesome-analytics](https://github.com/onurakpolat/awesome-analytics).
