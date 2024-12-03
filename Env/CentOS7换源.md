#  前言

在国内当然是要用国内的源了

首先安装 wget

```
yum -y install wget
```

![image.gif](assets/0aa5a372653c467689816101d7d7d18c.gif)

# 一、备份

```
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

![image.gif](assets/48836cbb9595422f94831fa37d7f7dd3.gif)

# 二、**下载新的CentOS-Base.repo 到/etc/yum.repos.d/**

### **CentOS 5**

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
#或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
```

![image.gif](assets/f82e100408f04da29d8d7215145f97ea.gif)

### **CentOS 6**

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
#或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

![image.gif](assets/9345a00f01f243e283a9f87b70a3bc5b.gif)

### **CentOS 7**

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
#或
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

![image.gif](assets/47ade1a1b8de42d09b4145d3dd27f7f6.gif)

#  三、**添加EPEL**

### CentOS 6

```
wget -O /etc/yum.repos.d/epel-6.repo http://mirrors.aliyun.com/repo/epel-6.repo
```

![image.gif](assets/de5bfb8c0cd8459783b528acac000c16.gif)

### CentOS 7

```
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

![image.gif](assets/cbb6b28b462048909da08763455e3570.gif)

# 四、**清理缓存并生成新的缓存**

```
yum clean all
yum makecache
```

![image.gif](assets/29ebba5d3a3d434b801e80b032170e93.gif)