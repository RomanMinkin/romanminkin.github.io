---
layout:     post
title:      Nested queries in MongoDB shell (almost joins)
date:       2016-10-02 20:57:26
summary:    Simple way to query/cross match data across multiple collections.
categories: blog
tags:       mongodb
---

I'm going to show you how to easily query data across multiple collections in MongoDB shell. Which is almost equal to SQL's JOIN but not ;)

Assume we have two collections `users`

{% highlight javascript %}
> db.users.insert({_id: "usr-1", name: "Tom"})
> db.users.insert({_id: "usr-2", name: "John"})
> db.users.insert({_id: "usr-3", name: "Kate"})
> db.users.find()

{ "_id" : "usr-1", "name" : "Tom" }
{ "_id" : "usr-2", "name" : "John" }
{ "_id" : "usr-3", "name" : "Kate" }
{% endhighlight %}

 and `posts`

{% highlight javascript %}
> db.posts.insert({_id: "pst-1", body: "body 1", tags: ["apples", "oranges"], user_id: "usr-1"})
> db.posts.insert({_id: "pst-2", body: "body 2", tags: ["apples"], user_id: "usr-1"})
> db.posts.insert({_id: "pst-3", body: "body 3", tags: ["oranges"], user_id: "usr-2"})
> db.posts.insert({_id: "pst-4", body: "body 4", tags: [], user_id: "usr-1"})
> db.posts.insert({_id: "pst-5", body: "body 5", tags: ["apples"], user_id: "usr-3"})
> db.posts.find()

{ "_id" : "pst-1", "body" : "body 1", "tags" : [ "apples", "oranges" ], "user_id" : "usr-1" }
{ "_id" : "pst-2", "body" : "body 2", "tags" : [ "apples" ], "user_id" : "usr-1" }
{ "_id" : "pst-3", "body" : "body 3", "tags" : [ "oranges" ], "user_id" : "usr-2" }
{ "_id" : "pst-4", "body" : "body 4", "tags" : [ ], "user_id" : "usr-1" }
{ "_id" : "pst-5", "body" : "body 5", "tags" : [ "apples" ], "user_id" : "usr-3" }
{% endhighlight %}

...and let's say we need to find all the users who has at least one post tagged with **oranges**.

There are quite a few options to do it in half manual way or with MongoDB v3.2 and new
[$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/) operator from aggregation framework,
but here is a pretty simple solution which is easy to remember and it’s a one liner:

{% highlight javascript %}
db.users.find({_id: {$in: db.posts.find({tags: {$in: ["oranges"]}}).map(function(post){ return post.user_id })}})

{ "_id" : "usr-1", "name" : "Tom" }
{ "_id" : "usr-2", "name" : "John" }
{% endhighlight %}

Let's break it down:

* First part is just a standard finding users with `_id` matching values `$in` a given array `db.users.find({_id: {$in:...})`

* Second part is a sub query which uses JavaScript [map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) function on the query
[cursor](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/) and maps results into array of users' `_id`
(as we know mongodb query does not return an array of objects, but [cursor](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/))

* Inside the `map()` functions we specify a callback functions and what field we want to return from every post object. If we run just the second query we would get an array of users' `_id`
{% highlight javascript %}
> db.posts.find({tags: {$in: ["oranges"]}}).map(function(post){ return post.user_id })
[ "usr-1", "usr-2" ]
{% endhighlight %}

* So basically under the hood our original query becomes this
{% highlight javascript %}
> db.users.find({_id: {$in: [ "usr-1", "usr-2" ]}})
{% endhighlight %}

Another trick is if we have a case when second query users `.FindOne()` which returns a singular object instead of [cursor](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/), we do not need to user `map()` function and out query becomes a bit simpler
{% highlight javascript %}
> db.users.find({_id: db.posts.findOne({_id: "pst-5"}).user_id})
{ "_id" : "usr-3", "name" : "Kate" }
{% endhighlight %}

Hope you enjoy! And keep hacking!