---
ID: 14
post_title: 10 Reasons To Use HTTPS
author: wpengine
post_date: 2015-07-10 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=14
published: true
---
<p>HTTPS, HTTP over TLS, has been around since 1994, and has been well adopted by the security sensitive web — online banking, shopping, taxes and more. However, the vast majority of websites (est. <a href="http://www.securitee.org/files/mixedinc_isc2013.pdf">81%</a> to <a href="http://trends.builtwith.com/ssl/SSL-by-Default">97%</a>) continue to communicate using clear (unencrypted) HTTP — no matter how insecure that is.</p>

<figure class="u--pull">
<img src="https://res.cloudinary.com/snyk/image/upload/v1448374341/BuiltWith-SSL-Usage.png" alt="A graph showing websites that are using SSL by default." />
<figcaption>
<a href="http://trends.builtwith.com/ssl/SSL-by-Default">BuiltWith</a> stats showing small but growing HTTPS adoption
</figcaption>
</figure>

<p>Today, however, there are more reasons than ever to switch to HTTPS — even for a news site, corporate site, or any site that doesn’t consider itself at the top of the security food chain. HTTPS adoption <a href="https://twitter.com/builtwith/status/614762645032464384">grew 80% last year</a> alone, much faster than previous years, but we’re still <strong><em>very far</em></strong> from encryption being the norm. If you’re not convinced HTTPS is right for you, or need ammo to convince your peers and bosses, here are 10 good reasons to go HTTPS.</p>

<h3 id="protect-your-users-privacy">1. Protect Your Users’ Privacy</h3>
<p>First and foremost, <a href="https://www.schneier.com/blog/archives/2015/06/why_we_encrypt.html">HTTPS protects your users</a>. Posting a news update on a user forum may cost a dissident his life in an oppressive regime ; A strict workplace may terminate employment based on an employee’s browsing activity; And of course, the Snowden affairs have clearly shown governments simply can’t get enough of this data. Using HTTPS makes it dramatically harder for these players to know what users are doing, and helps you maintain your most important responsibility — your users trust.</p>

<h3 id="search-engine-ranking-seo">2. Search Engine Ranking (SEO)</h3>
<p>Last year (2014), Google <a href="http://googleonlinesecurity.blogspot.co.uk/2014/08/https-as-ranking-signal_6.html">announced</a> HTTPS will be a factor in its ranking. Google, like other search engines, aims to promote the links that are best for the user, and a website protecting the user’s privacy using HTTPS is better (for the user) than one that isn’t. Whether you see this as a carrot (use HTTPS and be rewarded) or stick (use HTTPS or be downgraded), few things mobilize businesses as well as SEO. For more detailed advice on how to tune HTTPS for SEO purposes, check out <a href="https://moz.com/blog/seo-tips-https-ssl">this Moz article</a>.</p>

<h3 id="protect-your-brand">3. Protect Your Brand</h3>
<p>Unencrypted content can easily be tampered. A user browsing a news site on a malicious wifi <a href="http://www.computerworld.com/article/2470580/endpoint-security/hackers-use-hidden-device-to-manipulate-news-at-wi-fi-hotspots.html">may read reported facts that never took place</a>. A corporate web may show an apology for an embezzling CEO that never really existed. And any insecure website can be made to show porn (<a href="http://arstechnica.com/security/2015/03/massive-denial-of-service-attack-on-github-tied-to-chinese-government/">or worse</a>). In addition, unencrypted pages are often in the path to secure ones. For instance, consider a shopping site where a product page is unencrypted, but the actual purchase flow uses HTTPS. A <a href="https://en.wikipedia.org/wiki/Man-in-the-middle_attack">man-in-the-middle</a> can change the unprotected product page, making the “Add to Cart” button go to their evil copy of the website, and the browser (and user) will see no difference. If you only want your users to see the content you actually posted, and want their actions to always reach you, use HTTPS.</p>

<h3 id="browsers-to-mark-http-as-insecure">4. Browsers to mark HTTP as Insecure</h3>
<p>Today, browsers mark HTTPS by <a href="https://support.google.com/chrome/answer/95617?p=ui_security_indicator&rd=1">painting the URL green or adding a lock</a> only if the site is properly encrypted. These markers actually convey a false sense of security, stating HTTPS websites are secure when they may be woefully insecure once you reach the server. What <strong>is</strong> accurate is that <strong><em>unencrypted</em></strong> HTTP connections are <strong><em>insecure</em></strong><em>.</em> Browsers are now poised to update their indicators accordingly. Both <a href="https://www.chromium.org/Home/chromium-security/marking-http-as-non-secure">Google</a> & <a href="https://blog.mozilla.org/security/2015/04/30/deprecating-non-secure-http/">Firefox</a> have stated they’ll <strong>mark HTTP sites first as <em>dubious</em> and then as <em>insecure</em></strong>, possibly as early as this year. If you remain on HTTP, your users may be explicitly told you are insecure, reducing their trust in you. Note: The effectiveness of passive markers, positive or negative, <a href="http://devd.me/papers/alice-in-warningland.pdf">is minimal</a>, and browsers may evolve those warnings to actual click through warnings.</p>

<h3 id="http-2">5. HTTP 2</h3>

<figure>
  <iframe width="420" height="315" src="https://www.youtube.com/embed/GIDXISQs67w" frameborder="0" allowfullscreen=""></iframe>
  <figcaption>Introduction to HTTP2 Video, a part of the <a href="https://developer.akamai.com/stuff/Video_Gallery/Web_Performance_Series.html">Akamai Web Performance Videos Series</a>
  </figcaption>
</figure>

<p>Roughly 18 years after its inception, HTTP/1.1 is <em>finally</em> getting refreshed. It’s successor, <a href="https://http2.akamai.com/">HTTP/2</a>, has been <a href="https://www.mnot.net/blog/2015/06/15/http2_implementation_status">officially completed</a> in May (2015). HTTP/2 further evolves Google’s <a href="https://developers.google.com/speed/spdy/">SPDY</a>, and includes many significant improvements over HTTP/1.1, ranging from request multiplexing to header compression to server-side push. For compatibility reasons, as well as a desire to make the web secure, browsers will only support HTTP/2 over HTTPS (the <a href="https://tools.ietf.org/html/rfc7540">spec</a> states <a href="https://http2.github.io/faq/#does-http2-require-encryption">encryption is optional</a>). If you want to benefit from this evolution of the web — you need to switch to HTTPS.</p>

<h3 id="service-worker--powerful-features">6. Service Worker & Powerful Features</h3>

<p>Another exciting browser advancement, similar in impact to HTTP/2 (if less well known), is <a href="http://www.html5rocks.com/en/tutorials/service-worker/introduction/">ServiceWorker</a>. ServiceWorker’s main claim to fame is enabling web apps to act like native apps, allowing them to work offline, receive <a href="https://github.com/gauntface/simple-push-demo">push notifications</a> and more. It’s a new and improved replacement to <a href="http://alistapart.com/article/application-cache-is-a-douchebag">AppCache</a>, built with the principles of the <a href="https://extensiblewebmanifesto.org/">Extensible Web Manifesto</a>. Technically, ServiceWorker works as a JavaScript proxy, seamlessly installed when a user visits a site. This proxy is extremely powerful, and can extend the page in ways much broader than offline support. If a malicious ServiceWorker is installed, it can cause extreme and persistent damage, far greater than regular tampering of HTTP. To mitigate this risk, ServiceWorker, with all its glorious new capabilities, is only supported over HTTPS. Browsers are <a href="http://www.w3.org/2001/tag/doc/web-https">pushing for a secure web</a>, and you can expect more powerful features, such as geolocation and full screen, <a href="https://w3c.github.io/webappsec/specs/powerfulfeatures/">to work only on HTTPS</a>.</p>

<figure>
  <iframe width="420" height="315" src="https://www.youtube.com/embed/4uQMl7mFB6g" frameborder="0" allowfullscreen=""></iframe>
  <figcaption>Introduction Video to ServiceWorker, by Jake Archibald</figcaption>
</figure>

<h3 id="protect-your-revenue-from-proxies">7. Protect Your Revenue (From Proxies)</h3>
<p>Criminals are not the only ones looking to make money of your site — <a href="https://tools.ietf.org/html/draft-hildebrand-middlebox-erosion-00">Internet and WiFi providers want in on it too</a>. As many as <a href="https://blog.haschek.at/2015-lets-analyze-twenty-thousand-proxies">38% of WiFi proxies</a>, ranging from giants like <a href="http://arstechnica.com/tech-policy/2014/09/why-comcasts-javascript-ad-injections-threaten-security-net-neutrality/">Comcast</a> to <a href="https://en.dailysocial.net/post/the-unethical-advertising-behaviors-of-mobile-telcos">smaller</a> <a href="http://arstechnica.com/tech-policy/2013/04/how-a-banner-ad-for-hs-ok/">providers</a>, inject their own ads on unencrypted pages. If ads are how you make money, know those ads may be hijacked, and your users will be none the wiser. If your website is not using ads… Your users may see some anyway, and blame you for it. Use HTTPS to prevent such tampering and protect your revenue & brand.</p>

<figure>
  <img src="https://res.cloudinary.com/snyk/image/upload/v1448374341/Ad-On-Apple-Page.jpg" alt="A screenshot of an ad overlaid on a website" />
<figcaption>
  An H&R Block ad on the Apple Store page, <a href="http://arstechnica.com/tech-policy/2013/04/how-a-banner-ad-for-hs-ok/">injected by CMA in 2013</a>" (And yet, Apple Store homepage is still not using HTTPS )
</figcaption>
</figure>

<h3 id="better-analytics-https-referrers">8. Better Analytics: HTTPS Referrers</h3>
<p>HTTPS aims to protect your privacy, including not sharing with others what you’re browsing. Imagine you browse <a href="https://secret.com/">https://secret.com/HelloKitty/</a> and click a link to the <strong>unencrypted</strong> <a href="http://other.com/">http://other.com/</a>. If the request to other.com included the URL in a Referer (sic) header, anyone listening (as well as other.com) would know of your love for the little <a href="http://www.bbc.co.uk/newsbeat/article/28963085/hello-kitty-is-not-a-cat---shes-a-british-school-kid">not-a-cat</a>. To avoid such a violation, browsers do not send a Referer header when navigating from HTTPS to HTTP (unless <a href="http://w3c.github.io/webappsec/specs/referrer-policy/">explicitly overridden using a Referrer Policy</a>). As more websites switch to HTTPS, staying on HTTP would hurt your insight into where your visitors are coming from.</p>

<h3 id="ios-9android-5-api-compatibility">9. iOS 9+/Android 5+ API Compatibility</h3>
<p>The push for TLS doesn’t end in the browser. In their last developer conference, Apple announced that iOS 9 will require all connections to use TLS. Quoting from the <a href="https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html">iOS 9 documentation</a>:</p>

<blockquote>
  <p>“If you’re developing a new app, you should use HTTPS exclusively. If you have an existing app, you should use HTTPS as much as you can right now, and create a plan for migrating the rest of your app as soon as possible. In addition, your communication through higher-level APIs needs to be encrypted using TLS version 1.2 with forward secrecy. If you try to make a connection that doesn’t follow this requirement, an error is thrown”.</p>
</blockquote>

<p><a href="https://koz.io/android-m-and-the-war-on-cleartext-traffic/">Android M has will have a similar (a bit less strict) requirement</a>, implying this will be the default behavior on native. This change in defaults matters not only to native app developers, but to anyone creating content that <em>may</em> be consumed by native iOS apps. If you don’t support HTTPS, the bar to have iOS developers adopt your API will be that much higher.</p>

<h3 id="third-party-inclusion">10. Third Party Inclusion</h3>
<p>To maintain the secure page content, an HTTPS page cannot load an insecure HTTP resource. Doing so will usually fail with a <a href="https://developer.mozilla.org/en/docs/Security/MixedContent"><em>Mixed Content Warning</em></a><em>,</em> with a few small exceptions such as sandboxed iframes. Websites attempting to switch to HTTPS often find that many of the third party components they use do not support HTTPS. As a result, they need to either stay on HTTP, or remove the third party in question. While most websites have historically opted to stay on HTTP, the balance may shift as the motivation to go secure grows. If you produce content other sites may embed, you should quickly add support for HTTPS. It may give you a competitive advantage today, and soon enough you won’t be able to survive without it.</p>

<h3 id="start-your-https-plan-today">Start Your HTTPS Plan Today!</h3>
<p>This post enumerated some of the reasons to go HTTPS, but not the challenges involved in doing so. It’s quite likely that insecure third parties, costs or other legacy reasons (<a href="https://istlsfastyet.com/">but not performance!</a>) make the switch hard. Still, it’s important to understand HTTPS is not going anywhere. The longer you wait before you start switching, the deeper the hole you’re digging yourself into. New initiatives offer free and easy <a href="http://letsencrypt.org/">certificates</a>, <a href="https://www.cloudflare.com/ssl">CDN</a>, <a href="https://www.ssllabs.com/ssltest/">testing</a> and more, and can help you overcome some of the obstacles you meet. So start now: put a plan in place, get it approved, and get on the path to being secure.</p>