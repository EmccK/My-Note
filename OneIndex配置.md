## 在文件夹底部添加说明

在onedrive的文件夹中添加`README.md`文件，使用markdown语法。

## 在文件夹头部添加说明  

在onedrive的文件夹中添加`HEAD.md`文件，使用markdown语法。  

## 加密文件夹

在onedrive的文件夹中添加`.password`文件，填入密码，密码不能为空。 

## 直接输出网页

在onedrive的文件夹中添加`index.html`文件，程序会直接输出网页而不列目录，配合文件展示设置-直接输出效果更佳。

## 更改网站图标

找到“/view/主题目录/”下的layout.php文件

把favicon.ico上传到网站根目录后，在head里面【<head>.....</head>】里面加一行

```
<link rel="shortcut icon" href="/favicon.ico" type="image/x-icon" />
```

## **标题名：**修改如下语句

```xml
<title><?php e(config('site_name').' - '.$title);?></title>
```

## 如果只想显示为设置里所设置的名字，则改为

```
<title><?php e(config('site_name'));?></title>
```

