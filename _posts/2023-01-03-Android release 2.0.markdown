---
layout: post
title:  "Android update 2.0"
date:   2023-01-03 13:43:24 +0100
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight kotlin %}
TT2.initialize(
  context = //applicationContext
  apiUrl = // The api url to connect to
  apiKey = // Your API key
  clientId = //Your client ID
  // The intent which will be used by the foreground service running the positioning logic, it will also 
  // handle user interaction with the notification that will be displayed in the notification center when the app is in background.
  serviceNotificationIntent =  Intent(context, ClassToReceiveIntent::class.java) 
) { error ->
  if (error != null) {
    // Show error to user in case of any exception happened during initialization including network exception
  } else  {
    // Safe to do the next steps
  }
}
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
