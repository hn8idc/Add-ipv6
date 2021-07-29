# Add-ipv6
为服务器增加ipv6
## 安装WireGuard
先安装linux-headers
```
apt update    #如有需要可更新后面两项，updat之后系统重启一下 apt upgrade && apt dist-upgrade
apt install linux-headers-$(uname -r) -y
```
 
安装WireGuard
```
echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 150\n' > /etc/apt/preferences.d/limit-unstable
apt update
apt install wireguard-dkms wireguard-tools resolvconf -y
```
## wgcf生成WireGuard配置

下载wgcf
```
wget https://github.com/ViRb3/wgcf/releases/download/v2.1.4/wgcf_2.1.4_linux_amd64 -O wgcf
chmod +x wgcf
```
生成配置
```
./wgcf register
./wgcf generate
```
如果没有出错，会在当前目录生成`wgcf-account.toml、wgcf-profile.conf`两个文件。

如果出现`429`错误，可以多试几次。

编辑配置文件
```
vim wgcf-profile.conf
```
注释掉监听ipv4/ipv6(`AllowedIPs = 0.0.0.0/0`)，保留连接SSH的IP。否则会失联。

连接WARP
复制一份`wgcf-profile.conf`到`/etc/wireguard`
```
rm -rf /etc/wireguard
mkdir /etc/wireguard
cp wgcf-profile.conf /etc/wireguard
```
连接(up)，断开(down)
```
export WG_I_PREFER_BUGGY_USERSPACE_TO_POLISHED_KMOD=1
wg-quick up wgcf-profile
wg-quick down wgcf-profile
```
测试IPV6是否开启成功,若有返回即成功
```
curl ipv6.ip.sb
```

设置开机自启动
```
systemctl enable wg-quick@wgcf-profile
```
重启VPS,完事收工


## 让奈飞走IPV6线路
修改V2的配置文件
```
"outbounds": [
  {
    "tag":"IPv4_out",
    "protocol": "freedom"
  },
  {
    "tag":"IPv6_out",
    "protocol": "freedom",
    "settings": {
      "domainStrategy": "UseIPv6"
    }
  }
],
"routing": {
  "rules": [
    {
      "type": "field",
      "outboundTag": "IPv6_out",
      "domain": ["geosite:netflix","nflxvideo.net","nflxext.com","nflxso.net"]
    },
    {
      "type": "field",
      "outboundTag": "IPv4_out",
      "network": "udp,tcp"
    }
  ]
}
```

## 第二种傻瓜式方法
#### 第一步 
升级系统内核并重启,可尝试直接第二步是否能直接运行
``` bash
apt update && apt install curl lsb-release -y && echo "deb http://deb.debian.org/debian $(lsb_release -sc)-backports main" | tee /etc/apt/sources.list.d/backports.list && apt update && apt -t $(lsb_release -sc)-backports install linux-image-$(dpkg --print-architecture) linux-headers-$(dpkg --print-architecture) --install-recommends -y && reboot
```
#### 第二步
一键WARP解锁ipv6，请自行备份wgcf-profile.conf以便重装系统之后还原使用
``` bash
apt update && apt install curl lsb-release -y && echo "deb http://deb.debian.org/debian $(lsb_release -sc)-backports main" | tee /etc/apt/sources.list.d/backports.list && apt update && apt install net-tools iproute2 openresolv dnsutils -y && apt install wireguard-tools --no-install-recommends && curl -fsSL git.io/wgcf.sh | bash && wgcf register && wgcf generate && sed -i '/0.0.0.0/d' ./wgcf-profile.conf  && sed -i 's/engage.cloudflareclient.com/162.159.192.1/g' ./wgcf-profile.conf && sed -i 's/1.1.1.1/9.9.9.10,8.8.8.8,1.1.1.1,8.8.4.4/g' ./wgcf-profile.conf && cp wgcf-profile.conf /etc/wireguard/wgcf.conf && systemctl start wg-quick@wgcf && systemctl enable wg-quick@wgcf && grep -qE '^[ ]*label[ ]*2002::/16[ ]*2' /etc/gai.conf || echo 'label 2002::/16   2' | tee -a /etc/gai.conf
```


#### 查看WARP当前统计状态：```wg```

#### 查看当前IPV4 IP：```curl -4 ip.p3terx.com```

#### 查看当前IPV6 IP：```curl -6 ip.p3terx.com```
