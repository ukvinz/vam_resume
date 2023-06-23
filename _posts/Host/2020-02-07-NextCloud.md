---
title: 部署个人网盘
author: Lin Han
date: "2021-02-07 19:02 +8"
published: false
categories:
  - SelfHost
  - Drive
tags:
  - SelfHost
  - Drive
---

dietpi-software 安装 nextcloud 之后的步骤

## 扫描盘上文件

将文件复制到用户 files 文件夹下，让 nextcloud 从盘上扫描文件

```shell

cd /var/www/

sudo -u www-data php console.php files:scan --all
```

nextcloud 默认在客户端和服务器上都跳过 symlink，开启跟随 simlink：在 /var/www/nextcloud/ 中添加

```php
'localstorage.allowsymlinks' => true, # 允许在服务器端用软链接
```

## 日历

校验 ics： https://icalendar.org/validator.html （要求 crlf 结尾）

从谷歌日历同步：https://help.nextcloud.com/t/sync-share-google-calendar-nextcloud-calendar/114131

![image](https://user-images.githubusercontent.com/29757093/222927755-7e9322b3-22d4-40b3-bc40-36db3c440687.png)

修改日历同步周期

sudo -u www-data php occ config:app:set dav calendarSubscriptionRefreshRate --value "P1H"

同步到安卓：https://docs.nextcloud.com/server/latest/user_manual/en/groupware/sync_android.html

![image](https://user-images.githubusercontent.com/29757093/222928112-b525164c-e48d-41d0-be07-e7f49638de0c.png)

DAVx⁵ ICSx⁵ Etar

https://github.com/Etar-Group/Etar-Calendar

可以订阅只读的 iCal 链接，可以管理 CalDAV 日历

## 联系人

## 短信

## office

docker run -t -d -p 127.0.0.1:9980:9980 -e 'domain=cloud\\.linhan\\.ml' -e 'dictionaries=en cn' --restart always --cap-add MKNOD collabora/code

curl -k https://localhost:9980

https://fqdn/index.php/settings/admin/richdocuments

## 多媒体

[预览文档](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/config_sample_php_parameters.html#previews)

默认开启预览的文件格式应该是包括这些

![image](https://user-images.githubusercontent.com/29757093/222930255-df742d02-9294-4a1a-af39-2f4de7bf81e8.png)

其他格式有

![image](https://user-images.githubusercontent.com/29757093/222930290-643daf5c-aa90-48b8-85a2-5ec7c5c2a201.png)

设置给其他格式开启预览，修改 /var/www/nextcloud/config/configs.php

```php
  'enable_previews' => true,
  'enabledPreviewProviders' =>
    [
      'OC\Preview\PNG',
      'OC\Preview\JPEG',
      'OC\Preview\GIF',
      'OC\Preview\BMP',
      'OC\Preview\XBitmap',
      'OC\Preview\MP3',
      'OC\Preview\TXT',
      'OC\Preview\MarkDown',
      'OC\Preview\OpenDocument',
      'OC\Preview\Krita',
      'OC\Preview\Movie',
      'OC\Preview\PDF',
      // 添加更多格式
    ],
```

预览图大小，质量

```php
  'preview_max_x' => 512, # 最大预览图大小
  'preview_max_y' => 768,
  'preview_max_scale_factor' => 10, # 最大放大倍数
```

```shell
sudo -u www-data php occ config:app:set preview jpeg_quality --value="80" # 默认 90
```

给视频生成预览需要装 ffmpeg

```shell
sudo apt install ffmpeg
```

<!-- TODO: 中文txt乱码 -->

nextcloud 默认是打开目录的同时生成图片预览，预先生成并 cache 预览可以用第一方插件 [Preview Generator](https://github.com/nextcloud/previewgenerator)

```shell

rm -r /mnt/dietpi_userdata/nextcloud_data/appdata_*/preview/

sudo -u www-data php occ files:scan-app-data

sudo -u www-data php occ config:app:set --value="512" previewgenerator squareSizes
sudo -u www-data php occ config:app:set --value="512" previewgenerator widthSizes
sudo -u www-data php occ config:app:set --value="768" previewgenerator heightSizes

sudo -u www-data php occ preview:generate-all -vvv

sudo -u www-data php occ preview:generate-all -vvv --path=PATH

```

定时生成预览：

```shell
crontab -e

sudo -u www-data php /var/www/nextcloud/occ preview:pre-generate

```

## 安全
