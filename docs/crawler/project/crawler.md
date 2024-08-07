# 爬虫

## 数据爬取

* 从任意音乐网站中爬取歌曲和歌手信息。
    * 例如：网易云音乐、QQ音乐、酷我音乐等。
* 歌曲需爬取以下信息：
    * 歌曲名
    * 歌手名
    * 歌词（去除时间轴，仅保留歌词文本）
    * 歌曲图片（如专辑封面）
    * 歌曲原始网站URL
    * 可选但不要求：评论（每首歌曲约3条评论，包含文本、评论时间）
    * 其它可能需要的信息
* 歌手需爬取以下信息：
    * 歌手名
    * 歌手图片
    * 歌手简介
    * 歌手原始网站URL
    * 其他可能需要的信息

## 相关要求

* 爬取的歌曲数量不少于2000，歌手数量不少于100，每位歌手至少需爬取一首歌曲。
* 请勿爬取音视频内容！

## 注意事项

* 注意网站可能存在的反爬机制，并调整爬虫策略
    * 不要野蛮抓取、两次爬取中间建议停顿一定时间（建议两次爬取之间设置一定时间间隔，如1秒）
    * 设置与常规访问一致的 HTTP 头
    * 使用动态 User-Agent
    * 更换不同的 Cookie
    * 更换不同的 IP 地址
    * （有条件）使用代理池进行请求
* 提示：建议在本地存储已爬到的页面URL和HTML文件，防止爬虫程序意外崩溃或被反爬激活后损失数据，并能有效地继续爬取。
* 如被反爬机制封禁，允许保留已有数据，换其他网站继续爬取。
* 不限制爬虫所使用的Python第三方库(bs4、Scrapy等)
