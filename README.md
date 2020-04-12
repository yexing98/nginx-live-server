# nginx-live-server使用说明
#首先打开直播服务器，然后按照如下步骤安装：
#一、安装Git工具到服务器
[root@vultr ~]#yum install -y git

#二、用Git工具从服务器获取nginx-live-server安装的脚本到服务器的root目录中
[root@vultr ~]# git clone https://github.com/yexing98/nginx-live-server.git

#三、切换到nginx-live-server文件夹中
[root@vultr ~]# cd nginx-live-server

#四、赋予脚本nginx-rtmp-hls-live-install.sh可执行程序
[root@vultr nginx-live-server]# chmod +x nginx-rtmp-hls-live-install.sh

#五、用bash命令执行nginx-rtmp-hls-live-install.sh脚本
[root@vultr nginx-live-server]# bash nginx-rtmp-hls-live-install.sh

#六、到此服务器就安装完成，用#netstat -ntlp命名查看一下服务器打开的端口
[root@vultr nginx-live-server]#netstat -ntlp
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:1935            0.0.0.0:*               LISTEN      26610/nginx: master 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      26610/nginx: master 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1425/sshd     
#服务器启动正常的话，会看到1935端口和80号端口已经在监听中。

#七、下载并安装OBS推流软件，推流开始直播（详细教程，请参考OBS使用教程，下载地址：https://obsproject.com/）
#假设直播服务器IP地址是：43.224.34.195
#OBS推流地址是：rtmp://43.224.34.195:1935/hls/  流名称是：live
#有些推流软件只有一个地址可以填的话，就把流名称合起来：rtmp://43.224.34.195:1935/hls/live
#电脑端可以用VLC拉流收看，地址是：rtmp://43.224.34.195/hls/live
#移动端可以用微信直接收看地址：http://43.224.34.195/hls/live.m3u8
#移动端也可以用MX播放器拉流收看，地址是：http://43.224.34.195/hls/live.m3u8
#移动端可以用MX播放器拉流收看，地址是：rtmp://43.224.34.195/hls/live

#八、功能说明：Nginx+RTMP支持rtmp直播、hls直播和vod点播，支持直播时录制，直播时同时推拉流到第三方平台，从别的平台拉流到本地直播。具体说明如下：
rtmp{
	server{
		listen 1935;
		chunk_size 4096;
	application hls{
	live on;
	#push rtmp://p1.weizan.cn/v/11886982_132294270062269007?t=2e8e2;   //使用时，去掉#，推流到微赞平台
	#push rtmp://167.179.77.30:1935/hls/live;                         //使用时，去掉#，推流到其他直播服务器
	#pull rtmp://r2.weizan.cn/v/11886982_132310524333578171;          //使用时，去掉#，从其他服务器拉流直播到本服务器。（有客户观看时才会拉流）
	hls on;
	hls_path /home/html/hls;                      //直播文件放置的路径，直播介绍时会自动销毁。
	hls_fragment 5s;
	record all;
	record_path /home/html/record;               //录制文件的路径
	record_interval 60m;                         //每隔60分钟录制一次
	record_suffix rec.mp4;                      //录制文件的格式是mp4
	record_unique on;                           //直播时按照日期不同自动命名录制文件
	#record_max_size 51200K;                    //去掉#直播录制文件按每51200K分开
	}

	application vod{                           
	play  /home/html/record;        //回放点播的路径：点播只支持RTMP方式，点播地址rtmp://43.224.34.195/vod/XXX.mp4 (XXX是录制的文件名）
        }
}
}

#九、PC端网页观看直播的html代码(安装脚本中已经自动设定好以下index.html代码内容在/home/html/文件夹中）
<html>
<head>
    <title>live</title>
    <meta charset="utf-8">
    <link href="http://vjs.zencdn.net/5.5.3/video-js.css" rel="stylesheet">
    <!-- If you'd like to support IE8 -->
    <script src="http://vjs.zencdn.net/ie8/1.1.1/videojs-ie8.min.js"></script>
    <script src="http://vjs.zencdn.net/5.5.3/video.js"></script>
</head>
<body>
<video id="my-video" class="video-js" controls preload="auto" width="640" height="480"
       poster="http://ppt.downhot.com/d/file/p/2014/08/12/9d92575b4962a981bd9af247ef142449.jpg" data-setup="{}">
    <source src="rtmp://43.224.34.195/hls/live" type="rtmp/flv">      #这里是直播服务器地址#
    </p>
</video>

</body>
</html>
