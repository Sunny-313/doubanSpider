## -豆瓣Top数据爬取及可视化分析-
***
#### 本项目使用的语言为Python3, 用到的几个模块有：BeautifulSoup（爬数据），pandas（数据处理），Echarts.js（可视化），WordCloud库生成词云，部分图表由Tableau生成。
- 获取数据：使用urllib库获取豆瓣页面，BeautifulSoup进行网页解析，正则表示式抽取内容，获得豆瓣电影排行数据；
- 存储数据：利用python的xlwt库将抽取的数据datalist写入Excel表格；
- 数据可视化：利用Echarts丰富的可视化图表进行爬取数据的分析、利用WorldCloud依照特定图片合成词云；
- 应用flask框架完成网站搭建并能够本地访问。
***
#### 数据获取
- 计划要抓取的字段包括：电影详情链接、图片链接、影片中文名、影片外国名、评分、评价数、概况、相关信息等
- 需要抓取的影片信息有250条，每页25部影片，一共有10页。简单浏览网页不难发现，翻页的链接不需要从页面底端抓取，直接修改url参数即可。
#### 数据分析
将清洗好的csv文件导入Tableau，下面是豆瓣电影TOP250上的制片国家/地区分布和各个语言所占的比重。比重越大，字体越大。类似的图表也可以用Python wordcloud来做。
在这里插入图片描述


榜单上的美国影片占了相当大的比重，其次是日本，然后才是中国大陆、中国香港和英国。
在这里插入图片描述

从制片国家/地区上不难推断，榜单上英语将会占很大的比重，其次是日语，然后是普通话和粤语。

下面是榜单上影片的年代分布，在Tableau中可以创建组来实现对上映时间的划分。

在这里插入图片描述

1990年之后上映的电影几乎占据了整个榜单的85%，这应该和电影技术的发展有关系。更早期的电影在数量、画面、主题、拍摄手法等方面上可能比较难征服现在的观众。



“豆瓣用户每天都在对“看过”的电影进行“很差”到“力荐”的评价，豆瓣根据每部影片看过的人数以及该影片所得的评价等综合数据，通过算法分析产生豆瓣电影 Top 250。”

以上摘自豆瓣。

下面简单分析一下哪些特征会榜单排名产生比较大的影响。

首先需要拿掉诸如链接，片名，导演等非量化字段。
metrics = data.drop(['link','director','Chinese_title','main_country_region','main_language'],axis=1)
增加/转化一些字段:

相较于平均评分，评价人数，可能增加一个总评分=avg_rating*num_of_ratings会更直接体现影片的质量。

上映年份可以转换成已经上映了多少年，即2019-year，在时间显得更直观。

metrics.eval('total_rating_scores = avg_rating*num_of_ratings', inplace=True)
metrics['years to 2019'] = metrics['year'].apply(lambda y:2019-y)
metrics.drop(['year'],axis=1,inplace=True)
生成相关系数和heatmap。

corr = metrics.corr()
plt.figure(figsize=(6,5), dpi=100)
sns.heatmap(corr,cmap='coolwarm',linewidths=0.5)
在这里插入图片描述

观察heatmap的第一行不难发现，rank和多个字段存在较高的相关性。下面看一下具体的相关系数：
corr.head(1)
在这里插入图片描述

新建的字段total_rating_scores相关系数最高，进一步做显著性检验：
#total_rating_scores t-test
x = list(metrics['rank'])
y = list(metrics['total_rating_scores'])
r,p = stats.pearsonr(x,y)
print(r)
print(p)
在这里插入图片描述

相关系数为-0.7，显著性水平小于0.001，说明豆瓣电影TOP250榜单的排名与影片得到总评分存在较强的相关性。总评分越高，排名越靠前。

类似的字段还有平均评分、评分人数、观看人数，而想看人数、短评数、长评数相关性相对较弱，与上映时间几乎没有相关性。

因此，想要影片挤进这份榜单，需要影片能够得到足够多的评分和较好的评价。

由于样本数量只有250个，加上豆瓣内部可能还有其他隐藏的特征，现有的数据可能很难构建出比较满意的模型。可以尝试爬取更多的数据并增加其他特征然后再来构建榜单排名的预测模型。

豆瓣上每部影片都有很多短评/长评，观察heatmap的第6、7行可以发现，平均评分与影片短评/长评的相关性较弱。这与我们平常看到的烂片常常反而能够引来热烈讨论的现象相一致。同样与影评数量相关性较弱的还有标记为想看的数量和影片上映的时间。

可以通过boxplot来进一步了解。代码如下：
comment_reviews = metrics.loc[:,['num_comment','num_reviews']]
def bin_years(value):
    for i in range(10,91,10):
        if value<i:
            return f'{i-9}-{i}'
comment_reviews['years_bin'] = metrics['years to 2019'].apply(bin_years)
bins = [f'{i-9}-{i}' for i in range(10,91,10)]
plt.figure(figsize=(12,5), dpi=100)
sns.boxplot(x='years_bin', y='num_comment', data=comment_reviews, order=bins)
plt.figure(figsize=(12,5), dpi=100)
sns.boxplot(x='years_bin', y='num_reviews', data=comment_reviews, order=bins)
在这里插入图片描述

在这里插入图片描述

豆瓣电影TOP250榜单上，除了最近10年的影片获得的影评相比较高以外，其他上映时间的电影得到的影评数量大体保持在同一水平。这也与这两组特征之间较低的相关系数相吻合。



