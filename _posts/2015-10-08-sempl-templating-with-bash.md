---
title:  "Sempl - Templating with Bash"
date:   2015-10-08
description: Stupid Simple Bash Templating Script
---

## Background
A few months back, I began working in an environment that was migrating a number of applications to Docker containers. Some of these applications were new, others had existing configuration management code bases to install/manage them. While migrating the installation logic to a Dockerfile, there came the decision of what to do with environment-specific configuration.

Often time, and in numerous examples on how to get started with containers, we are shown how to install some libraries, add some static files, and specify a command to run an application. Slightly more complex examples could detail how to link to other containers by name. Most of the examples I've seen do not include something prevelent in most environments, and that's what to do with environment-specific application data.

Existing configuration management solutions provide hierarchies, data-bags, or other lookup sources to pass application data directly to the CM application, and then render a template based on that data. We employed some of these techniques, but found it a bit frustrating (not to mention time consuming) installing a full CM tool for just handling environment specific configuration. Ideally, we wanted to pass in data to the container at runtime via environment variables (see [http://12factor.net/config](http://12factor.net/config)).

## Sempl

Driven out of the desire to have as minimal footprint inside of a container as possible, and using common Linux utilities, [sempl](https://github.com/nextrevision/sempl) was created. Sempl uses environment variables, either inherited from an environment or sourced from a file (optionally encrypted) to render a template. It also allows for using command substitution and inline bash similar to other templating engines.

## Workflow

### Dockerfile

To provide an example, here is a relatively simple Dockerfile:

{% gist nextrevision/12b78ef5ea72ff0e71ab Dockerfile %}

All we are doing here is installing Apache, downloading sempl, and adding a few application files, namely the index.html template (```index.html.tmpl```). Let's take a look at the template file.

{% gist nextrevision/12b78ef5ea72ff0e71ab index.html.tmpl %}

This template file makes use of a few of the features available in sempl: variable expansion, command substituion, and bash looping. Lets step through the interesting lines:

* __line 3__: we are executing the ```hostname -f``` command at render time and outputting the value in the template
* __line 7__: try and resolve the variable ```HEADING```, otherwise default to the value ```Hello!```
* __line 8__: same logic as __line 7__
* __line 10__: instruct sempl to enter in a bash code block, whereby everything preceded with a ```# ``` is interpretted as a command, and everything else is treated as output (note the single space after the ```#```)
* __line 11__: simple bash ```if``` statement to test the value of ```DEBUG```
* __line 15__: perform a bash ```for``` loop with command substitution, enumerating a list of machine IPs
* __line 17__: close the ```for``` loop
* __line 20__: close the ```if``` statement
* __line 21__: tell sempl to stop evaluating the loop

Lastly there is the ```startup.sh```:

{% gist nextrevision/12b78ef5ea72ff0e71ab startup.sh %}

This will execute sempl passing in the template file and the desired output file then start and run Apache in the foreground.

### Running our workflow

With all of these files in the same directory, I can run the following commands to build an image from the Dockerfile and run the container with all defaults:

{% highlight bash %}
docker build -t appx .
docker run --rm -it -p 8080:80 appx
{% endhighlight %}

Now if I browse to [http://localhost:8080](http://localhost:8080), or the IP of the docker host the container is running on, I should see our application running with all the defaults:

![AppX Defaults](/assets/images/2015-10-08-appx-default.png)

Now that we know our application and template are working as expected with all default values, lets tweak the runtime rendering by passing in a custom ```HEADING``` and ```REGION``` via environment variables.

{% highlight bash %}
docker run --rm -it -p 8080:80 \
  -e 'HEADING=Guten Tag!' \
  -e 'REGION=eu-west-1' \
  appx
{% endhighlight %}

We should get the following:

![AppX Defaults](/assets/images/2015-10-08-appx-heading-region.png)

Now lets test the debugging condition in our template by setting that environment variable:

{% highlight bash %}
docker run --rm -it -p 8080:80 \
  -e 'HEADING=Guten Tag!' \
  -e 'REGION=eu-west-1' \
  -e 'DEBUG=1' \
  appx
{% endhighlight %}

![AppX Defaults](/assets/images/2015-10-08-appx-debug.png)

## Conclusion

Sempl is meant to be a lightweight utility that allows using familiar bash logic inside of templates with low overhead and learning curve. Templates are manipulated by environment variables, or via a variable file (optionally encrypted). If you have an idea for how to extend/improve the tool or run into any issues, feel free to open a PR or issue on the [github page](https://github.com/nextrevision/sempl).
