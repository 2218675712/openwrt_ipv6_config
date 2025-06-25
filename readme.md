# OpenWrt IPv6 设置方案

## 前言

> 原 GitHub 链接已失效/无法访问
> 原地址 [github](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenWrt-IPv6-设置方案)

本方案旨在帮助您优雅地使用 OpenWrt 的 IPv6 功能，主要针对 **OpenWrt 主路由拨号环境**进行设置。

### 关于旁路由和二级路由

*   **不提供旁路由的任何设置方案。**
*   **强烈建议使用主路由环境，抛弃旁路由。** 旁路由出现问题请自行解决，本方案不提供支持。
    具体原因请参考：[关于旁路由的一些吐槽](https://web.archive.org/web/20250410000101/https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/关于旁路由的一些吐槽)
*   关于二级路由，网上教程很多，本人没有相关使用环境，因此不提供任何设置方案，请自行百度。

### 关于 IPv6 设置前提

目前三大运营商的家庭宽带基本都提供 IPv6 地址。**首先请确认您的宽带能够获得 IPv6-PD 地址**，本方案才适用。如果无法获取，请暂时寻求其他解决方案。后续可能会更新无 PD 地址的设置方案。

---

## IPv6 设置方案

由于 OpenWrt 和 Lean's Lede 的设置界面有所不同，本文提供了两种固件的相应截图。请根据您的设置界面选择对应的方案。

### OpenWrt 设置

本人使用的是 ImmortalWrt 的 SNAPSHOT 版本，OpenWrt 设置同理。

#### 1. Dnsmasq 设置

*   **关闭 Dnsmasq 的“过滤 IPv6 AAAA 记录”功能。**
    如果不关闭此项，Dnsmasq 解析的地址中将不会返回 IPv6 地址，从而无法访问 IPv6 网站。

    ![Dnsmasq IPv6 AAAA 记录过滤设置](/pics/v6-7.png)

#### 2. WAN 口 IPv6 地址设置

**WAN 口方法分为“自动创建”和“手动创建”，请二选一。**

##### 自动创建

适用于没有特殊要求的用户。

1.  **不需要新建 WAN6 接口**，请删除已有的 WAN6 接口。
2.  在 WAN 口的**高级设置**中，开启 IPv6 选项，并勾选“使用运营商通告的 DNS”。
3.  **禁用**“IPv6 分配长度”。
4.  **启用**“委托 IPv6 前缀”（启用后 LAN 口将没有 IPv6 地址）。
5.  **不要填写**“IPv6 首选项”，否则可能无法获取到地址。
6.  按照以下截图进行设置：

    ![WAN 口高级设置 1](/pics/v6-1.png)
    ![WAN 口高级设置 2](/pics/v6-2.png)

*   在 **WAN 接口的 DHCP 设置**中，确保 **DHCP > IPv6 设置**已经关闭。

    ![WAN 接口 DHCP IPv6 设置](/pics/v6-3.png)

*   保存并应用设置后，您的接口界面中应该会出现一个虚拟的 `wan_6` 接口。**注意此接口无法编辑设置。**
*   **确认红框中的 IPv6-PD 地址**，只有获取到此地址才能进行下一步操作。

    ![WAN 口 IPv6-PD 地址确认](/pics/wan.png)

##### 手动创建

禁用以下选项：

*   `WAN > 高级设置 > 获取 IPv6 地址`
*   `WAN > 高级设置 > IPv6 源路由`
*   `WAN > 高级设置 > 委托 IPv6 前缀`
*   `WAN > 高级设置 > IPv6 分配长度`

在 `OpenWrt > 网络 > 接口` 界面，新建一个接口并命名为 `WAN6`。
按照图中内容对 `WAN6` 接口进行配置：

![WAN6 接口配置 1](/pics/wan6-1.png)
![WAN6 接口配置 2](/pics/wan6-2.png)

保存并应用配置后，检查 WAN6 接口是否取得了 **IPv6-PD 地址**。

![WAN2 接口 IPv6-PD 地址确认](/pics/wan2.png)

**如果未获取到 PD 地址：**
*   **情况一：** 您的 PD 地址被上一级路由占用，不适用本方案。
*   **情况二：** 您的运营商未提供 IPv6-PD 地址。请直接联系光猫上的安装维护人员（避免拨打 10000 号等客服电话，以免浪费时间），确认宽带是否能提供 IPv6-PD 地址，并检查光猫桥接设置中是否选择了 IPv4&IPv6（部分安装人员可能只设置 IPv4）。

#### 3. LAN 口下发 IPv6 地址设置

完成 WAN 口设置后，接着进行 LAN 口设置。

*   **“委托 IPv6 前缀”**允许下级设备再划分子网，请按需勾选。

    ![LAN 口委托 IPv6 前缀设置](/pics/v6-4.png)

接下来设置 LAN 口的 IPv6 网络地址分配服务，以便局域网设备获取 IPv6 地址。

**IPv6 地址由前缀和后缀组成：**
*   **前缀**由运营商下发。
*   **后缀**有两种获取 IP 的方式：
    1.  **SLAAC（无状态）：** 后缀由局域网设备自身生成。所有类型的设备都支持该功能。
    2.  **DHCPv6（有状态）：** 后缀由 OpenWrt 统一管理。安卓设备及部分其他设备不支持该功能。

因此，此处建议**关闭 DHCPv6，启用 SLAAC**，并使用 `eui64` 参数来启用 EUI-64 网络地址分配方式，从而形成固定 IP。
参考设置：[immortalwrt/user-FAQ/如何优雅的使用IPV6？](https://web.archive.org/web/20250410000101/https://github.com/immortalwrt/user-FAQ/blob/main/immortalwrt%20常见问题指北.md#5-如何优雅的使用ipv60)

在 `LAN > 高级设置 > IPv6 后缀` 中填入 `eui64`。

![LAN 口 IPv6 后缀设置](/pics/eui64.png)

EUI-64 允许设备的后缀地址由 MAC 地址生成，从而生成唯一的后缀。
关于 EUI-64 网络地址分配方式的技术解释，请参阅 ImmortalWrt 仓库文档：[`关于eui64的一些说明.md`](/关于eui64的一些说明.md)

同时，我们需要**禁止 OpenWrt 通告 IPv6 地址的 DNS**，因为设备只需 OpenWrt 的 IPv4 DNS 地址即可实现 IPv6 解析。

![OpenWrt IPv4 地址作为 DNS](/pics/nslookup.png)
*192.168.1.1 为 OpenWrt 的 IPv4 地址*

强制下游设备使用 OpenWrt 的 IPv4 地址（例如 192.168.1.1）来解析包括 IPv6 域名在内的所有域名，可以有效避免许多问题。

![LAN 口 DHCP 服务器 IPv6 设置 1](/pics/v6-5.png)
![LAN 口 DHCP 服务器 IPv6 设置 2](/pics/v6-6.png)

如此设置后，局域网内支持 IPv6 的设备都将获得一个固定且唯一的 IPv6 地址，并且 IPv6 DNS 为空。

#### 4. 测试

使用 `Edge`/`Chrome` 或其他 `Chromium 内核`的浏览器，在**关闭浏览器安全 DNS 功能**的情况下，访问 IPv6 测试网站来验证设置是否正确：

**请勿使用 Firefox 访问测试网站！** Firefox 默认 IPv4 优先，可能无法通过测试。

[https://testipv6.cn/](https://testipv6.cn/)

正常情况下，您将获得“全部通过”的结果。

![IPv6 测试结果](/pics/ipv6test.png)

至此，OpenWrt 的 IPv6 功能设置完毕。

### Lean's LEDE 设置方案

目前本人所有设备均不再使用 Lean's LEDE 源码固件，仅因个人更偏好官方版和 ImmortalWrt。因此，未来将不再提供 LEDE 的设置方案。

请自行摸索相关设置，设置内容与 OpenWrt 方案相似。
以下图片仅供参考：

![LEDE IPv6 设置 1](https://web.archive.org/web/20250410000101im_/https://github.com/Aethersailor/Custom_OpenClash_Rules/raw/main/doc/ipv6/lede/pics/lede6-1.png)
![LEDE DHCPv6 设置](https://web.archive.org/web/20250410000101im_/https://github.com/Aethersailor/Custom_OpenClash_Rules/raw/main/doc/ipv6/lede/pics/dhcpv6.png)

## IPv6 端口转发的正确设置

首先明确一点，您的下游设备获取的都是**公网 IPv6 地址**。因此，此处实际上**不需要“端口转发”功能**，只需设置对应的防火墙规则，即可实现与 IPv4 端口转发相同的使用效果。

尽管局域网内设备取得了 IPv6 公网地址，您可能会发现从公网虽然可以 ping 通这些地址，但并不能直接访问这些地址的端口服务。

**原因在于：** OpenWrt 的防火墙规则默认放行转发给下游设备的 IPv6 ICMP 数据包，但并未对其他数据进行放行。这是一种安全设定，旨在避免下游设备在取得 IPv6 公网地址后不安全地暴露于公网环境中。

如果您需要从公网访问 OpenWrt 下游设备的 IPv6 地址的特定端口（例如群晖的 5000 端口），则需要建立相应的防火墙通信规则，对特定地址和端口进行放行。

具体设置参考：[immortalwrt/user-FAQ/IPV6如何正确配置端口转发？](https://web.archive.org/web/20250410000101/https://github.com/immortalwrt/user-FAQ/blob/main/immortalwrt%20常见问题指北.md#6-ipv6如何正确配置端口转发)

**注意填写地址部分：** 只需填写设备的 IPv6 地址的**后 16 位**（即 MAC 生成的部分）。这样防火墙规则会按照地址后缀去匹配设备，无需担心运营商下发的地址前缀变动，因为地址后缀是根据 MAC 生成的固定后缀。

## 关于本地 IPv6 DNS

需要说明一点，能否解析 IPv6 的 AAAA 地址并不取决于服务器自身是否有 IPv6 地址。

例如，当您的 OpenWrt 的上游 DNS 服务器提供 IPv6 域名解析时，您的 OpenWrt 的 Dnsmasq 就可以提供 IPv6 域名的解析服务。这与您通过 IPv4 还是 IPv6 去请求 Dnsmasq 解析无关。即使您使用路由器的 IPv4 地址去请求域名解析，一样可以取得 IPv6 结果。

![OpenWrt IPv4 地址作为 DNS](/pics/nslookup.png)
*192.168.1.1 为 OpenWrt 的 IPv4 地址*

也就是说，当 OpenWrt 作为您的 DNS 服务器时，只要您的局域网设备获得了 IPv6 地址并且可以正常访问 IPv6 网络，即使局域网设备只取得了 IPv4 的 DNS 地址（例如 192.168.1.1），仍然可以通过 IPv4 的 DNS（192.168.1.1）去解析获得 AAAA 记录，从而正常访问 IPv6 网站。

在实际使用中，Windows 11 偶尔会出现可以 ping 通 OpenWrt 的 IPv4 局域网地址（192.168.1.1）但无法 ping 通 OpenWrt 的 IPv6 ULA 地址（fda6:xxxx:xxxx::1）的情况。此时，只需取消“本地 IPv6 DNS 服务器”勾选，并在“RA 标记”中取消“其他配置”的勾选，即让 OpenWrt 不再向 Windows 11 通告 IPv6 DNS 地址，从而强制系统以 IPv4 DNS（192.168.1.1）来解析 IPv6 网站的域名，以正常进行访问。

可能有些拗口，那么请**无脑按照以下截图内容设置即可。**

**一句话解释：**
您可以直接使用 IPv4 DNS（例如路由本身的 192.168.1.1）来取得 IPv6 解析结果，**不需要去折腾 IPv6 相关的 DNS 设置**。让 OpenWrt 不要向下游设备通告 IPv6 DNS 地址即可，这可以避免很多问题。