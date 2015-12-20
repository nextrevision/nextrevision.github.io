---
title:  "Testdummy - Another Bash Script Testing Framework"
date:   2015-12-19
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

Let's take a look at our script (`myscript.sh`):

{% highlight bash %}
#!/bin/bash

# print out usage instructions
_usage() {
  echo "usage: $0 [string]"
  echo
  echo "outputs the specified string"
  echo
}

# print an error message and exit with return code 1
_error() {
  echo "ERROR: ${1}"
  exit 1
}

# primary entrypoint
_main() {
  # if no arguments are passed, error and print the usage
  [ -z "${1}" ] && { _usage; _error "must specify a string"; }
  # if the first argument is '-h', print usage and exit
  [[ "${1}" == "-h" ]] && { _usage; exit 0; }
  # print all arguments to stdout
  echo "$@"
  return 0
}

# allow for sourcing
if [[ $(basename ${0//-/}) == "myscript.sh" ]]; then
  _main "$@"
fi
{% endhighlight %}

Let's take a look at what a test counterpart might look like (`myscript_test.sh`):

{% highlight bash %}
source myscript.sh

describe "test functions"
test_usage_function() {
  cmd "_usage"
  assert_content "^usage: "
  assert_rc 0
}
test_error_function() {
  cmd "_error 'foo'"
  assert_content '^ERROR: foo$'
  assert_rc 1
}
test_main_function() {
  cmd "_main 'foo'"
  assert_content '^foo$'
  assert_rc 0
}
test_main_function_no_args() {
  cmd "_main"
  assert_content '^ERROR: must specify a string$'
  assert_rc 1
}
test_main_function_help_args() {
  cmd "_main -h"
  assert_content '^usage'
  assert_rc 0
}

describe "test script args"
test_empty_arg() {
  cmd "./myscript.sh"
  assert_content '^ERROR: must specify a string$'
  assert_rc 1
}
test_help_arg() {
  cmd "./myscript.sh -h"
  assert_content '^usage:'
  assert_rc 0
}
test_single_arg() {
  cmd "./myscript.sh foo"
  assert_content '^foo$'
  assert_rc 0
}
test_multiple_args() {
  cmd "./myscript.sh foo bar"
  assert_content '^foo bar$'
  assert_rc 0
}
{% endhighlight %}

Let's break down these tests a little further. We start by sourcing the script we want to test, loading in the functions we are targeting. We group our tests by using the `describe` function, which is solely for aesthetics when the script runs. Then we test each function individually by creating bash functions beginning with `test_`. This informs `testdummy` to execute this function as a unit test. The contents of each function contain a few helpers, namely `cmd`, `assert_content`, and `assert_rc`. These helpers execute a command, then assert the output and return code of that command. These helpers could be ignored altogether and custom logic could be used. So long as the test function returns 0, the test is considered to have passed; any other exit code is assumed to be a failure.

Now running our test, we should see the following output:

{% highlight bash %}
$ testdummy myscript_test.sh

test functions
  test usage function                                  [PASS]
  test error function                                  [PASS]
  test main function                                   [PASS]
  test main function no args                           [PASS]
  test main function help args                         [PASS]

test script args
  test empty arg                                       [PASS]
  test help arg                                        [PASS]
  test single arg                                      [PASS]
  test multiple args                                   [PASS]
-------------------------------------------------------------
Tests:    9  |  Passed:    9  |  Failed:    0  |  Time:    1s
{% endhighlight %}

We can see that each of our tests are run without any errors. We also get a summary of the run.

## Downloading and Contributing

You can download the script from the [testdummy GitHub repository](https://github.com/nextrevision/testdummy). Pull requests and GitHub issues are welcome!
