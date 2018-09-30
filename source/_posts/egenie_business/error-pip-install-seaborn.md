---
title: mac安装seaborn遇到的问题
tags:
  - python
  - mac
  - seaborn
categories:
  - 异常
abbrlink: 1950799952
date: 2017-11-30 07:58:00
---
# 异常信息
```
pip install numpy

MacBook-Pro:~ victor$ pip install seaborn
Collecting seaborn
  Downloading seaborn-0.8.1.tar.gz (178kB)
    100% |████████████████████████████████| 184kB 666kB/s
Collecting pandas (from seaborn)
  Using cached pandas-0.21.0-cp27-cp27m-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl
Requirement already satisfied: python-dateutil in /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python (from pandas->seaborn)
Collecting numpy>=1.9.0 (from pandas->seaborn)
  Using cached numpy-1.13.3-cp27-cp27m-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl
Requirement already satisfied: pytz>=2011k in /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python (from pandas->seaborn)
Installing collected packages: numpy, pandas, seaborn
  Found existing installation: numpy 1.8.0rc1
    DEPRECATION: Uninstalling a distutils installed project (numpy) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
    Uninstalling numpy-1.8.0rc1:
Exception:
Traceback (most recent call last):
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/basecommand.py", line 215, in main
    status = self.run(options, args)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/commands/install.py", line 342, in run
    prefix=options.prefix_path,
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_set.py", line 778, in install
    requirement.uninstall(auto_confirm=True)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_install.py", line 754, in uninstall
    paths_to_remove.remove(auto_confirm)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_uninstall.py", line 115, in remove
    renames(path, new_path)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/utils/__init__.py", line 267, in renames
    shutil.move(old, new)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 302, in move
    copy2(src, real_dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 131, in copy2
    copystat(src, dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 103, in copystat
    os.chflags(dst, st.st_flags)
OSError: [Errno 1] Operation not permitted: '/var/folders/sk/hl26sn7n1pg9jrzgzqydvfq00000gn/T/pip-gOIIvZ-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/numpy-1.8.0rc1-py2.7.egg-info'
```

# 发现卸载这个numpy都不行
```
sudo -H pip uninstall  numpy
DEPRECATION: Uninstalling a distutils installed project (numpy) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
Uninstalling numpy-1.8.0rc1:
  /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/numpy-1.8.0rc1-py2.7.egg-info
Proceed (y/n)? y
Exception:
Traceback (most recent call last):
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/basecommand.py", line 215, in main
    status = self.run(options, args)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/commands/uninstall.py", line 76, in run
    requirement_set.uninstall(auto_confirm=options.yes)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_set.py", line 346, in uninstall
    req.uninstall(auto_confirm=auto_confirm)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_install.py", line 754, in uninstall
    paths_to_remove.remove(auto_confirm)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_uninstall.py", line 115, in remove
    renames(path, new_path)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/utils/__init__.py", line 267, in renames
    shutil.move(old, new)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 302, in move
    copy2(src, real_dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 131, in copy2
    copystat(src, dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 103, in copystat
    os.chflags(dst, st.st_flags)
OSError: [Errno 1] Operation not permitted: '/tmp/pip-Ez2DTF-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/numpy-1.8.0rc1-py2.7.egg-info
```

# 解决方案
https://blog.wizchen.com/2016/06/17/Mac%E4%B8%8B%E6%9B%B4%E6%96%B0python%E7%A7%91%E5%AD%A6%E8%AE%A1%E7%AE%97%E5%BA%93numpy/

- 说是System Integrity Protection的问题，解决的办法是关闭SIP:

- 重启电脑，电脑启动的时候安住 command + R
- 等画面上出现 apple logo 的，你会看到 OS X 工具程式的窗口，选择终端，等待终端打开，直接输入csrutil disable，回车后重启即可
- 重启完毕后，再次在终端输入

# 最后成功
```
pip install -U numpy
Collecting numpy
  Using cached numpy-1.13.3-cp27-cp27m-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl
Installing collected packages: numpy
  Found existing installation: numpy 1.8.0rc1
    DEPRECATION: Uninstalling a distutils installed project (numpy) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
    Uninstalling numpy-1.8.0rc1:
Exception:
Traceback (most recent call last):
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/basecommand.py", line 215, in main
    status = self.run(options, args)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/commands/install.py", line 342, in run
    prefix=options.prefix_path,
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_set.py", line 778, in install
    requirement.uninstall(auto_confirm=True)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_install.py", line 754, in uninstall
    paths_to_remove.remove(auto_confirm)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/req/req_uninstall.py", line 115, in remove
    renames(path, new_path)
  File "/Library/Python/2.7/site-packages/pip-9.0.1-py2.7.egg/pip/utils/__init__.py", line 267, in renames
    shutil.move(old, new)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 302, in move
    copy2(src, real_dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 131, in copy2
    copystat(src, dst)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/shutil.py", line 103, in copystat
    os.chflags(dst, st.st_flags)
OSError: [Errno 1] Operation not permitted: '/var/folders/sk/hl26sn7n1pg9jrzgzqydvfq00000gn/T/pip-rfPgIG-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/numpy-1.8.0rc1-py2.7.egg-info'
shengsiyudeMacBook-Pro:~ victor$ sudo -H pip install -U numpy
Password:
Collecting numpy
  Using cached numpy-1.13.3-cp27-cp27m-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl
Installing collected packages: numpy
  Found existing installation: numpy 1.8.0rc1
    DEPRECATION: Uninstalling a distutils installed project (numpy) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
    Uninstalling numpy-1.8.0rc1:
      Successfully uninstalled numpy-1.8.0rc1
Successfully installed numpy-1.13.3
```