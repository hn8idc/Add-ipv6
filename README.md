# Add-ipv6
为服务器增加ipv6
## 安装WireGuard
先安装linux-headers
```
apt update
apt install linux-headers-$(uname -r) -y
```
 
安装WireGuard
```
echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 150\n' > /etc/apt/preferences.d/limit-unstable
apt update
apt install wireguard-dkms wireguard-tools resolvconf -y
```
wgcf生成WireGuard配置

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
设置开机自启动
```
systemctl enable wg-quick@wgcf-profile
```
完事收工


