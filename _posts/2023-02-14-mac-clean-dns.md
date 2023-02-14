---

layout: post
title: Mac清空DNS缓存命令
date: 2023-02-12 09:00:00
categories: 网络
---

### Mac清空DNS缓存命令

<table>
<thead>
<tr>
<th>MacOS Version</th>
<th>Command</th>
</tr>
</thead>
<tbody>
<tr>
<td>macOS 12 (Monterey)</td>
<td><code>sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>macOS 11 (Big Sur)</td>
<td><code>sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>macOS 10.15 (Catalina)</td>
<td><code>sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>macOS 10.14 (Mojave)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>macOS 10.13 (High Sierra)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>macOS 10.12 (Sierra)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>OS X 10.11 (El Capitan)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>OS X 10.10 (Yosemite)</td>
<td><code>sudo discoveryutil udnsflushcaches</code></td>
</tr>
<tr>
<td>OS X 10.9 (Mavericks)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>OS X 10.8 (Mountain Lion)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>Mac OS X 10.7 (Lion)</td>
<td><code>sudo killall -HUP mDNSResponder</code></td>
</tr>
<tr>
<td>Mac OS X 10.6 (Snow Leopard)</td>
<td><code>sudo dscacheutil -flushcache</code></td>
</tr>
<tr>
<td>Mac OS X 10.5 (Leopard)</td>
<td><code>sudo lookupd -flushcache</code></td>
</tr>
<tr>
<td>Mac OS X 10.4 (Tiger)</td>
<td><code>lookupd -flushcache</code></td>
</tr>
</tbody>
</table>
