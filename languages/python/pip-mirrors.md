# PIP Mirrors Configuration

## Using PIP Mirrors in China

### Chinese Package Sources
```
Tsinghua University: https://pypi.tuna.tsinghua.edu.cn/simple
Alibaba Cloud: http://mirrors.aliyun.com/pypi/simple/
University of Science and Technology of China: https://pypi.mirrors.ustc.edu.cn/simple/
Huazhong University of Science and Technology: http://pypi.hustunique.com/
Shandong University of Technology: http://pypi.sdutlinux.org/
Douban: http://pypi.douban.com/simple/
Note: New versions of Ubuntu require HTTPS sources, please pay attention.
```

### Temporary Usage

You can add the -i parameter when using pip: https://pypi.tuna.tsinghua.edu.cn/simple
For example: pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspider, this will install the pyspider library from Tsinghua's mirror.

### Permanent Configuration (Recommended)

#### Linux
Modify ~/.pip/pip.conf (create the folder and file if it doesn't exist. The folder should start with "." to indicate it's a hidden folder)
Content as follows:
```shell script
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

#### Windows
Create a pip directory in the user directory, then create a new file pip.ini. (For example: C:\Users\WQP\pip\pip.ini) Content is the same as above.