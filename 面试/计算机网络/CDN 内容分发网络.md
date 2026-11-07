
# CDN 的作用是什么？CDN 是怎么实现的？

CDN，即 Content Delivery Network，内容分发网络，是一种通过在不同地理位置部署缓存服务器，使用户就近获取资源，从而提高访问速度、降低源服务器压力的技术。CDN 的核心思想是，将原本集中在源站上的资源缓存到离用户更近的边缘节点上。在图片、视频、静态资源分发等场景中应用广泛。

以不存在 CDN 时为例，所有服务都部署在北京，包括一些静态资源，当上海、广州等的用户试图访问时，必须从北京的服务器获取资源，这种方式有三个不足：
- 所有用户都访问同一个服务器，会对服务器造成较大压力；
- 所有用户都访问北京的服务器，距离较远，网络传输速度会更慢；
- 一旦服务器出现问题，所有用户都会受到影响。

通过使用 CDN，在全国甚至全世界，大量部署边缘节点，同时在边缘节点中缓存源站的一些静态资源，从而保证，世界各地的用户都可以以相对较短的时间访问到资源，并且不会将压力全都给到同一台服务器，且多台 CDN 服务器均衡下，服务的可用性也会更高。然而，引出了两个 CDN 使用上的问题：
1. 用户是如何知道，哪一台 CDN 边缘节点离自己更近的？[[CDN 内容分发网络#当浏览器输入一个 CDN 域名时，DNS 是如何让用户访问到距离自己更近的 CDN 节点的？|DNS节点选择]]
2. 这么多 CDN 节点，如何保证资源缓存的同步性？**缓存更新**

CDN 节点查询时，会使用 DNS 解析，引导用户使用合适的 CDN 节点，并在 CDN 节点上查找对应的静态资源，基本流程如下：
- 用户请求对应 URL，如：`https://cdn.example.com/a001.jpg`；
- 通过对 URL 进行 DNS 解析，并通过 CDN 调度系统将用户引导至合适的 CDN 节点；
- CDN 节点检查本地是否缓存了该资源；
- 如果 CDN 中存在有效缓存 (Cache Hit)，则直接返回；
- 如果不存在 (Cache Miss)，CDN 节点会向源站获取对应资源，这个过程称为回源 (Origin Fetch)；
- CDN 将获取的资源缓存起来，并将对应资源返回给用户。

CDN 的常见应用场景有：
- 图片和视频分发；
- 前端静态资源加速；
- 视频点播和直播 (直播 CDN 与普通文件 CDN 分发略有不同)。

# 当浏览器输入一个 CDN 域名时，DNS 是如何让用户访问到距离自己更近的 CDN 节点的？

CDN 可以让用户访问距离自己更近的服务器，主要依赖于 **DNS 智能调度**和**边缘节点选择策略**。


对于 **DNS 智能调度**策略：

![](data:image/svg+xml;charset=utf-8,%3Csvg%20font-family%3D%22-apple-system-body%2C%20ui-sans-serif%2C%20-apple-system%2C%20system-ui%2C%20%26quot%3BSegoe%20UI%26quot%3B%2C%20Helvetica%2C%20%26quot%3BApple%20Color%20Emoji%26quot%3B%2C%20Arial%2C%20sans-serif%2C%20%26quot%3BSegoe%20UI%20Emoji%26quot%3B%2C%20%26quot%3BSegoe%20UI%20Symbol%26quot%3B%22%20font-weight%3D%22400%22%20data-d-component%3D%22svg%22%20fill%3D%22currentColor%22%20style%3D%22color%3Argb\(237%2C%20237%2C%20237\)%22%20viewBox%3D%220%200%20340%20305%22%20width%3D%22100%25%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%3E%3Crect%20x%3D%2277%22%20y%3D%226%22%20width%3D%22186%22%20height%3D%2239%22%20rx%3D%227%22%20fill%3D%22%23DCEAFE%22%20stroke%3D%22%238CB6E8%22%2F%3E%3Ctext%20x%3D%22170%22%20y%3D%2222%22%20text-anchor%3D%22middle%22%20font-size%3D%2212%22%20fill%3D%22%23275889%22%20font-weight%3D%22600%22%3E%E9%80%92%E5%BD%92%20DNS%20%E8%A7%A3%E6%9E%90%E5%99%A8%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%2235%22%20text-anchor%3D%22middle%22%20font-size%3D%2210%22%20fill%3D%22%23275889%22%3E%E6%9F%A5%E8%AF%A2%20CDN%20%E5%9F%9F%E5%90%8D%3C%2Ftext%3E%3Cpath%20d%3D%22M170%2045%20V65%22%20stroke%3D%22%238292A7%22%20stroke-width%3D%221.6%22%2F%3E%3Cpath%20d%3D%22M166%2059%20L170%2065%20L174%2059%22%20stroke%3D%22%238292A7%22%20stroke-width%3D%221.6%22%20fill%3D%22none%22%2F%3E%3Crect%20x%3D%2269%22%20y%3D%2267%22%20width%3D%22202%22%20height%3D%2239%22%20rx%3D%227%22%20fill%3D%22%23FCE7D8%22%20stroke%3D%22%23E8A876%22%2F%3E%3Ctext%20x%3D%22170%22%20y%3D%2284%22%20text-anchor%3D%22middle%22%20font-size%3D%2212%22%20fill%3D%22%23845022%22%20font-weight%3D%22600%22%3ECDN%20%E6%9D%83%E5%A8%81%20DNS%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%2297%22%20text-anchor%3D%22middle%22%20font-size%3D%2210%22%20fill%3D%22%23845022%22%3E%E6%8E%A5%E6%94%B6%20DNS%20%E6%9F%A5%E8%AF%A2%3C%2Ftext%3E%3Cpath%20d%3D%22M170%20106%20V124%22%20stroke%3D%22%238292A7%22%20stroke-width%3D%221.6%22%2F%3E%3Cpath%20d%3D%22M166%20119%20L170%20125%20L174%20119%22%20stroke%3D%22%238292A7%22%20stroke-width%3D%221.6%22%20fill%3D%22none%22%2F%3E%3Crect%20x%3D%2248%22%20y%3D%22127%22%20width%3D%22244%22%20height%3D%2265%22%20rx%3D%227%22%20fill%3D%22%23DDF5E8%22%20stroke%3D%22%2372B99B%22%2F%3E%3Ctext%20x%3D%22170%22%20y%3D%22145%22%20text-anchor%3D%22middle%22%20font-size%3D%2213%22%20fill%3D%22%2322694B%22%20font-weight%3D%22600%22%3EGSLB%20%E6%99%BA%E8%83%BD%E8%B0%83%E5%BA%A6%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%22161%22%20text-anchor%3D%22middle%22%20font-size%3D%2211%22%20fill%3D%22%2322694B%22%3E%E5%9C%B0%E5%9F%9F%20%2F%20%E8%BF%90%E8%90%A5%E5%95%86%20%2F%20%E5%BB%B6%E8%BF%9F%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%22177%22%20text-anchor%3D%22middle%22%20font-size%3D%2211%22%20fill%3D%22%2322694B%22%3E%E8%8A%82%E7%82%B9%E5%81%A5%E5%BA%B7%20%2F%20%E8%B4%9F%E8%BD%BD%20%2F%20%E8%B0%83%E5%BA%A6%E7%AD%96%E7%95%A5%3C%2Ftext%3E%3Cpath%20d%3D%22M170%20192%20L55%20219%22%20stroke%3D%22%238AA99B%22%20stroke-width%3D%221.4%22%20fill%3D%22none%22%2F%3E%3Crect%20x%3D%224%22%20y%3D%22221%22%20width%3D%22102%22%20height%3D%2242%22%20rx%3D%226%22%20fill%3D%22%23F0F1F3%22%20stroke%3D%22%23BDC5CC%22%2F%3E%3Ctext%20x%3D%2255%22%20y%3D%22238%22%20font-size%3D%2211%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3E%E5%8C%97%E4%BA%AC%E8%8A%82%E7%82%B9%3C%2Ftext%3E%3Ctext%20x%3D%2255%22%20y%3D%22252%22%20font-size%3D%2210%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3EIP%20A%3C%2Ftext%3E%3Cpath%20d%3D%22M170%20192%20L170%20219%22%20stroke%3D%22%238AA99B%22%20stroke-width%3D%221.4%22%20fill%3D%22none%22%2F%3E%3Crect%20x%3D%22119%22%20y%3D%22221%22%20width%3D%22102%22%20height%3D%2242%22%20rx%3D%226%22%20fill%3D%22%23F0F1F3%22%20stroke%3D%22%23BDC5CC%22%2F%3E%3Ctext%20x%3D%22170%22%20y%3D%22238%22%20font-size%3D%2211%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3E%E4%B8%8A%E6%B5%B7%E8%8A%82%E7%82%B9%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%22252%22%20font-size%3D%2210%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3EIP%20B%3C%2Ftext%3E%3Cpath%20d%3D%22M170%20192%20L285%20219%22%20stroke%3D%22%238AA99B%22%20stroke-width%3D%221.4%22%20fill%3D%22none%22%2F%3E%3Crect%20x%3D%22234%22%20y%3D%22221%22%20width%3D%22102%22%20height%3D%2242%22%20rx%3D%226%22%20fill%3D%22%23F0F1F3%22%20stroke%3D%22%23BDC5CC%22%2F%3E%3Ctext%20x%3D%22285%22%20y%3D%22238%22%20font-size%3D%2211%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3E%E5%B9%BF%E5%B7%9E%E8%8A%82%E7%82%B9%3C%2Ftext%3E%3Ctext%20x%3D%22285%22%20y%3D%22252%22%20font-size%3D%2210%22%20text-anchor%3D%22middle%22%20fill%3D%22%23354656%22%3EIP%20C%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%22287%22%20font-size%3D%2211%22%20fill%3D%22currentColor%22%20text-anchor%3D%22middle%22%3E%E4%BE%8B%E5%A6%82%E6%9C%80%E7%BB%88%E9%80%89%E4%B8%AD%E5%B9%BF%E5%B7%9E%E8%8A%82%E7%82%B9%20IP%20C%3C%2Ftext%3E%3Ctext%20x%3D%22170%22%20y%3D%22300%22%20font-size%3D%2210%22%20fill%3D%22%237C8792%22%20text-anchor%3D%22middle%22%3E%E8%B0%83%E5%BA%A6%E6%95%B0%E6%8D%AE%E7%94%B1%E5%90%8E%E5%8F%B0%E7%9B%91%E6%8E%A7%E5%92%8C%E9%85%8D%E7%BD%AE%E7%B3%BB%E7%BB%9F%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0%3C%2Ftext%3E%3C%2Fsvg%3E)

DNS 本身并不知道哪个 CDN 节点距离用户最近，而是 CDN 服务商通过自己的 DNS 调度系统，根据用户的位置、边缘节点负载、网络质量的因素，选择合适的节点并返回对应的 IP 地址。

具体来说，以电商平台的商品图片为例，整个过程如下：
- 针对对应资源，浏览器发起 DNS 查询，如 `cdn.example.com`；浏览器会先查询本地是否有对应缓存，如果没有，就像递归 DNS 解析器发起查询请求；
- 对于通过 **CNAME** 将原 URL 的域名解析交给 CDN 服务商管理的域名体系；
- 递归 DNS 解析器查询 CDN 服务商的权威 DNS，CDN 权威 DNS 通过 GSLB 调度系统根据地域位置、网络质量、节点负载等信息选择节点；
- DNS 解析完成后，浏览器获得节点 IP 地址，完成信息发送请求。

DNS 查询往往经过递归解析器转发，CDN 权威 DNS 默认看到的通常是递归 DNS 解析器的 IP，而不是浏览器的真实 IP。因此，CDN 主要依靠两种方式判断用户的地址：
1. 根据**递归 DNS 的 IP** 判断地域：将递归 DNS 的地址作为用户的地址，直接返回对于地域的节点，但是如果用户使用了非本地的 DNS，就会返回错误的节点位置。
2. **EDNS Client Subnet (ECS)**：ECS 机制允许递归 DNS 在查询中携带用户 IP 的部分网络前缀，CDN 权威 DNS 可以利用这段网络前缀更准确的判断用户的网络区域。

对于**边缘节点选择策略**：
现代 CDN 还有一种技术，即 Anycast，任播。
DNS 智能调度策略主要在 DNS 层完成调度，每个节点使用不同的网络 IP，通过返回给用户不同的 IP 确保用户使用不同的边缘节点。而 Anycast 在网络层完成调度，多个边缘节点使用同一个 IP，具体使用哪个边缘节点由网络路由系统选择对应路径。

用户访问同一个 IP 时，网络通过 BGP 路由机制，将流量引导至路由意义上较合适的节点。并不是地理距离最近的节点，而是根据实际网络路由情况决定的。Cloudflare 的 CDN 就广泛使用了 Anycast 技术。

**CNAME 选择 CDN 服务商管理的流程**
- 对于 `cdn.example.com`，递归 DNS 解析器通过权威 DNS 服务器查询其记录；
- 权威 DNS 服务器中，会配置对应的 CNAME 域名，查询 `cdn.example.com` 的请求会被转换为查询 `example.cdn-provider.com` 的请求；
- 接下来，递归 DNS 解析器会继续查询 `example.cdn-provider.com`，此时 CDN 服务商就有机会介入到 DNS 解析过程，将对应的边缘节点 IP 返回；
- 用户拿到边缘节点 IP。

# CDN 权威 DNS 的 GSLB 调度系统是什么？他是如何完成工作的？



# CDN 节点缓存如何进行更新？

常见的 CDN 缓存更新解决方法有三种，分别是 TTL 过期、主动刷新、URL 版本化。

首先是 TTL 过期。为缓存设置有效期，到期后再根据缓存规则检查或获取新内容。

其次是主动刷新。图片修改后，调用 CDN 服务商的刷新接口，使对应的缓存失效。

最后是 URL 版本化。更新图片时，修改对应 URL，使 CDN 将其视为一个新资源。




# 如果用户访问 CDN 时，DNS 解析已经完成，但对应 CDN 节点突然宕机，会发生什么？


