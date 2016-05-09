---
ID: 12
post_title: >
  Keeping your Open Source credentials
  closed
author: wpengine
post_date: 2015-12-14 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=12
published: true
---
<p>Earlier this month, a researcher named <a href="https://github.com/ChALkeR/">ChALkeR</a> shared <a href="https://github.com/ChALkeR/notes/blob/master/Do-not-underestimate-credentials-leaks.md">his research</a> on leaked credentials in npm packages. The findings showed credentials, such as npm and GitHub tokens and passwords, are frequently included in published npm packages or GitHub repositories. In fact, 13 of the top 15 npm packages depend on packages that leaked credentials, thus exposing their users.</p>

<p>ChALKeR’s post is a good read and the first npm-specific leaked credentials analysis I know of, but the problem itself isn’t new. GitHub search made it easy to <a href="https://github.com/search?utf8=%E2%9C%93&q=%22begin+rsa+private+key%22&type=Code&ref=searchresults">find private keys</a> nearly three years ago (<a href="http://news.softpedia.com/news/GitHub-Forced-to-Disable-Search-After-Exposing-Private-SSH-Keys-324200.shtml">reportedly</a> exposing keys from the Chrome team), and Google can expose <a href="http://www.hackersforcharity.org/ghdb/">all sorts of private info</a>. As code gets more discoverable, mistakes like this would be more easily found, which is all the more reason for you to do something about it.</p>

<h2 id="why-you-should-care-about-leaked-credentials">Why you should care about leaked credentials</h2>

<p>Leaking credentials means exposing secrets that provide access to different accounts.</p>

<p>The most common types of leaked credentials are passwords, API keys and SSH private keys. These keys may be sufficient for an automated tool - be it your own or an attacker’s - to perform an action, for instance publish a new version of a package or delete code. The actual range of actions each key is allowed to do varies by key and system.</p>

<p>If you’re consuming open source packages, the credentials that should worry you most are those that allow publishing new code, such as an <a href="http://blog.npmjs.org/post/118393368555/deploying-with-npm-private-modules">npm token</a> or a GitHub <a href="https://developer.github.com/guides/managing-deploy-keys/">deploy key</a>. An attacker with access to those can easily publish a new “fix” version with malicious code in it, which you’ll often implicitly install.</p>

<p>If you’re hosting your open source project somewhere, you should also beware leaking any information about your runtime systems. This goes beyond npm, as you may expose your <a href="https://securosis.com/blog/my-500-cloud-security-screwup">AWS</a> <a href="http://vertis.io/2013/12/17/an-update-on-my-aws-bill.html">keys</a>, <a href="http://www.computerworld.com/article/2941156/network-security/cisco-warns-of-default-ssh-keys-shipped-in-three-products.html">SSH private keys</a> or passwords for different systems. Any one of these keys could give attackers access to your system, which can be used for anything from stealing your IP and user data to simply getting free compute (leaving you <a href="http://wptavern.com/ryan-hellyers-aws-nightmare-leaked-access-keys-result-in-a-6000-bill-overnight">with the bill</a>).</p>

<h2 id="as-an-open-source-developer-what-should-i-do">As an open source developer, what should I do?</h2>
<p>Beyond being aware of the risk, you should establish a few best practices as part of your flow, and introduce multiple layers of defense to help prevent mistakes. Here are some specific suggestions you could consider:</p>

<h3 id="avoid-blanket-git-add--commands">Avoid blanket <code class="highlighter-rouge">git add *</code> commands.</h3>
<p>Using wildcards can easily capture local files not truly intended to be shared. This is especially important when adding entire subfolders (e.g. <code class="highlighter-rouge">git add dirname</code>), as doing so will include hidden (<code class="highlighter-rouge">dot</code>) files too.</p>

<p>Instead of wildcards, name each file you commit, or use <code class="highlighter-rouge">git add -p</code> to review each change you add.</p>

<h3 id="name-sensitive-files-in-gitignore--npmignore">Name sensitive files in .gitignore & .npmignore</h3>
<p>Both git and npm support a local file listing exclusions from packaging and commits, which you can use as a safety measure against accidental inclusion of sensitive files. If you don’t specify ignores, npm will still <a href="https://docs.npmjs.com/misc/developers#keeping-files-out-of-your-package">ignore certain files</a>, but git will include them all. Defaults aside, neither tool knows your application as well as you do, so its best to edit these files yourself.</p>

<p>Sample files to be wary of are:</p>

<ul>
  <li>CI configuration files (e.g. <code class="highlighter-rouge">.travis.yml</code>, <code class="highlighter-rouge">circle.yml</code>)</li>
  <li>Dockerfile and docker-compose.yml</li>
  <li>Build output files (e.g. <code class="highlighter-rouge">gypi</code>)</li>
  <li>Custom deploy scripts (e.g. anything under /deploy/ or /scripts/).</li>
</ul>

<p>ChALKeR <a href="https://github.com/ChALkeR/notes/blob/master/Do-not-underestimate-credentials-leaks.md#common-sources-of-leaks">lists a few other examples</a> in his post, and you can use GitHub’s <a href="https://github.com/github/gitignore">sample <code class="highlighter-rouge">.gitignore</code> files</a> for other inspiration.</p>

<p>Note that npm’s <a href="https://docs.npmjs.com/misc/developers#keeping-files-out-of-your-package">broader default ignores</a> were only introduced in <a href="https://github.com/npm/npm/releases/tag/v2.14.1">npm@2.14.1</a>, so be sure to upgrade to that version or newer.</p>

<h3 id="encrypt-or-use-environment-vars-when-publishing-from-ci">Encrypt or use environment vars when publishing from CI</h3>
<p>If you publish to npm through your CI, you’ll need the CI to have access to your npm token, and its easy to mistakenly add that token to the CI config files and expose it.</p>

<p>When possible, use an environment variable to hold that token instead, keeping it out of source control and obfuscating it in the output (at least on <a href="https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings">Travis</a> & <a href="https://circleci.com/docs/environment-variables#setting-environment-variables-for-all-commands-without-adding-them-to-git">Circle</a>). If for some reason you have to include a key in your source control (for instance, if you want to <a href="http://awestruct.org/auto-deploy-to-github-pages/">commit content back into GitHub</a>), make sure you <a href="https://docs.travis-ci.com/user/encryption-keys/">encrypt</a> it well.</p>

<h3 id="git-secrets-git-hook-prevents-committing-in-credentials">git-secrets: git hook prevents committing in credentials</h3>
<p><a href="https://github.com/mtdowling">Michael Dowling</a> at AWS Labs recently released a useful tool called <a href="https://github.com/awslabs/git-secrets">git-secrets</a>. The tool hooks onto <code class="highlighter-rouge">git commit</code>, and breaks the commit if it includes patterns that appear to be credentials. This is a good content-focused safety net, complementing the previously suggested filename based protection.</p>

<p>It’s important to note you need to catch these <em>before</em> you commit (or rather, before you push), like this tool does. For open source projects, testing as part of the CI or pull request process is too late - the credentials have already leaked.</p>

<h3 id="invalidate-leaked-credentials">Invalidate leaked credentials</h3>
<p>Lastly, if you learn that you’ve already leaked some key or password, make sure you quickly invalidate those tokens.</p>

<p>Removing the token from the latest release or branch will not be enough. Even if you deleted historical references, copies of your key will be around even if you now remove them from GitHub and npm - the internet never forgets. It’s also a good idea to do some house inspection and see if anybody used the key in the meantime to publish bad code or access your systems.</p>

<h2 id="as-an-open-source-consumer-what-should-i-do">As an open source consumer, what should I do?</h2>

<p>There isn’t much you can do as a consumer of open source packages. By using a package you’ve explicitly put your trust in it and implicitly trust the dependencies it pulls along. Security blunders by those dependencies can compromise your own systems, be they leaked credentials or vulnerabilities (you can use <a href="https://snyk.io">Snyk</a> to address the latter). <a href="https://github.com/ChALkeR/notes/blob/master/Do-not-underestimate-credentials-leaks.md#qa">According to ChALKeR</a>, the credentials he discovered have now been invalidated by GitHub and npm respectively, and invalidated credentials do not pose a security risk anymore (assuming they haven’t been exploited in the interim).</p>

<p>That said, if you want to be proactive you could consider auditing your own dependencies and see if they share information you think they shouldn’t. Some bash foo can help you find the relevant dot files in your dependencies, after which you can make your own decision about whether or not they contain private information.</p>

<p>Here’s a bash Mac/Linux command to get you started. It finds some of these files, drops duplicates and outputs them along with the filename:</p>

<div class="highlight">
<pre><code class="language-bash">find . \( -name '*.gypi' -o -name '.npmrc' -o -name '.travis.yml' \) | xargs md5 -r | sort | awk '{print $2" "$1}' | uniq -f1 | sort | awk '{print "echo "$1";cat "$1}'</code></pre>
</div>

<h2 id="summary">Summary</h2>

<p>Developing in the open is great, but not everything is meant to be shared. Leaking credentials puts your consumers at risk, and in turn their users.</p>

<p>Separate your credentials from your code, and make everybody aware and alert to this risk. Look for leaks in code reviews, and add it as a check to your standard practices. Beyond process, set up the safety nets described above to catch cases of human error. Lastly, if you learn you’ve leaked, handle it well - invalidate the tokens, alert your users, and inspect your code for malicious traces.</p>