## -豆瓣Top250数据爬取及可视化分析-
本项目使用的语言为Python3, 用到的几个模块有：BeautifulSoup（爬数据），pandas（数据处理），Echarts.js（可视化），WordCloud库生成词云，部分图表由Tableau生成。
- 获取数据：使用urllib库获取豆瓣页面，BeautifulSoup进行网页解析，正则表示式抽取内容，获得豆瓣电影排行数据；
- 存储数据：利用python的xlwt库将抽取的数据datalist写入Excel表格；
- 数据可视化：利用Echarts丰富的可视化图表进行爬取数据的分析、利用WorldCloud依照特定图片合成词云；
- 应用flask框架完成网站搭建并能够本地访问。
