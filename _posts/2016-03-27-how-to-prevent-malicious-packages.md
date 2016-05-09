---
ID: 8
post_title: How to prevent malicious packages
author: wpengine
post_date: 2016-03-27 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=8
published: true
---
<p>Last week, CERT <a href="http://www.kb.cert.org/vuls/id/319816">alerted users</a> to the risk of publishing or consuming a malicious npm package. While not unique to <em>npm</em>, this substantial risk is more likely to happen in this ecosystem. It should be noted that while this is definitely an attack vector, in my opinion it’s not a <em>vulnerability</em> as much as an insecure default.</p>

<p>This post explains the risk and how you can protect yourself. We’ll see the tradeoffs that triggered the problem, review an exploit scenario, and share how to mitigate the risk in your own environments. If you just want the action items, jump to the <a href="#preventing-malicious-publishing-public">mitigation sections</a>.</p>

<h2 id="the-attack">The attack</h2>

<h3 id="malicious-packages-in-5-easy-steps">Malicious Packages in 5 easy steps</h3>
<p>At the heart of the problem lies the ease of publishing a new package. Like <em>git</em>, <a href="https://docs.python.org/2/distutils/packageindex.html#the-pypirc-file"><em>pip</em></a> and others, <em>npm</em> allows users to store their credentials in a dotfile (<code class="highlighter-rouge">.npmrc</code>), and then publish without interactive prompts.</p>

<p>Unfortunately, this is one case where you can make something too easy. The streamlined publish process increases the chance of publishing by mistake, but more importantly it opens the door for attackers to intentionally publish malicious code.</p>

<p>To publish a malicious package version, all the attacker needs to do is:</p>

<ol>
  <li>Run code on the developers machine.</li>
  <li>Determine who’s the logged in npm user (<code class="highlighter-rouge">npm whoami --silent</code>)</li>
  <li>Download one of the user’s packages to a temp folder</li>
  <li>Edit package.json: add a malicious <code class="highlighter-rouge">postinstall</code> hook and increment version by 0.0.1</li>
  <li>Publish (<code class="highlighter-rouge">npm publish</code>)</li>
</ol>

<p>Of these, the only challenging step is the first one. Unfortunately, most developers tend to run their environments pretty loosely…</p>

<p>Did you ever install something via <code class="highlighter-rouge">curl X | bash</code>? That’s <a href="https://speakerdeck.com/barnbarn/security-for-non-unicorns-2?slide=36">one way for an attacker to get in</a> - they don’t even need <code class="highlighter-rouge">sudo bash</code>. Alternatively, perhaps you occasionally install an npm package locally, just to try it out? Any one if its dependencies can have a <code class="highlighter-rouge">postinstall</code> hook running the above.</p>

<p>Fact is, we developers <em>experiment</em>, which naturally means we’re not using pristine environments. Such environment should not have publishing rights.</p>

<h3 id="fast-propagation-slow-detection">Fast propagation, slow detection</h3>
<p>Once done, the attacker can just sit back and relax while his code propagates across the Node ecosystem. Most consumers of this package will load it using a <a href="https://docs.npmjs.com/getting-started/semantic-versioning">semver range</a> that implicitly takes patch (0.0.X) versions, and will take this new version without. They can also make the malicious code propagate itself to packages owned by its victims, creating an even faster propagating <em>worm</em>.</p>

<p>To make things worse, if such a malicious package <em>was</em> published, there’s no easy way to spot it happened. <em>npm</em> doesn’t notify the user when a new release is published, and there’s no simple way to see delta between packages (unlike <em>git</em>). Your best chance at spotting such an issue is noticing the new version number. If you’re actively maintaining a single-developer project, you’re indeed likely to notice version changes, but for dormant projects or ones developed by a full team, those can easily slip through.</p>

<h2 id="mitigation-techniques">Mitigation techniques</h2>

<h3 id="how-to-prevent-malicious-publishing-public">How to prevent malicious publishing (public)</h3>
<p>If you’re a collaborator in an npm package, it’s your responsibility to protect yourself. Don’t let attackers to post a bad release on your behalf. Doing so is simple: <strong>Stay logged out</strong>.</p>

<p>Stick to publishing through your CI when a build on a the master branch succeeds, using <a href="https://github.com/semantic-release/semantic-release">semantic-release</a> or custom scripts. Here are the step-by-step instructions:</p>

<ol>
  <li><strong>Invalidate all current tokens</strong>. You can do so through your <a href="https://www.npmjs.com/settings/tokens">token page</a>, and should do the same for all collaborators.</li>
  <li><strong>Configure a new token on your CI</strong>. Remy wrote <a href="https://remysharp.com/2015/10/26/using-travis-with-private-npm-deps">detailed instructions</a> for Travis, but most CIs support hidden variables. Note that, once your token is configured, running <code class="highlighter-rouge">npm logout</code> will invalidate it. Delete your <code class="highlighter-rouge">~/.npmrc</code> file instead to return to a logged out state.</li>
  <li><strong>For dev versions, use git</strong>. If users need access to a temp package version (e.g. when troubleshooting), commit it to a branch and guide them to <a href="https://docs.npmjs.com/cli/install">install via a git commit</a>.</li>
</ol>

<h3 id="how-to-prevent-malicious-publishing-private">How to prevent malicious publishing (private)</h3>
<p>If you’re using a private package, being logged out isn’t an option, as you’ll need to be logged in to have access to your private packages. Fortunately, <em>npm</em> has recently released organizations, and support read-only users, letting you provide access without risking publishing.</p>

<p>Once more, here are the step-by-step instructions for doing so:</p>

<ol>
  <li>Create an <em>npm</em> <a href="https://docs.npmjs.com/orgs/what-are-orgs">organization</a>. If you only have one user, this will bump the minimum cost by $7/month, but is well worth it.</li>
  <li>Create a <a href="https://docs.npmjs.com/orgs/teams">team</a> with <a href="https://docs.npmjs.com/orgs/package-access">read-only access</a> to your packages or scope.</li>
  <li>Add users to that team as <a href="https://docs.npmjs.com/orgs/roles#member">members</a>. You can also reuse one read-only user.</li>
  <li>Have your team (and yourself) login as these read-only users</li>
</ol>

<p>These steps do not alleviate the need to be careful when installing random software on your computer, but at least they reduce the likelihood of you being a distribution vehicle for malicious code.</p>

<h3 id="how-to-protect-yourself-from-malicious-packages">How to protect yourself from malicious packages</h3>
<p>As a consumer of <em>npm</em> packages, you can’t truly avoid this risk (note this is true for other package managers as well). The only way to mitigate the risk is by controlling which packages you install.</p>

<p>If you’re developing locally, consider using <a href="https://docs.npmjs.com/cli/shrinkwrap">npm shrinkwrap</a> to contain the package versions you use. <em>shrinkwrap</em> freezes the versions you use, not taking new versions unless explicitly asked, and so gives you the opportunity to and manually scrutinize every update. This is cumbersome, but will make you safer.</p>

<p>If you’re an organization, consider using <a href="https://www.npmjs.com/on-site">npm On-Site</a> and <a href="https://docs.npmjs.com/enterprise/mirroring#whitelisting">whitelisting</a> the packages you allow. Again, this will be a bit of a hassle, but will reduce the risk of malicious packages making their way in. To be truly safe here, make sure you set <em>Read through cache</em> to <em>Off</em>.</p>

<p>Both of these solutions will slow down your intake of new versions, and so may leave you exposed when vulnerabilities are disclosed on older versions of these packages. If you choose them, I’d encourage you to stay on top of known vulnerabilities in your <em>npm</em> dependencies, for instance by <a href="https://snyk.io/docs/using-snyk/">using Snyk</a>.</p>

<h2 id="cant-npm-just-fix-this">Can’t npm just fix this?</h2>
<p>Wouldn’t it be awesome if <em>npm</em> can simply throw in a feature or two and make this entire risk go away?</p>

<p>Unfortunately, they can’t. This risk is a natural byproduct of using open source packages. It means we use many small pieces of code written by many people. While awesome for productivity, such crowdsourcing means we are substantially dependent on the authors of those packages and their intents.</p>

<p>It’s also important to note these behaviors aren’t unique to npm. Most other package managers allow silent publish, and practically all of them allow custom package code to run on install. The only reason this risk is greater with <em>npm</em> is the larger number of indirect dependencies used, which make it all but impossible to track them all manually.</p>

<p>That said, there are a few things I would love to see <em>npm</em> team do to help  the following:</p>

<ol>
  <li>Send an email to all collaborators when a new package version is released. This should be the default behavior, optionally allowing users to opt-out later.</li>
  <li>Make <code class="highlighter-rouge">npm publish</code> require credentials, unless used with an explicitly generated automation token. This will make the default behavior (using <code class="highlighter-rouge">npm login</code>) safer. This is likely a breaking change, and so I wouldn’t expect it to happen before npm 4.</li>
  <li>If possible, contain the actions installations scripts can perform during installation. This won’t eliminate this risk, but it’ll make it that much harder, and so that much less likely to occur.</li>
</ol>