> 前言：看到好看的可爱的小姐姐想要一个个保存下载到本地然后慢慢欣赏或者珍藏？不如学会爬虫解放你的双手吧！今天带大家来爬取老司机最喜欢逛的一个网站！咳咳咳！不是你们想的那种网站！是正经的养眼网站！

![image](https://user-images.githubusercontent.com/88499526/191899598-518a6fce-22f7-4f10-aec2-4c3bef7f8322.png)


源码地址：[python爬取某网站上的图片2_梦里逆天的博客-CSDN博客](https://blog.csdn.net/username666/article/details/125799723?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-4-125799723-blog-113725731.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-4-125799723-blog-113725731.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=6)
源码作者：[梦里逆天的博客_CSDN博客-学习笔记,PHP,Python领域博主](https://blog.csdn.net/username666)
改进升级：[图欧学习资源库](https://tuostudy.com/)
新增功能：以相册名+固定位数为2的数字作为图片文件名，可以支持输入最大页数，实现多页爬取，修复了部分BUG
实现过程：增加了自动创建images文件夹的功能，用以保存图片；增加了一个for循环，把整个爬取过程定义为一个download()方法，嵌入到for循环，这样子就可以实现循环爬取了

爬取对象：https://www.jdlingyu.com/tag/%e5%b0%91%e5%a5%b3（tag为少女）
![image](https://user-images.githubusercontent.com/88499526/191899666-dcf09aa6-9392-46b2-ac14-5da01f14a0dd.png)


特别说明：如果你想要爬完所有图片，有多少页数你就输入多少页数，比如15页，你就输入15
![\[图片\]](https://img-blog.csdnimg.cn/04cc5ea446dc4f56b3f668f5a96dd3e5.png)

完整代码：

```python
"""
爬虫：模拟客户端<浏览器,app应用>批量请求服务器数据

爬虫数据采集的一般步骤：
1.找数据对应的链接地址
2.发送指定地址请求，请求数据
3.数据提取（提取需要的数据）
4.数据保存
"""
import requests  # 数据请求，第三方模块：pip install requests
import parsel  # 数据解析模块，第三方模块：pip install parsel
import os  # 内置模块，文件目录操作的模块

page = input('请输入你想要爬取的起始页数（从第一页开始则输入1）：')  # 爬取起始页数，从第一页开始则输入1

page_max = input('请输入你想要爬取的最大页数（尾页为15则输入15）：')  # 如果你想要爬完所有图片，有多少页数你就输入多少页数

os.mkdir('images')  # 自动创建images文件夹用以保存图片


def download():
    # 1.找数据对应的链接地址
    url = "https://www.jdlingyu.com/tag/%e5%b0%91%e5%a5%b3/page/" + str(page)   # 默认tag是“少女”，你也可以选择别的tag

    # user-agent：浏览器的身份标识
    headers = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'}

    # 2.发送指定地址请求，请求数据
    response = requests.get(url, headers=headers)
    # text 获取对象里面的文本数据
    html_str = response.text
    # print(html_str)

    # 3.数据提取（提取需要的数据）  html数据, xpath提取 parsel
    # parsel库可以解析HTML和XML，并支持使用Xpath和CSS选择器对内容进行提取和修改，同时融合了正则表达式的提取功能
    selector = parsel.Selector(html_str)  # 转换数据类型

    # 提取所有的li标签
    lis = selector.xpath('//div[@id="post-list"]/ul/li')

    for li in lis:
        # 获取标题
        pic_title = li.xpath('.//h2/a/text()').get()  # 相册标题，用于保存相册的文件夹名
        # 获取相册链接
        pic_url = li.xpath('.//h2/a/@href').get()  # 相册链接
        # print(pic_title, pic_url)
        print(f"正在下载相册：{pic_title}")

        dir_name = 'images/' + ''.join(['' if x in ['\\', '/', ':', '*', '?', '"', '<', '>', '|'] else x for x in pic_title])
        # 创建相册文件夹（当出现特殊符号将会替换成空格以防创建失败导致报错）
        # 判断该文件夹是否存在
        if not os.path.exists(dir_name):
            # 不存在则创建
            os.mkdir(dir_name)

        # 发送相册详情页地址请求
        response_pic = requests.get(url=pic_url, headers=headers).text  # 详情页数据

        # 解析详情页中的图片地址
        selector_2 = parsel.Selector(response_pic)
        # 获取所有链接
        pic_url_list = selector_2.xpath('//div[@class="entry-content"]//img/@src').getall()  # 所有图片链接

        num = 1
        # 遍历每一个图片链接
        # noinspection PyBroadException
        try:
            for pic_url in pic_url_list:
                # 发送图片链接请求，获取图片数据，图片数据是二进制数据
                # content 提取二进制数据
                img_data = requests.get(url=pic_url, headers=headers).content

                # 4.数据保存
                # 4.1 准备图片的文件名

                file_name = dir_name[7:] + str(num).zfill(2) + '.jpg'  # 以相册名+固定位数为2的数字作为文件名
                num += 1

                # 4.2 保存图片
                # w-->write b-->binary
                with open(dir_name + '/' + file_name, mode='wb') as f:
                    f.write(img_data)
                    print('保存完成：', file_name)
        except Exception as e:
            pass
        continue


for a in range(1, int(page_max)):
    download()
    page += 1
```

运行效果：
![\[图片\]](https://img-blog.csdnimg.cn/360dacc8251248c6b430b95e9f895b3a.png)


本地效果：
![\[图片\]](https://img-blog.csdnimg.cn/e59bae2c7212447eabfa2c439f3c586d.png)

爬取结果：
![image](https://user-images.githubusercontent.com/88499526/191899755-0205b0b3-0da7-49a9-b414-59b4a3a5e350.png)


中途翻车：
爬取的过程中出现了点意外，如果网站图片无法成功加载就会报错，运行自动结束
![\[图片\]](https://img-blog.csdnimg.cn/39f0b80317f94e58b65c14452e4753ed.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/06931d16b0314bdba2b6d617d7906eb6.png)

![\[图片\]](https://img-blog.csdnimg.cn/4f1897c173934b01a45413962f7a22ad.png)

好吧……遇到这种问题那就没办法了……只能怪自己来晚了吧……


处理方法：然后我在想，能不能遇到运行报错则自动跳过呢？于是我尝试在代码中间的各种位置加try…except代码，不行，还是报错，然后在在for循环内加了try…except代码，还是不行，爬取进入了死循环了，都以失败告终。最后我终于到找到了原因所在！是在【发送图片链接请求，获取图片数据的环节】出现了问题，只要将这个for循环整体嵌套进try…except代码即可！最终完美解决问题！

结尾：
OK，完整代码我都放在上面了，你们安装好python和pycharm就能够运行，如果你是初学者的话，具体教程可自行去CSDN搜索学习~当然，我的代码还有很多不足的地方，欢迎各位大佬帮忙修改与指点！
如果你懒得自己爬取，也可以直接用我爬取的资源即可，完整的图片我全都放在了网盘上，你们点下面的链接即可全部抱走！

> 「绝对领域站点小姐姐美少女全套写真图集共357套（夸克首发 全网独家）」
链接：https://pan.quark.cn/s/bcdf5933abc2
