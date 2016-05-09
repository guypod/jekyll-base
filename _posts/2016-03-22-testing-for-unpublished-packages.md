---
ID: 9
post_title: Testing for unpublished packages
author: wpengine
post_date: 2016-03-22 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=9
published: true
---
<p>Yesterday, Azer Koçulu <a href="https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c#.fir4ivq5f">unpublished</a> nearly <a href="https://gist.githubusercontent.com/azer/db27417ee84b5f34a6ea/raw/50ab7ef26dbde2d4ea52318a3590af78b2a21162/gistfile1.txt">300 packages</a>, notably including <em>left-pad</em>, which is used by top projects like Node & Babel. The <em>npm</em> team <a href="https://twitter.com/seldo/status/712414400808755200">un-unpublished</a> <em>left-pad</em>, but the remaining packages <a href="https://medium.com/@sheerun/as-callmevlad-mentioned-on-hackernews-ef571c3fe3f4#.riv8abimk">remained exposed</a> for malicious actors to grab.</p>

<p>When a package is completely unpublished, <em>npm</em> (currently) allows anyone to publish a new version for that package’s name. When Azer’s popular packages disappeared, several npm users jumped in to do just that, presumably focusing on making their build work. <strong>However, there’s no telling whether some packages were grabbed by malicious actors - or will be in the future</strong>.</p>

<p>Such a malicious user can then publish a <em>patch</em> release to each major and minor stream, incrementing the version number by 0.0.1. Most consumers of this package will likely use a <a href="https://docs.npmjs.com/getting-started/semantic-versioning">semver range</a> to pull it in, e.g. <code class="highlighter-rouge">pkg@^1.2.1</code>, intending to take in bug fixes as they come. As a result, they will pull in the new version blindly, running the malicious code without even knowing it.</p>

<p>It’s important to find out whether you are exposed to this risk or not. To help you identify whether your dependency tree includes these now-risky packages, we enhanced Snyk to support a new <code class="highlighter-rouge">test-unpublished</code> command. To use it, run the following:</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3</pre></td><td class="code"><pre><span style="color: #f6aa11">cd</span> ~/myproj/
npm install -g snyk
snyk <span style="color: #f6aa11">test</span>-unpublished
</pre></td></tr></tbody></table>
</div>
</div>

<p>Snyk will iterate your project’s dependencies, and print out only the ones that have been unpublished. Note that this is currently limited only to the packages Azer just unpublished, as opposed to all unpublished packages.</p>

<p>The output will look roughly like this:</p>

<p class="u--centered">
<img src="https://res.cloudinary.com/snyk/image/upload/v1458746722/blog-test-unpublished-scnreeshot.png" alt="" />
</p>

<p>These packages are risky, as they may be (or may have been) republished by a malicious actor. We recommend you inspect them to ensure their content is what you expect. You can also compare them to the contents of the equivalent repository on <a href="https://github.com/azer?tab=repositories">Azer’s GitHub account</a>. If you’re using the dependency directly, you can also switch to pulling it directly from that GitHub repo.</p>

<p>Note that, while risky, these dependencies are not necessarily vulnerable. Therefore, we’ve decided not to include them in the standard <code class="highlighter-rouge">snyk test</code> command, which focuses on finding more guaranteed security holes in your dependencies.</p>

<p>We’ve created <code class="highlighter-rouge">snyk test-unpublished</code> quickly, to help <em>npm</em> users deal with the current emergency. We expect to iterate on it, please let us now over Twitter (<a href="https://twitter.com/snyksec">@snyksec</a>) if you have any feedback or suggestions regarding it.</p>