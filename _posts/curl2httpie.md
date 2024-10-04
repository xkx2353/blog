---
title: curl转换httpie

date: 2022-04-29 12:56:36

tags: [curl httpie 效率]
---

### 写在前面

> 公司日常中大量使用curl，但是终端下curl的响应结果没有格式化，看的时候找字段很不直观，同时有时想把别的环境curl在本地调试时，需要手动修改curl中的请求域名，比较麻烦，所有才有想法做一个脚本快速处理这样琐碎的事情

1. 做了什么事情：将curl 转换成 httpie
1. 涉及到的工具库

### curl

> [curl](https://curl.se)

1. 对于curl的使用，有很多参数，看看文档还是很不错的，某些场景可能就一个参数就搞定了
2. 在shell中处理网络请求，对于参数的处理好像node里面的库更适合做这个事情，当然具体我也没有深度使用过

### httpie

> [httpie](https://httpie.io)

1. **a command-line HTTP client. Its goal is to make CLI interaction with web services as human-friendly as possible**
1. 具体细节看官方文档

### curlipie

> [curlipie](https://github.com/hongquan/CurliPie)

1. Python library to convert curl command to httpie

2. [pip](https://pypi.org/project/curlipie)

### pyperclip

> [pyperclip](https://pypi.org/project/pyperclip)

1. A cross-platform clipboard module for Python. (Only handles plain text for now.)

### 使用上述工具组装成功能脚本

- 通用脚本

```
#!/usr/local/opt/python@3.7/bin/python3.7
# 通用脚本，如何使用：
# 1.复制需要转换的curl文本
# 2.执行脚本，结果会保存到粘贴板中
# 3.终端粘贴执行即可
#------------------------------------------------------------------
# 准备工作：
# 1. `/usr/local/opt/python@3.7/bin/pip3.7 install pyperclip`
# 2. `/usr/local/opt/python@3.7/bin/pip3.7 install curlipie`
import pyperclip
from curlipie import curl_to_httpie

curl = pyperclip.paste();

curl = curl.replace("-data-binary","d").replace("--compressed \\","").replace("--insecure","").replace("iPhone","IPHONE");

result = curl_to_httpie(curl)

pyperclip.copy(result.httpie)

```

- 对于想转换一些curl中的参数，就使用常用的文本工具（sed，awk等）进行处理即可

### 思考

1. 工具的稳定性，比如python版本换一下可能安装不成功
2. curl转到httpie有些参数的兼容性可能还有问题，需要注意
3. 对于这一套想直接分享给同事使用，还是太麻烦了，对于不熟悉各种脚本的人，安装这，安装那，很容器搞错，所以我决定做个brew包，这样同事只需要执行一个命令即可在自己电脑上面使用（大家使用的都是mac）

---

### brew包做好了

1. 打开你的终端，执行 `brew tap azouever/tap;brew install azouever/tap/curl2httpie`即可

2. 包含两个命令

   1. `curl2httpie.py`
      1. 复制curl （copy）
      2. 执行命令
      3. 粘贴（paste）
      4. 回车看结果
   2. `curl_result_json.sh`
      1. 复制curl （copy）
      2. 执行命令（看结果）
