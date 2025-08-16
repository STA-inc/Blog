最近用request编写了一个简单的爬虫下载全量网站数据，刚运行就每个url都报403错误。但是手动在浏览器端访问却是能够访问的。  
排查发现可能是服务器开启了反爬虫，针对这种情况添加headers浏览器头，模拟人工访问网站行为：  

```python
def download(url):
    filename = url.split('/')[-1]
    mk = "E:/存储下载的文件的地址"
    dest_dir = os.path.join(mk, filename)
    try:
        opener = urllib.request.build_opener()
        opener.addheaders = [('User-Agent',
                              'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1941.0 Safari/537.36')]
        urllib.request.install_opener(opener)
        urllib.request.urlretrieve(url, dest_dir)
    except Exception as e:
        print(e)
        print("is wrong ")
```
即可解决。  
