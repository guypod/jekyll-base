---
ID: 5
post_title: >
  Mitigating ImageMagick vulnerabilities
  in Node.js
author: wpengine
post_date: 2016-05-06 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=5
published: true
---
<p>Multiple severe and trivially exploited vulnerabilities in <a href="http://www.imagemagick.org/script/index.php"><em>ImageMagick</em></a> were disclosed earlier this week, and are known to be exploited in the wild. As there is no official fix yet, we created a package called <a href="https://npmjs.com/package/imagemagick-safe">imagemagick-safe</a> which disables the vulnerable features, protecting against the known exploits.</p>

<h2 id="tl-dr">TL; DR</h2>

<p><a href="http://www.imagemagick.org/script/index.php"><em>ImageMagick</em></a> is an extremely popular library and binary for manipulating images. Amongst other uses, it’s often used to process user generated content, such as avatar images, product photos or displaying images in social channels.</p>

<p>Earlier this week, multiple severe and easily exploited vulnerabilities in <em>ImageMagick</em> <a href="https://imagetragick.com/">were disclosed</a>. There is no official fix yet, but it can be mitigated by disabling certain features.</p>

<p>In Node.js, <em>ImageMagick</em> is primarily invoked through the <a href="https://www.npmjs.com/package/imagemagick">imagemagick</a> npm package. This package has not been updated in over 2 years, and so is unlikely to help address this issue.</p>

<p>To help protect Node.js <em>ImageMagick</em> users, we released a dedicated npm package called <em>imagemagick-safe</em>, which disables the vulnerable features. We also submitted a <a href="https://github.com/rsms/node-imagemagick/pull/131">pull request</a> with this change to the <a href="https://www.npmjs.com/package/imagemagick">imagemagick</a> package GitHub repo, but given the repo’s inactivity we think it’s unlikely it’ll be pulled in.</p>

<h2 id="imagetragick---background">ImageTragick - Background</h2>

<p>The disclosed vulnerabilities are especially noteworthy for three reasons:</p>

<ol>
  <li>Some of the vulnerabilities are extremely severe, allowing attackers to execute commands on the server, expose server files and more.</li>
  <li>The vulnerabilities trivial to exploit, requiring no more than a purpose-built image file and a single HTTP request.</li>
  <li>There is still no official fix available.</li>
  <li>The vulnerabilities are known to be exploited in the wild.</li>
</ol>

<p>The whole incident is being referred to as <em>ImageTragick</em>, and you can find more detail about the vulnerabilities on its <a href="https://imagetragick.com/">dedicated website, imagetragick.com</a>.</p>

<h2 id="cause--mitigation">Cause & Mitigation</h2>

<p>From what is currently known, the vulnerabilities are all related to the processing of specific image file types and keywords, such as MVG, URL and HTTPS. The file types support an internal action (e.g. include another image) and the processing of these actions is not sufficiently sanitized, enabling remote command execution. Here’s an example of an exploit, copied from the <a href="https://imagetragick.com">official vulnerability website</a>:</p>

<p>Sample exploit.svg</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5
6
7
8
9</pre></td><td class="code"><pre><span style="color: #75715e"><?xml version="1.0" standalone="no"?></span>
<span style="color: #75715e"><!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd";></span>
<span style="color: #f92672"><svg</span> <span style="color: #a6e22e">width=</span><span style="color: #e6db74">"640px"</span> <span style="color: #a6e22e">height=</span><span style="color: #e6db74">"480px"</span> <span style="color: #a6e22e">version=</span><span style="color: #e6db74">"1.1"</span>
<span style="color: #a6e22e">xmlns=</span><span style="color: #e6db74">"http://www.w3.org/2000/svg"</span><span style="color: #960050">;</span> <span style="color: #a6e22e">xmlns:xlink=</span>
<span style="color: #e6db74">"http://www.w3.org/1999/xlink"</span><span style="color: #960050">;</span><span style="color: #f92672">></span>
<span style="color: #f92672"><image</span> <span style="color: #a6e22e">xlink:href=</span><span style="color: #e6db74">"https://example.com/image.jpg&quot;|ls &quot;-la"</span>
<span style="color: #a6e22e">x=</span><span style="color: #e6db74">"0"</span> <span style="color: #a6e22e">y=</span><span style="color: #e6db74">"0"</span> <span style="color: #a6e22e">height=</span><span style="color: #e6db74">"640px"</span> <span style="color: #a6e22e">width=</span><span style="color: #e6db74">"480px"</span><span style="color: #f92672">/></span>
<span style="color: #f92672"></svg></span>
</pre></td></tr></tbody></table>
</div>
</div>

<p>Sample execution</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4</pre></td><td class="code"><pre>$ convert exploit.mvg out.png
total 32
drwxr-xr-x 6 user group 204 Apr 29 23:08 .
drwxr-xr-x+ 232 user group 7888 Apr 30 10:37 ..
</pre></td></tr></tbody></table>
</div>
</div>

<p>The way to disable support for these formats and keywords is by editing <em>ImageMagick</em>’s (optional) <em>policy.xml</em> configuration file, ensuring it includes the content below. Doing so will <em>actually remove some functionality</em>, but will also address the vulnerabilities. Note that editing the policy is not guaranteed to address all possible exploits, but the vulnerability researchers claim it blocks all those currently known to them.</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11</pre></td><td class="code"><pre><span style="color: #f92672"><policymap></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"EPHEMERAL"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"URL"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"HTTPS"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"MVG"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"MSL"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"TEXT"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"SHOW"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"WIN"</span> <span style="color: #f92672">/></span>
  <span style="color: #f92672"><policy</span> <span style="color: #a6e22e">domain=</span><span style="color: #e6db74">"coder"</span> <span style="color: #a6e22e">rights=</span><span style="color: #e6db74">"none"</span> <span style="color: #a6e22e">pattern=</span><span style="color: #e6db74">"PLT"</span> <span style="color: #f92672">/></span>
<span style="color: #f92672"></policymap></span>
</pre></td></tr></tbody></table>
</div>
</div>

<h2 id="imagemagick-safe-npm-package">imagemagick-safe npm package</h2>

<p>Given that the primary <a href="https://www.npmjs.com/package/imagemagick">imagemagick</a> package is not active, we created a new package called <a href="https://npmjs.com/package/imagemagick-safe"><em>imagemagick-safe</em></a>.</p>

<p>This package forces the use of the above policy, disabling features but removing vulnerabilities in the process. We recommend you use it until new ImageMagick binaries are released, and you’ve confirmed your systems are upgraded.</p>

<p>Note you can also use ImageMagick via the <a href="https://npmjs.com/package/gm"><em>gm</em></a> package, we’re looking at submitting a pull request there too.</p>

<p>If you find issues or otherwise have feedback about this package, please let us know at <a href="mailto:support@snyk.io">support@snyk.io</a>.</p>