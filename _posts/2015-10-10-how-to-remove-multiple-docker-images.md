---
layout:     post
title:      How to remove multiple Docker images
date:       2015-10-10 20:18:00
summary:    A simple shell command will help you to remove multiple docker images.
categories: blog
tags:       docker
---


When I’m playing with Docker for a new project I usually end up having dozens
or even more of `<none>` images:

{% highlight bash %}
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              affd29c496e2        3 seconds ago       525.5 MB
<none>              <none>              aeffd5bae1bb        27 seconds ago      526.3 MB
<none>              <none>              55b57705daa5        2 minutes ago       526.3 MB
mongo               3                   e047731b9266        4 days ago          261.3 MB
gliderlabs/alpine   3.2                 2cc966a5578a        3 weeks ago         5.25 MB
golang              1.4                 c5a2538d92ec        4 weeks ago         517.3 MB
{% endhighlight %}

It's pretty annoying to remove images one by one and copy/paste their `IDs`,
so I came up with simple one line command to remove all the `<none>`:

{% highlight bash %}
$ docker rmi -f $(docker images | grep "<none>" | awk '{print $3}')
Deleted: aeffd5bae1bbc107116634cc86a0d07f43ab448fc341e529b910ee85ed4b9c5d
Deleted: 5c7cb2f19237ff87d7f47a770307ec42cac666a9184ec39e0903d4f584b0e4e9
Deleted: 55b57705daa53e579e561244fd3a42f86a504b607c4915e7173e9904cc746f93
Deleted: dabad59a42fd1e7ea32acda815655e64de57d3d20632d24d87e6dfd0cb03d5b9
Deleted: 23a6040841af053dfb0e4a96612f827b5121b4736398bf282ab96240426b71ee
{% endhighlight %}


And as you can see no more `<none>` images:
{% highlight bash %}
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
mongo               3                   e047731b9266        4 days ago          261.3 MB
gliderlabs/alpine   3.2                 2cc966a5578a        3 weeks ago         5.25 MB
golang              1.4                 c5a2538d92ec        4 weeks ago         517.3 MB
{% endhighlight %}

You can also change filter criteria for `grep` command to what ever you want to remove.

Or even go further and create an alias in your `~/.bash_profile`:

{% highlight bash %}
dockerRemoveImagesWithCriteria() {
    if [[ -z "$1" ]]; then
        $1="<none>"
    fi

    count=$(docker images | grep "$1" | wc -l)

    if [[ "$count" -eq 0 ]]; then
        echo "No docker images found for criteria '$1'"
    else
       docker rmi -f $(docker images | grep "$1" | awk '{print $3}')
    fi
}

alias docker-remove=dockerRemoveImagesWithCriteria
{% endhighlight %}

...do not forget to source your `~/.bash_profile` :

{% highlight bash %}
$ source ~/.bash_profile
{% endhighlight %}

...and call it from the terminal:

{% highlight bash %}
$ docker-remove none
Deleted: 13bb8e2bbc8731802084b496ed9909d9818c58ff219e6043bfbc2cb1fabcd1ad
Deleted: a75b35de499ffd7959bced489e87a83ab3505b6af47e58eb26bf259e468a95d0
Deleted: 3aedddfdd65de9fb2b2ce14378b8cd051fbea775b3959eead24bde551f945b10
{% endhighlight %}