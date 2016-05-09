---
ID: 11
post_title: >
  Using Node.js Event Loop for Timing
  Attacks
author: wpengine
post_date: 2016-02-16 00:00:00
post_excerpt: ""
layout: post
permalink: http://snyk.wpengine.com/?p=11
published: true
---
<p>A little over 3 years ago, <a href="https://www.linkedin.com/in/yuval-ofir-b4809a76">a</a> <a href="https://github.com/lipner">few</a> <a href="https://github.com/barkatz">friends</a> <a href="https://twitter.com/talzeltzer">and</a> I started a group called <em>pasten</em> to participate in the <a href="https://www.ccc.de/en/">Chaos Computer Club’s</a> <a href="https://events.ccc.de/tag/ctf/"><em>Capture The Flag</em></a> (CTF) competition. It is a jeopardy style CTF, where the participating teams need to solve security related challenges in various categories such as exploitation, reverse engineering, web, forensic & crypto. CTF is fun and educational, and I definitely recommend participating in it. It’s even more fun when you win, like <a href="https://archive.aachen.ccc.de/32c3ctf.ccc.ac/">we did this year</a>!</p>

<p>This year’s competition included, amongst other challenges, a Node.js application which was susceptible to an interesting timing attack, leveraging the Node.js-specific Event Loop. This post walks through the challenge, explaining the risk while demonstrating how an attacker may unravel and exploit such a vulnerability. Hopefully it will also help you avoid similar vulnerabilities in your own code.</p>

<h3 id="timing-attacks">Timing attacks</h3>

<p>Before we dig into the challenge, let’s explain what a <em>timing attack</em> is.</p>

<p>Imagine a service where, when you provide the wrong password to an existing email, the service would respond with “The fifth letter of your password is wrong, please try again”. Seems absurd, right? Giving such information lets an attacker brute-force the password one character at a time, making it trivial to break in. However, this is exactly what happens when we use naive string comparison when verifying passwords or authentication tokens.</p>

<p>Native string comparison, including JavaScript’s <code class="highlighter-rouge">==</code> operator, typically works by iterating the two (equal length) strings, comparing one character at a time, and stopping when a character differs. So, if I compared <code class="highlighter-rouge">foo</code> to <code class="highlighter-rouge">bar</code>, the loop would iterate once, while comparing <code class="highlighter-rouge">foo</code> to <code class="highlighter-rouge">fox</code> would compare 3 chars, taking longer to complete.</p>

<p>Sample vulnerable authentication function:</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4</pre></td><td class="code"><pre><span style="color: #66d9ef">function</span> <span style="color: #ffffff">isAuthenticated</span><span style="color: #ffffff">(</span><span style="color: #ffffff">user</span><span style="color: #ffffff">,</span> <span style="color: #ffffff">token</span><span style="color: #ffffff">)</span> <span style="color: #ffffff">{</span>
  <span style="color: #66d9ef">var</span> <span style="color: #ffffff">correctToken</span> <span style="color: #f92672">=</span> <span style="color: #ffffff">FetchUserTokenFromDB</span><span style="color: #ffffff">(</span><span style="color: #ffffff">user</span><span style="color: #ffffff">);</span>
  <span style="color: #f92672">return</span> <span style="color: #ffffff">token</span> <span style="color: #f92672">===</span> <span style="color: #ffffff">correctToken</span><span style="color: #ffffff">;</span>
<span style="color: #ffffff">}</span>
</pre></td></tr></tbody></table>
</div>
</div>

<p>A single character comparison is pretty fast to do, but it’s slow enough. <a href="https://www.cs.rice.edu/~dwallach/pub/crosby-timing2009.pdf">Research shows</a> an attacker can measure events with 15-100µs accuracy across the internet, and a 100ns accuracy over a local network. Attackers can use these techniques, making these small delays quite similar to telling the user which character was wrong.</p>

<p>This type of attack is called a timing attack, and it can be performed each time input impacts processing time. To prevent it, we need to make the string comparison take the same amount of time regardless of the entered password, for instance by <code class="highlighter-rouge">xoring</code> the two passwords, and seeing if the result was zero.</p>

<p>Not vulnerable to timing attacks</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5</pre></td><td class="code"><pre><span style="color: #66d9ef">var</span> <span style="color: #ffffff">mismatch</span> <span style="color: #f92672">=</span> <span style="color: #ae81ff">0</span><span style="color: #ffffff">;</span>
<span style="color: #f92672">for</span> <span style="color: #ffffff">(</span><span style="color: #66d9ef">var</span> <span style="color: #ffffff">i</span> <span style="color: #f92672">=</span> <span style="color: #ae81ff">0</span><span style="color: #ffffff">;</span> <span style="color: #ffffff">i</span> <span style="color: #f92672"><</span> <span style="color: #ffffff">a</span><span style="color: #ffffff">.</span><span style="color: #ffffff">length</span><span style="color: #ffffff">;</span> <span style="color: #f92672">++</span><span style="color: #ffffff">i</span><span style="color: #ffffff">)</span> <span style="color: #ffffff">{</span>
  <span style="color: #ffffff">mismatch</span> <span style="color: #f92672">|=</span> <span style="color: #ffffff">(</span><span style="color: #ffffff">a</span><span style="color: #ffffff">.</span><span style="color: #ffffff">charCodeAt</span><span style="color: #ffffff">(</span><span style="color: #ffffff">i</span><span style="color: #ffffff">)</span> <span style="color: #f92672">^</span> <span style="color: #ffffff">b</span><span style="color: #ffffff">.</span><span style="color: #ffffff">charCodeAt</span><span style="color: #ffffff">(</span><span style="color: #ffffff">i</span><span style="color: #ffffff">));</span>
<span style="color: #ffffff">}</span>
<span style="color: #f92672">return</span> <span style="color: #ffffff">mismatch</span><span style="color: #ffffff">;</span>
</pre></td></tr></tbody></table>
</div>
</div>

<h2 id="the-challenge---sequence-hunt">The challenge - Sequence Hunt</h2>
<p>Armed with this information, let’s dig into the challenge. It started with a link to a page holding the following:</p>

<blockquote>
  <p>Sequence Hunt</p>

  <p>Welcome to my integer sequence hunt. You have to compute five numbers between 0 and 100 implementing
the algorithms below. When you think you have a correct solution, send them as GET parameters <a href="javascript:alert('This link would take you to /check')">here</a>.
You can also get some info about you here.
I expect a lot of load, so I built this around async <a href="javascript:alert('This link would take you to /info')">IOLoops</a>.
Have Fun!</p>

  <p>[EDIT]
Someone sent me <a href="https://en.wikipedia.org/wiki/Timing_attack">this</a>. Since each check takes considerable time, the server now waits until
three seconds have passed since the request came in to prevent these attacks.
[/EDIT]</p>

  <p>Algorithms
TODO publish algorithms</p>
</blockquote>

<p>The text <em>here</em> points to <code class="highlighter-rouge">/check</code>, where we need to send the correct parameters to get the flag, and <em>IOLoops</em> links to <code class="highlighter-rouge">/info</code>, which shows the following:</p>

<blockquote>
  <p>you are 37.120.106.140
request count on your IOLoop: 1</p>
</blockquote>

<p>What do we know so far?</p>

<ol>
  <li>There’s an unknown algorithm that takes in 5 numbers, ranging from 0 to 100</li>
  <li>To get the flag we need to provide the correct numbers.</li>
  <li>The algorithm’s execution time probably depends on the input, potentially allowing a timing attack.</li>
  <li>The server <em>always</em> waits for at least 3 seconds before sending back the response, to hinder timing attacks.</li>
</ol>

<p>Now we need to figure out how to capture the flag. Since the challenge was titled <code class="highlighter-rouge">Sequence Hunt</code>, I’ll use <code class="highlighter-rouge">sequencehunt.com</code> as a placeholder domain name in my examples, but do not send any attacks to this (unrelated) domain! You can run the <a href="https://gist.github.com/grnd/d5b22e8bcd9942fb7c7b">vulnerable sample application I wrote</a>. It should behave similarly to the CTF one.</p>

<h2 id="exploring">Exploring</h2>
<p>A quick confirmation shows it indeed takes about 3 seconds for a <code class="highlighter-rouge">check</code> request to return, regardless of the values we provide. This means brute-forcing all the combinations is not an option. It will require trying 100^5 (10 billion) combinations, which will take thousands of years to complete…</p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5
6
7
8
9</pre></td><td class="code"><pre><span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/check?val0=1&val1=1&val2=1&val3=1&val4=1"</span>
you are 37.120.106.140<br />At least one value is wrong!

<span style="color: #555555">$ </span><span style="color: #f6aa11">time </span>curl <span style="color: #e6db74">"http://sequencehunt.com/check?val0=1&val1=1&val2=1&val3=1&val4=1"</span>
you are 37.120.106.140<br />At least one value is wrong!
0.00s user
0.01s system
<span style="color: #555555">0% </span>cpu
3.174 total
</pre></td></tr></tbody></table>
</div>
</div>

<p>Playing with the <code class="highlighter-rouge">/info</code> page we notice that each <code class="highlighter-rouge">/check</code> request increases the IOLoop count, and so does the <code class="highlighter-rouge">/info</code> request itself.</p>

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
11
12
13
14</pre></td><td class="code"><pre><span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/info"</span>
you are 37.120.106.140<br />request count on your IOLoop: 1

<span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/info"</span>
you are 37.120.106.140<br />request count on your IOLoop: 2

<span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/info"</span>
you are 37.120.106.140<br />request count on your IOLoop: 3

<span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/check?val0=1&val1=1&val2=1&val3=1&val4=1"</span>
you are 37.120.106.140<br />At least one value is wrong!

<span style="color: #555555">$ </span>curl <span style="color: #e6db74">"http://sequencehunt.com/info"</span>
you are 37.120.106.140<br />request count on your IOLoop: 5
</pre></td></tr></tbody></table>
</div>
</div>

<p>Sending <code class="highlighter-rouge">/check</code> requests with various random numbers does not give us any statistically significant differences in timing. The algorithm apparently takes less than 3 seconds to run, and so all requests simply return in the specified minimum of 3 seconds.</p>

<p>However, we noticed that sending a request to <code class="highlighter-rouge">/info</code> while there’s an outstanding <code class="highlighter-rouge">/check</code> request takes considerably longer to respond - giving us a timing related hook to explore. To understand the next step, let’s take a moment to talk about Node’s event loop.</p>

<h2 id="nodejs-event-loop">Node.js event loop</h2>

<p>Designed with scalability in mind, Node.js (and JavaScript in general) is an asynchronous event driven framework. When a piece of code wants to call a potentially blocking action, such as opening a file or writing to the network, it registers a callback function, triggers the relevant action, and terminates. When the action completes (e.g. the file was opened), the callback event is called by the event loop.</p>

<p>The event-driven model is extremely scalable, as threads never sit and “wait”, but it introduces a problem when a function takes a long time to complete. Since the Node.js server runs only one thread per core (by default), such a long running function would take up the entire core, while others wait in the queue for their turn. In the browser, this is the reason for the “script taking to long to run” error message you may have encountered. On the server, it will simply make requests hang.</p>

<p>To better understand Node JS event loop, see the <a href="https://www.youtube.com/watch?v=8aGhZQkoFbQ">excellent talk</a> <em>Philip Roberts</em> gave at JSConf, or read <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/EventLoop">one</a> <a href="https://nodesource.com/blog/understanding-the-nodejs-event-loop/">of</a> <a href="http://blog.carbonfive.com/2013/10/27/the-javascript-event-loop-explained/">these</a> <a href="http://altitudelabs.com/blog/what-is-the-javascript-event-loop/">articles</a>.</p>

<h2 id="breaking-the-puzzle">Breaking the puzzle</h2>

<p>In our case, the event loop provided us with the timing information we needed. As long as <code class="highlighter-rouge">/check</code> was running, the event loop didn’t regain control, making the <code class="highlighter-rouge">/info</code> request hang. In contract, the <code class="highlighter-rouge">setTimeout</code> function simply registers a scheduled event and yields control, letting <code class="highlighter-rouge">/info</code> requests to be processed quickly. I put the code for a simple server showing this effect on <a href="https://gist.github.com/grnd/d5b22e8bcd9942fb7c7b">GitHub</a> if you want to see the effect locally.</p>

<p>Now that we had a way to get timing info (also known as having a side-channel), all we needed to do is to send a <code class="highlighter-rouge">/check</code> request and, without waiting for the response, send a <code class="highlighter-rouge">/info</code> request, measuring the time it takes for it to return.</p>

<p class="u--centered">
<img src="http://res.cloudinary.com/snyk/image/upload/v1455214448/ccc-timing-attack-sequence-diagram.png" alt="" />
</p>

<p>To automate the search process, and cancel out network jitter, the next step was to write a python script that, given a set of inputs:</p>

<ul>
  <li>Opens <code class="highlighter-rouge">n</code> threads</li>
  <li>Sends a <code class="highlighter-rouge">check</code> request and measures the time it takes for a parallel <code class="highlighter-rouge">/info</code> request to return</li>
  <li>Averages the times and returns the result</li>
</ul>

<p>I used n=5, and it proved to be enough. It might be higher depending on the quality of the internet connection and the algorithm’s timing properties. Here are the results from the first run - note the significant difference with the value <code class="highlighter-rouge">4</code></p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4
5
6
7
8
9</pre></td><td class="code"><pre>[0, 0, 0, 0, 0]: 0.4933500289916992
[1, 1, 1, 1, 1]: 0.3603970050811768
[2, 2, 2, 2, 2]: 0.4297104358673095
[3, 3, 3, 3, 3]: 0.4705570697784423
[4, 4, 4, 4, 4]: 0.9154952526092529 <<<<<<<<<
[5, 5, 5, 5, 5]: 0.4637355804443359
[6, 6, 6, 6, 6]: 0.3557830333709716
[7, 7, 7, 7, 7]: 0.4418128013610840
[8, 8, 8, 8, 8]: 0.4297045707702637
</pre></td></tr></tbody></table>
</div>
</div>

<p>Since the difference in timing appeared only when 4 was in the first position, the next step was to enumerate over the rest of the parameters. Second digit was found to be <code class="highlighter-rouge">12</code></p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4</pre></td><td class="code"><pre>[4, 10, 10, 10, 10]: 0.840841388702392
[4, 11, 11, 11, 11]: 0.859339499473571
[4, 12, 12, 12, 12]: 1.228700304031372 <<<<<<<
[4, 13, 13, 13, 13]: 0.907831811904907
</pre></td></tr></tbody></table>
</div>
</div>

<p>And so on, until we got to all the correct parameters <code class="highlighter-rouge">[4, 12, 77, 98, 35]</code></p>

<div class="highlighter-rouge"><div class="syntax"><table style="border-spacing: 0"><tbody><tr><td class="gutter gl" style="text-align: right"><pre class="lineno">1
2
3
4</pre></td><td class="code"><pre><span style="color: #555555">$ </span>curl <span style="color: #e6db74">'http://sequencehunt.com/check?val0=4&val1=12&val2=77&val3=98&val4=35'</span>
you are 37.120.106.140
Congratulations, here is your prize:
32C3_round_and_round_and_round_IT_goes
</pre></td></tr></tbody></table>
</div>
</div>

<p>Flag captured.</p>

<h2 id="summary">Summary</h2>

<p>Timing attacks are a real threat, and affect both small and big players (like <a href="http://rdist.root.org/2009/05/28/timing-attack-in-google-keyczar-library/">Google</a>). Even small differences in execution time can be gleaned with a fairly small sample set (in internet scale), and used by attackers to glean information and refine brute-forcing. In Node.js specifically, the event loop can be used as another signal for timing attacks.</p>

<p>A few tips to protect your own application from such flaws:</p>

<ul>
  <li>When implementing authentication checks, keep the authentication process running in a constant amount of time. You can use <a href="https://www.npmjs.com/package/secure-compare">secure-compare</a> or <a href="https://www.npmjs.com/package/scmp">other</a> <a href="https://www.npmjs.com/package/buffer-equal-constant-time">packages</a> to achieve such constant time comparisons.</li>
  <li>Be sure to check they don’t have <a href="https://snyk.io/vuln/npm:secure-compare:20151024">known vulnerabilities</a></li>
  <li>Test for timing vulnerabilities <a href="https://github.com/dmayer/time_trial">with</a> <a href="https://github.com/ecbftw/nanown">tools</a>.</li>
</ul>

<p>Lastly, if you want to show case your own skills and take on these challenges, try participating in CCC’s Capture The Flag yourself next year!</p>