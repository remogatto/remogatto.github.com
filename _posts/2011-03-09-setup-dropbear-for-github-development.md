---
layout: post
title: "Setup Dropbear for GitHub development on Angstrom"
---
# Preamble

This is just a note to myself rather then a comprehensive blog
post. However, I thought it could be helpful for someone else, so here
we are!

# The problem

[Dropbear](http://matt.ucc.asn.au/dropbear/dropbear.html) is a small
SSH2 server and client. Its small memory footprint is suitable for
embedded devices like the [BeagleBoard](http://beagleboard.org/) and
the [OpenPandora](http://www.openpandora.org/). In fact, dropbear is
installed by default in the [OpenPandora
OS](http://git.openpandora.org/cgi-bin/gitweb.cgi) (an Angstrom-derived
OS) replacing the standard OpenSSH distribution.

However, when I decided to do my first coding experiment with the
Pandora, I had some issue interacting with github. In particular:

1. I didn't readily understand how to generate a pair of proper
public/private keys understandable by GitHub

2. After I got my keys, Github refused to authenticate me using the
uploaded public key

# The solution

Well, the first point was trivial. To generate the keys we need to
install the <tt>openssh-keygen</tt>:

<pre class="terminal">
sudo opkg install openssh-keygen
</pre>

At this point, we can proceed as usual, generating our keys:

<pre class="terminal">
ssh-keygen -t rsa
</pre>

and then uploading the content of <tt>id_rsa.pub</tt> on our github account.

I solved the second issue thanks to this [blog
post](http://tumblelog.jauderho.com/post/151678345/using-dropbear-with-git). The
problem resides in the fact that the dropbear client doesn't
automatically look in <tt>~/.ssh/</tt> for a public key. To do that,
we need to create a wrapper script in <tt>/usr/local/bin</tt>. I named
the script <tt>wrap_ssh.sh</tt> and I put in it the following content:

{% highlight bash %}
#!/bin/sh
ssh -i ~/.ssh/id_rsa $*
{% endhighlight %}

Don't forget to make the script executable:

<pre class="terminal">
sudo chmod +x /usr/local/bin/wrap_ssh.sh
</pre>

At this point, we need to tell <tt>git</tt> to use
<tt>wrap_ssh.sh</tt> instead of the plain <tt>ssh</tt> command. This
action is accomplished setting the <tt>GIT_SSH</tt> environment
variable:

<pre class="terminal">
export GIT_SSH=/usr/local/bin/wrap_ssh.sh
</pre>

To make this change permanent, you should add the line above in your
<tt>~/.bashrc</tt> file.

Now, you should be able to interact with your github repository from
your device.



