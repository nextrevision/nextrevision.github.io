---
title:  "Testdummy - Another Bash Script Testing Framework"
date:   2015-10-20
description: Testdummy is a yet another way to test your bash scripts
---

## Beating the Dead Horse

There are many reasons why you should be running unit tests, and if you are unfamiliar with the concept, [here](http://martinfowler.com/bliki/UnitTest.html) is a decent summary on some of the history and concepts.

For most programming languages, frameworks for testing your code exist, and the same is true for bash scripts ([bats](https://github.com/sstephenson/bats), [shunit2](https://code.google.com/p/shunit2/), [assert.sh](https://github.com/lehmannro/assert.sh)). Testdummy builds on top of these by wrapping common testing patterns into easy to use components.

## Another Take

I wanted to create a library that made reading and writing tests easy to do. Like any other framework, you have to learn how to use the components of the framework. The goal in writing `testdummy` was to make getting started very easy by encapsulating the logic of common testing patterns into helper methods.

Most of the time, we have a function, or an entire script that we want to execute, check the return code and output. This is the basic pattern you'll find in `testdummy`. For example, if we want to test that running `/bin/false` has an exit code of `1`, a manual test may look like:

{% highlight bash %}
test_bin_false() {
  /bin/false
  [ $? -eq 1 ]
}
{% endhighlight %}

Testdummy has an assertion helper that is a wrapper around that very thing:

{% highlight bash %}
test_bin_false() {
  cmd '/bin/false'
  assert_rc 1
}
{% endhighlight %}

It's worth noting that `testdummy` can execute both tests with the same result, but the latter just makes use of the helpers exposed in `testdummy`. So long as the testing functions starts with `test_`, `do_`, or `it_`, it will be picked up by `testdummy` and executed as a unit test. The contents of the function (the actual tests) can be any valid bash, you can choose to use the builtin helpers of `cmd`, `assert_rc`, and `assert_content` or your own logic.

This is a very trivial test case, and most of the time we'll be testing other things like command output. Let's try another example where we want to make sure the result of `curl http://google.com` results in a `301 Moved` and a link to `<A HREF="http://www.google.com/">here</A>` with a return code of `0`.

{% highlight bash %}
test_curl_google() {
  curl -o output.txt http://google.com
  [ $? -eq 0 ]
  grep '301 Moved' output.txt
  grep 'http://www.google.com' output.txt
  rm output.txt
}
{% endhighlight %}

Using `testdummy` helpers, this would look like:

{% highlight bash %}
test_curl_google() {
  cmd 'curl http://google.com'
  assert_rc 0
  assert_content '301 Moved'
  assert_content 'http://www.google.com'
}
{% endhighlight %}

## A Trivial Example

Let's take a look at our super awesome script:

{% highlight bash %}
#!/bin/bash

\_usage() {
  echo "usage: $0 [string]"
  echo
  echo "outputs the specified string"
  echo
}

\_error() {
  echo "ERROR: ${1}"
  exit 1
}

\_main() {
  [ -z "${1}" ] && \_error "must specify a string"
  echo "$@"
  return 0
}

# allow for sourcing
if [[ $(basename ${0//-/}) == "myscript.sh" ]]; then
  \_main "$@"
fi
{% endhighlight %}
