---
ID: 10
post_title: Tackling the new npm@3 dependency tree
author: wpengine
post_date: 2016-02-25 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=10
published: true
---
<p>Until recently Snyk’s CLI tool only supported npm@2. That all changed when we released snyk@1.9.0 and added full support for the new npm@3 directory structures.</p>

<p>We wanted to share some of the technical challenges involved and the new tooling that came out of the process.</p>

<h2 id="whats-different-about-npm3">What’s different about npm@3</h2>

<p>With npm@2 node dependencies would be installed into the <code class="highlighter-rouge">node_modules</code> directory of each respective node package. For example, the directories for the <a href="http://www.npmjs.com/package/request">request</a> module looks like this:</p>

<pre>
  request/node_modules
  ├── aws-sign2
  ├── aws4
  │   └── node_modules
  │       └── lru-cache
  │           └── test
  ├── bl
  │   ├── node_modules
  │   │   └── readable-stream
  │   │       ├── doc
  │   │       │   └── wg-meetings
  │   │       └── node_modules
  │   │           ├── core-util-is
  ...snipped
</pre>

<p>As you can see from the snippet above the <code class="highlighter-rouge">node_modules</code> appears a number of times already. This isn’t so bad, but it can lead to a lot of duplication. Popular utilities like <a href="https://lodash.com/">lodash</a> can appear in a project many, many times!</p>

<p>Originally, npm@2 does some work to de-duplicate this, but npm@3 completely flattens this directory structure in a bid to completely remove the duplication. The same request package looks like this with npm@3:</p>

<pre>
  request/node_modules
  ├── ansi-regex
  ├── ansi-styles
  ├── asn1
  │   └── lib
  ├── assert-plus
  ├── async
  │   └── lib
  ├── aws-sign2
  ├── aws4
  ├── bl
  │   └── test
  ...snipped
</pre>

<p>As you can see, it’s completely different, but importantly the way node requires modules is not affected at all.</p>

<h2 id="how-does-this-affect-snyk">How does this affect Snyk?</h2>

<p>To start with, the CLI package walking logic needed a complete rewrite. Originally Snyk would walk your <code class="highlighter-rouge">node_modules</code> directory, then iterate through each sub-directory and build up a tree representation of your packages. Relatively simple really.</p>

<p>Except now, the flat directory structure with npm@3 does not represent your package relationships at all. For example the <code class="highlighter-rouge">async</code> package in the npm@3 listing above, is actually a dependency of the <code class="highlighter-rouge">form-data</code> package, which in turn is a dependency of <code class="highlighter-rouge">request</code>. But you can’t see that from the file tree.</p>

<p>So Snyk’s package resolution has been completely rewritten and extracted out into a standalone module called <a href="https://github.com/Snyk/resolve-deps">snyk-resolve-deps</a> (open source under an Apache 2 license).</p>

<p>This module is used inside of the <a href="https://www.npmjs.com/package/snyk">snyk</a> CLI tool but can also be installed as a standalone CLI tool (installed using <code class="highlighter-rouge">npm install -g snyk-resolve-deps</code> gives a utility called <code class="highlighter-rouge">snyk-resolve</code>).</p>

<p>What the snyk-resolve-deps does is: first pass through the entire directory structure building up the physical tree. This physical tree is then passed to the next stage that creates a <em>logical tree</em>, which is the structure that represents where packages can be loaded from.</p>

<p>This means that both npm@2 and npm@3 directory structures are supported and create a virtual tree looking like this:</p>

<pre>
  ❯ snyk-resolve
  request@2.69.1
  ├── aws-sign2@0.6.0
  ├─┬ aws4@1.2.1
  │ └── lru-cache@2.7.3
  ├─┬ bl@1.0.2
  │ └─┬ readable-stream@2.0.5
  │   ├── core-util-is@1.0.2
  ...snipped
</pre>

<p>Ultimately this means Snyk can now happily support both your npm@3 installed projects just as well as npm@2. We’re able to find the correct paths for patching and able to report all the vulnerable paths accurately.</p>

<h2 id="how-is-this-different-from-npm-ls">How is this different from ‘npm ls’?</h2>

<p>If you’re familiar with npm’s tools, you would have heard of <code class="highlighter-rouge">npm ls</code> which is useful to see these trees.</p>

<p>The big difference between <code class="highlighter-rouge">snyk-resolve-deps</code> and <code class="highlighter-rouge">npm ls</code> is that our tool will show the complete logical tree and shows all ways through which a package entered your project.</p>

<p>If we look at how the <code class="highlighter-rouge">request</code> is included in the <code class="highlighter-rouge">npm</code> code (on the <code class="highlighter-rouge">2.x</code> branch) we can see that <code class="highlighter-rouge">npm ls</code> is telling us one story (of what’s available on disk):</p>

<p><img src="https://cldup.com/pJvi9ChIpH.png" alt="npm ls output" /></p>

<p>Whereas our own method of resolving tells a different story. This doesn’t mean that <code class="highlighter-rouge">npm ls</code> is wrong, it’s that we want to know exactly <strong>what</strong> is loading <code class="highlighter-rouge">request</code>, and our own <code class="highlighter-rouge">snyk-resolve-deps</code> can give us that:</p>

<p><img src="https://cldup.com/n7L4BHBAp5.png" alt="npm ls output" /></p>

<p>As you can see, the <code class="highlighter-rouge">request</code> module is depended upon by many more packages than you might initially think. <em>If</em> that package had a vulnerability, the vulnerable paths are clearly known to Snyk now.</p>

<h2 id="snyk-resolve-deps">snyk-resolve-deps</h2>

<p>The package <a href="https://github.com/Snyk/resolve-deps">snyk-resolve-deps</a> is available today, under an Apache 2 open source license. Once installed globally, it’s available under the alias of <code class="highlighter-rouge">snyk-resolve</code>.</p>

<p>The CLI utility has a number of filters and flags, including <code class="highlighter-rouge">--disk</code> view (which reports very similarly to <code class="highlighter-rouge">npm ls</code>) and <code class="highlighter-rouge">--filter X</code> and <code class="highlighter-rouge">--count X</code> to filter and find occurrences of a specific dependency. You can find out more with <code class="highlighter-rouge">snyk-resolve --help</code>.</p>

<hr />

<p>You can <code class="highlighter-rouge">npm install -g snyk</code> today and anonymously <a href="https://snyk.io/docs#test">test</a> any package or <a href="https://snyk.io/docs#git-url-formats">github repo</a> and with a free account, you can <a href="https://snyk.io/docs/using-snyk/#monitor">start to monitor</a> your projects for vulnerabilities <strong>today</strong>.</p>