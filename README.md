# SSPanel Uim 端口偏移及教程

引用《[聊聊sspanel为什么要端口偏移，NAT服务器它的意义是什么](https://blog.66cto.cn/index.php/archives/37/)》的一段话：

> sspanel为什么要端口偏移：
> 总结以上NAT服务器的见解，我们知道了购买NAT服务器可以用最少的钱享受比较好的线路，但是问题来了并不是所有商家都可以给你分配你想要的端口，比如我的sspanel单端口使用的是其他端口，那么就需要sspanel 的端口偏移使用户订阅到的服务器端口是我们购买的NAT服务器所开通的相应端口。

## SSPanel Uim 前端端口偏移教程

本部分所用代码根据引用文章修改自 [SSPanel-Uim](https://github.com/Anankke/SSPanel-Uim/tree/master) 最新 master 分支代码

> 本部分引用自 [科科博客网](https://kkw.me)
> 引用文章链接：<https://kkw.me/563.html> (转载时请注明本文出处及文章链接)

前端教程非常简单，本身sspanel就有一个分支提供了端口偏移，只需要我们把文件替换一下就可以进行添加节点，使用后端进行对接了

我在这里给两套方案，两套方案都是替换文件夹：

### 方案一：

原版代码，是需要在节点后面加#偏移值

```php
/*$node_name = $node->name;*/
/***节点名字后加#偏移值***/
$temp = explode("#", $node->name);
$offset = 0;
if ($temp[1] != null) {
	$node_name = $temp[0];
	if (is_numeric($temp[1])) {
		$offset = $temp[1];
	}
} else {
	$node_name = $node->name;
}
/************/

/*$return_array['port'] = $user->port;*/
/***端口偏移***/
$return_array['port'] = $user->port + $offset;
/************/
```

用 [URL.php](https://github.com/whunt1/SSPanel_Uim_port_offset/blob/master/URL.php) 替换掉网站根目录的`/app/Utils/URL.php`文件

替换完成后，然后就是添加节点的事情了！

添加你需要偏移的节点名后面加上#需要偏移多少位数，需要偏移大就正数即可，如果需要偏移小就用负数即可！

例如：

> 本身单端口为10000
> 节点名：日本免费节点#80
> 偏移端口结果：10080

到了这里前端已经完成了！就只需要后端进行对接就可以了！

### 方案二：

虽然上面方案一提供的方案我们能正常进行端口偏移，但是我们在用户查看节点列表的时候就会显示：

> 日本免费节点#80

这样的节点对于用户来说非常丑，我们想他只显示

> 日本免费节点

不需要后面有个#+偏移值，怎么办?

那么我们就需要下面的代码了，下面是我自己魔改过的代码！

```php
$node_name = $node->name;
/***节点描述填写#偏移值***/
$temp = explode("#", $node->info);
$offset = 0;
if ($temp[1] != null && is_numeric($temp[1])) {
	$offset = $temp[1];
}
/************/

/*$return_array['port'] = $user->port;*/
/***端口偏移***/
$return_array['port'] = $user->port + $offset;
/************/
```

魔改后的代码，只需要在描述后面加#偏移值即可，对于用户来说体验也是很好的

用 [mod/URL.php](https://github.com/whunt1/SSPanel_Uim_port_offset/blob/master/mod/URL.php) 替换掉网站根目录的`/app/Utils/URL.php`文件

前端以上两套方案，看你怎么使用都行！

## SSPanel Uim 端口偏移后端教程 (脚本或手动部署)

后端一键安装脚本可以参考《[SSPanel-Uim wiki](https://github.com/Anankke/SSPanel-Uim/wiki/%E5%90%8E%E7%AB%AF%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC)》

后端手动部署教程可以参考《[安装ss-panel-v3-mod（魔改版）后端](https://zorz.cc/post/install-sspanel-v3-mod-back-end.html)》

部署完毕后，我们使用 linux 的 iptables 来实现自身端口转发
首先启用网卡转发功能

`echo 1 > /proc/sys/net/ipv4/ip_forward`

然后使用 iptables 转发本机自身端口

```bash
iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 偏移后端口 -j ACCEPT
iptables -I INPUT -m state --state NEW -m udp -p udp --dport 偏移后端口 -j ACCEPT
iptables -t nat -A PREROUTING -p tcp --dport 偏移后端口 -j REDIRECT --to-ports 面板对接所用单端口
iptables -t nat -A PREROUTING -p udp --dport 偏移后端口 -j REDIRECT --to-ports 面板对接所用单端口
service iptables save
```

## SSPanel Uim 端口偏移后端教程 (docker)

> 本部分转载自《[ss-panel-v3-mod_Uim 端口偏移后端教程（docker）](https://blog.66cto.cn/index.php/archives/22/)》
> 该文基于《署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)》许可协议授权 
> 文章链接：<https://blog.66cto.cn/index.php/archives/22/> (转载时请注明本文出处及文章链接)

本文档僅供個人研究使用，請自覺遵守中國大陸相關法律法規。請勿用於非法用途，如由此引起的相關法律法規責任，與我們無關！

根据上一篇 ss-panel-v3-mod_Uim 前端端口偏移教程

> https://blog.66cto.cn/index.php/archives/9/

后端端口偏移非常简单，最简单的方式就是使用docker的容器映射地址来做调整达到目的。
但是如果你是使用 十一 大佬一键脚本部署的，我建议你可以使用linux的防火墙自身端口转发来实现。

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/marisn2017/ss-panel-v3-mod_Uim/master/node.sh && chmod +x node.sh && ./node.sh
```

配置环境：

1. 安装了docker-ce，或者其他版本。
2. 关闭了selinux ，并且关闭自身防火墙。
   

开始教程：

1. 启动docker
   
```bash
service docker start
```

2. 修改下面的参数直接执行，具体参数可以理解为api 对接方式。
   
```bash
docker run -d --name=name -e NODE_ID=0 -e API_INTERFACE=modwebapi -e WEBAPI_URL=https://666.cn  -e SPEEDTEST=0 -e WEBAPI_TOKEN=token --log-opt max-size=50m --log-opt max-file=3 -p nat服务器端口:面板对接所用单端口/tcp -p nat服务器端口:面板对接所用单端口/udp  --restart=always stone0906/ssrmuv2
```

docker基本命令：

```bash
docker ps -a #查看容器
docker rm #容器id 删除容器
docker logs #容器id 查看日志
service docker restart #重启容器
```