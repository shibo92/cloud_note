

<!-- vim-markdown-toc GFM -->

* [ubuntu 修改wine分辨率](#ubuntu-修改wine分辨率)
* [scp上传下载](#scp上传下载)

<!-- vim-markdown-toc -->

### ubuntu 修改wine分辨率
 + WINEPREFIX=~/.deepinwine/Deepin-WeChat deepin-wine winecfg

---

### scp上传下载
 + scp [参数] [原路径] [目标路径]
  - 上传
  命令格式：scp local_file remote_username@remote_ip:remote_folder 
  例如： scp  -P 40022 /Users/shibo/local/huawei_unsigned_signed.apk root@ip:/home/qiban/bzit_app/
  - 下载
  例如： scp root@ip:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/
  参数：-v 查看进度 
		-P 端口
		-r 传输目录
