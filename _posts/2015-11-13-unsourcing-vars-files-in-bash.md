---
title:  "Unsourcing Vars Files in BASH"
date:   2015-11-13
description: Use BASH to unset variables in a given vars file
---

## The Problem

BASH has a nifty builtin for loading variables into the current environment, however, there really doesn't exist much in the way of clearing all variables set in a file. To paint a picture, take the following vars file (secrets.env):

{% highlight bash %}
mysecretvar=p@ssword
anothersecretvar=supersecret
{% endhighlight %}

We can load this file into our BASH environment with the use of a BASH builtin (`source` or `.`):

{% highlight bash %}
$ source secrets.env
{% endhighlight %}

Now we have access to those variables in our environment:

{% highlight bash %}
$ echo $mysecretvar
p@ssword
$ echo $anothersecretvar
supersecret
{% endhighlight %}

These variables can be `unset`, or erased, using a BASH builtin as well:

{% highlight bash %}
$ unset $mysecretvar
$ echo $mysecretvar

{% endhighlight %}

This is a trivial and common use case. Typically I find myself loading in secrets to the BASH environment and then using those secrets to launch an app, script or container. When I'm done executing, I don't want those variables still present in my environment.

Our `secrets.env` file is fairly straightforward and small. However, what if it was much larger and contained more than just two variables being set? Or what if there was conditional logic in that file?

We would have to keep track of which variables got set, then work to unset them one by one. BASH doesn't come with any aid here to "unsource" a vars file, that is, unset all the variables it would normally set.

## Unsource

I created a small little script to help with just that. It uses a few BASH tricks (namely `env -i`) to compare a blank environment with an environment after a file has been sourced. Then it loops through the variables that were added, and uses `unset` to clear them of any values.

{% gist nextrevision/3f1583eea12017f0838f unsource.sh %}

Using our previous example with `secrets.env`, lets see the script in action:

{% highlight bash %}
$ echo $mysecretvar
p@ssword
$ source unsource.sh secrets.env
Cleared: mysecretvar
Cleared: anothersecretvar
$ echo $mysecretvar

{% endhighlight %}

So this may seem a little wonky, but what we did was `source` the `unsource.sh` script. This tells BASH to execute all commands in the `unsource.sh` file in the current environment, instead of a child environment. By doing this, we can execute `unset` commands just as though we were in the parent environment. The other thing to note is that `source` is taking two arguments, the first is the script/file (in our case `unsource.sh`) and the second are any arguments to that script/file which will be used as positional arguments. Because of this, we are able to pass `secrets.env` as a positional argument to `unsource.sh` and everything will work as expected.

## Aliasing FTW

Let's optimize our process a little bit. Typing in `source unsource.sh <varsfile>` is a bit much. We can easily create an alias to do this for us:

{% highlight bash %}
$ alias unsource="source /path/to/unsource.sh"
{% endhighlight %}

Now we can easily just type in `unsource`. Take it one step further and add it to your `~/.bash_profile` to have it persist between sessions.

## Known limitations

1. Loading current environment variables/functions in to the sourced file at comparison time. Really only comes into play when referencing functions from other files or needing access to the parent's environment vars.
2. Does not unsource BASH functions.