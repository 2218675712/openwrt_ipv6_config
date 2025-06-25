# IPv6 中继方案：告别 NAT6，实现原生公网 IPv6 地址

本文旨在阐述 IPv6 中继（透传）方案的优势，并提供 OpenWrt 路由器上的配置指南，帮助用户实现 LAN 端设备直接获取原生公网 IPv6 地址。

## 为什么选择 IPv6 中继？

许多用户仍在尝试配置 NAT6，但实际上 IPv6 可以直接通过中继方式实现透传，让 LAN 端设备直接获取原生公网 IPv6 地址，这不仅设置简单，而且性能更优。

**强烈不推荐 NAT6 方案**：
*   NAT6 方案复杂且性能低下。
*   IPv6 地址较长，NAT 后会极度消耗路由器资源，导致速度下降。
*   除非上游只能分配 `/128` 地址（这种情况极为罕见，通常至少会分配 `/64`），否则不应使用 NAT6。

## 中继方案与 NAT6 方案对比

| 特性       | IPv6 中继（透传，厂商主流方案）                               | NAT6（恩山流行方案）                                         |
| :--------- | :------------------------------------------------------------- | :----------------------------------------------------------- |
| **适用场景** | 路由器无法获取 IPv6-PD，只能获取 `/64` 地址的情况（如校园网）。 | 理论上适用于上游只能分配 `/128` 地址的情况。                 |
| **客户端 IPv4** | 私有地址，由 OpenWrt 分配，流量需 NAT 改包转发，效率低，速度慢。 | 私有地址，由 OpenWrt 分配，流量需 NAT 改包转发，效率低，速度慢。 |
| **客户端 IPv6** | 公网地址，通过 WAN6 中继到 LAN，流量直接路由转发，效率高，速度快。 | 私有地址，由 OpenWrt 分配，流量需 NAT6 改包转发，效率更低，速度更慢。 |
| **外部访问** | 无需端口映射，只需打开防火墙规则。                             | 需要端口映射。                                               |
| **资源消耗** | 不存在地址转换，不占用路由器资源，速度极快。                   | IPv6 地址转换极度消耗路由器资源。                            |

## 常见路由器支持

许多家用路由器原生支持 IPv6 中继功能，例如网件（Netgear）和华硕（ASUS）路由器，它们通常称之为 `IPv6 Passthru`（IPv6 透传），功能与中继方案相同。

**重要提示**：原生公网 IPv6 地址可以直接从公网访问。在 OpenWrt 上，WAN 区域的防火墙规则仍需根据需求进行配置以允许外部访问。

## OpenWrt IPv6 中继配置指南 (基于 OpenWrt 21.02.2 英文原版)

本指南适用于 WAN 端只能分配 `/64` 地址，无法向下分配 IPv6-PD 的情况。

**前提**：确保已新建 WAN6 接口，协议为 DHCPv6 客户端，防火墙区域设置为 `wan`，并已成功获取 IPv6 地址。

### WAN6 接口配置

1.  **DHCP Server 设置**：
    *   进入 WAN6 接口的 DHCP Server 设置。
    *   勾选 `Ignore interface`（忽略此接口）。

2.  **IPv6 Settings**：
    *   勾选 `Designated master`（指定为主）。

3.  **RA-Service, DHCPv6-Service, NDP-Proxy**：
    *   将这三个下拉框全部设置为 `relay mode`（中继模式）。

4.  **Learn routes**：
    *   勾选 `Learn routes`（学习路由）。

5.  点击 `Save`（保存）。

### LAN 接口配置

1.  **DHCP Server 设置**：
    *   进入 LAN 接口的 DHCP Server 设置。
    *   在 `IPv6 Settings` 中：
        *   `RA-Service` 设置为 `relay mode`。
        *   `DHCPv6-Service` 设置为 `hybrid`。
        *   `NDP-Proxy` 设置为 `relay mode`。

2.  **Local IPv6 DNS server**：
    *   勾选 `Local IPv6 DNS server`（本地 IPv6 DNS 服务器）。

3.  **Learn routes**：
    *   勾选 `Learn routes`。

4.  点击 `Save`。

完成上述配置后，LAN 端设备即可通过 WAN 端获取原生 IPv6 地址。您还可以根据需要设置 IPv6 防火墙规则。

## 原理简述

1.  **WAN6 端 `Designated master`**：告知系统此接口是上游网络。
2.  **LAN 端通过 `NDP-proxy`**：从上游获取原生 IPv6 地址。
3.  **LAN 端的 `DHCPv6-Service`**：用于告知客户端 IPv6 DNS 解析服务器地址。
4.  **`RA-Service`**：告知 LAN 端获取 IPv6 地址的主机，其上级路由服务器是哪个。

**注意**：通常情况下，联通、移动、电信等运营商都会提供 IPv6-PD（前缀委派），此时无需使用中继方案，只需在 LAN 端开启 DHCPv6 服务器即可。中继方案主要用于 WAN 口只能获取 `/64` 地址且无法向下分配的情况。

## 已知问题与解决方案

**OpenWrt 21.02 版本已知问题**：
*   当启用软件流卸载（Software Flow Offloading）时，可能会导致 IPv6 数据包丢失（断流）。
*   **解决方案**：关闭软件流卸载。此功能默认是关闭的。

    *   **原文参考**：
        ```
        Known issues
        Some IPv6 packets are dropped when software flow offloading is used: FS#3373
        As a workaround, do not activate software flow offloading, it is deactivate by default.
        ```

## 中文改版 OpenWrt 的特殊情况

部分中文改版 OpenWrt 可能没有 `Designated master` 选项。在这种情况下，您需要通过 SSH 修改配置文件。请参考相关中文资料进行操作。
