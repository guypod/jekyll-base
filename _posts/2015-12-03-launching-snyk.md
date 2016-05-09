---
ID: 13
post_title: Launching Snyk
author: wpengine
post_date: 2015-12-03 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=13
published: true
---
<p>I’m excited to announce Snyk is now live!<br />
Snyk helps you find and fix known vulnerabilities in your Node.js dependencies. These are publicly documented security holes, making them easy for attackers to track and exploit.</p>

<p>Snyk’s goal is to make it even easier for you to fix them first. Note that Snyk focuses on fixing security issues, finding them is just a necessary step along the way.</p>

<p>We soft-launched Snyk at Velocity conference a month back. Here’s our keynote video showing the problem, a demo of Snyk (though this was before we added the <a href="https://snyk.io/docs#wizard">wizard</a>) and a live exploit!</p>

<p><iframe width="560" height="315" src="https://www.youtube.com/embed/iXA14OFXxZA" frameborder="0" allowfullscreen=""></iframe></p>

<p>To secure a project, <a href="https://snyk.io/docs#installation--getting-started">install <code class="highlighter-rouge">snyk</code> using npm</a> and run Snyk’s <a href="https://snyk.io/docs#wizard"><code class="highlighter-rouge">wizard</code></a>. The wizard will help secure your project in several steps:</p>

<ul>
  <li>Use Snyk’s API to match your dependencies against our <a href="https://github.com/Snyk/vulndb">open source vulnerability database</a>.</li>
  <li>Help you understand and fix each security issue found.</li>
  <li>Suggest the best direct dependency upgrades that will close the security holes.</li>
  <li>When an upgrade isn’t available, determine if one of our security team’s <a href="https://github.com/Snyk/vulndb#patches">patches</a> can fix the issue.</li>
  <li>If neither an upgrade nor a patch is available, remember the current state. We’ll notify you when a new remediation path is made available.</li>
</ul>

<div>
<pre class="code">$ snyk wizard

<span class="syn--green">?</span> <span class="syn--white syn--bold">High severity vulnerability found in bassmaster@1.5.1
  - info: <a title="Vulnerability report" href="https://snyk.io/vuln/npm:bassmaster:20140927">https://snyk.io/vuln/npm:bassmaster:20140927</a>
  - from: myapp@0.0.0 > bassmaster@1.5.1</span> <span class="syn--blue">Upgrade</span>

<span class="syn--green">?</span> <span class="syn--white syn--bold">Low severity vulnerability found in hapi@10.5.0
  - info: <a title="Vulnerability report" href="https://snyk.io/vuln/npm:hapi:20151020">https://snyk.io/vuln/npm:hapi:20151020</a>
  - from: myapp@0.0.0 > hapi@10.5.0</span> <span class="syn--blue">Ignore</span>
<span class="syn--green">?</span> <span class="syn--white syn--bold">[audit] Reason for ignoring vulnerability?</span> <span class="syn--blue">Not Exploitable</span>

<span class="syn--green">?</span> <span class="syn--white syn--bold">Low severity vulnerability found in ms@0.1.0
  - info: <a title="Vulnerability report" href="https://snyk.io/vuln/npm:ms:20151024">https://snyk.io/vuln/npm:ms:20151024</a>
  - from: myapp@0.0.0 > mongoose@4.1.12 > ms@0.1.0</span>
  Upgrade to mongoose@4.2.4
<span class="syn--blue">❯ Patch (modifies files locally, updates policy)</span>
  Set to ignore for 30 days
  Skip</pre>
  <br />
</div>

<p>Once you’re vulnerability free, you can use <a href="https://snyk.io/docs#test"><code class="highlighter-rouge">snyk test</code></a> in your CI/CD systems to avoid shipping with vulnerabilities and <a href="https://snyk.io/docs#protect"><code class="highlighter-rouge">snyk protect</code></a> to patch the vulnerabilities you chose. Using <a href="https://snyk.io/docs#monitor"><code class="highlighter-rouge">snyk monitor</code></a> will remember which dependencies you use, so we can notify you when a newly disclosed vulnerability affects them. You can read the full details about Snyk and its commands <a href="https://snyk.io/docs">in our docs</a>.</p>

<p class="u--centered"><img src="https://res.cloudinary.com/snyk/image/upload/v1449224934/screenshot-monitor-email.png" alt="A screenshot of the email that Snyk sends when new vulnerabilities have been disclosed." /></p>

<p>Lastly, if you’re the creator of an open source package, use Snyk to ensure you’re not distributing vulnerabilities to your users. Upgrade dependencies to fix such issues where possible, and use <a href="https://snyk.io/docs#protect"><code class="highlighter-rouge">snyk protect</code></a> to patch them <code class="highlighter-rouge">postinstall</code> when you can’t. Once your package has no security issues, put a <a href="https://snyk.io/docs#badge">badge</a> on your README showing it has no known security holes. This will show your users you care about security, and tell them that they should care too.</p>

<p class="u--centered">
  <svg id="snyk-badge" xmlns="http://www.w3.org/2000/svg" width="113" height="20">
    <lineargradient id="b" x2="0" y2="100%">
      <stop offset="0" stop-color="#bbb" stop-opacity=".1" />
      <stop offset="1" stop-opacity=".1" />
    </lineargradient>
    <mask id="a">
      <rect width="113" height="20" rx="3" fill="#fff" />
    </mask>
    <g mask="url(#a)">
      <path fill="#555" d="M0 0h90v20H0z" />
      <path fill="#4c1" d="M90 0h113v20H90z" />
      <path fill="url(#b)" d="M0 0h113v20H0z" />
    </g>
    <g fill="#fff" text-anchor="middle" font-family="DejaVu Sans,Verdana,Geneva,sans-serif" font-size="11">
      <text x="45" y="15" fill="#010101" fill-opacity=".3">Vulnerabilities</text>
      <text x="45" y="14">Vulnerabilties</text>
      <text x="100" y="15" fill="#010101" fill-opacity=".3">0</text>
      <text x="100" y="14">0</text>
    </g>
  </svg>
  <svg id="snyk-badge" xmlns="http://www.w3.org/2000/svg" width="116" height="20">
    <lineargradient id="b" x2="0" y2="100%">
      <stop offset="0" stop-color="#bbb" stop-opacity=".1" />
      <stop offset="1" stop-opacity=".1" />
    </lineargradient>
    <mask id="a">
      <rect width="116" height="20" rx="3" fill="#fff" />
    </mask>
    <g mask="url(#a)">
      <path fill="#555" d="M0 0h90v20H0z" />
      <path fill="#e05d44" d="M90 0h116v20H90z" />
      <path fill="url(#b)" d="M0 0h116v20H0z" />
    </g>
    <g fill="#fff" text-anchor="middle" font-family="DejaVu Sans,Verdana,Geneva,sans-serif" font-size="11">
      <text x="45" y="15" fill="#010101" fill-opacity=".3">Vulnerabilities</text>
      <text x="45" y="14">Vulnerabilties</text>
      <text x="102" y="15" fill="#010101" fill-opacity=".3">15</text>
      <text x="102" y="14">15</text>
    </g>
  </svg>
</p>

<p>Snyk is in beta, and we encourage all of you to try it out. If you use it, please share your feedback with us on <a href="https://twitter.com/snyksec">@snyksec</a> or by emailing <a href="mailto:support@snyk.io">support@snyk.io</a>. Try it out, and help make building and consuming open source secure!</p>