---
layout: post

title: Error Catching in Ansible
subtitle: "How to emulate a try-catch in Ansible tasks"
cover_image: blog-cover.jpg

excerpt: "So what if you want to write an Ansible script that requires you to shut down Apache at the beginning, run the rest of your commands and then restart Apache at the end and one of your commands fails in the middle? This leaves you with your a failed script and Apache still stopped."

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---

##Error Catching in Ansible

###The Problem

So what if you want to write an Ansible script that requires you to shut down Apache at the beginning, run the rest of your commands and then restart Apache at the end and one of your commands fails in the middle? This leaves you with your a failed script and Apache still stopped.

This is a problem if you are trying to quickly make an update and can't be taking down a server indefinitely upon failure. The problem is that Ansible doesn't support an explicit form of try/catch. It only supports the "try" part. So we work with it.

###The Solution
To fix the problem I had to use `ignore_errors`. So I want to shut down Apache, run some Yum commands, and then regardless of the results, restart Apache. I begin by simply shutting down Apache. The next command in my case is to run `yum clean all`. This is the first place that failure is possible. So I register the result as the `clean_result` variable. This includes the output from running the command, including a boolean regarding whether or not it failed. And finally you want to ignore any errors with `ignore_errors: True`.

So far you've got:

{% highlight yaml %}
---
- hosts: <hosts_names>
  sudo: yes
  tasks:
   - name: shut down apache
     service: name=apache state=stopped
   - name: clean all packages
     shell: yum clean all
     register: clean_result
     ignore_errors: True
{% endhighlight %}

All's fine and dandy, but what happens if that actually fails? You don't want to go running `yum update` all willy nilly if cleaning failed. So you need to add a conditional to that command. So you add the line `when: clean_result | success`. This checks the boolean `clean_result.success` that was registered in the `clean all packages` task. You could do the opposite with `clean_result | failed`. Since you want to start Apache back up regardless of whether or not the packages update, make sure to ignore errors again.

If you wanted more tasks between updating packages and restarting Apache, you'd also need to register an `update_result` variable so you can check for success. Here we don't need it since we're jumping straight to restarting Apache.

Now you've got:

{% highlight yaml %}
---
- hosts: <hosts_names>
  sudo: yes
  tasks:
   - name: shut down apache
     service: name=apache state=stopped
   - name: clean all packages
     shell: yum clean all
     register: clean_result
     ignore_errors: True
   - name: update package {{ package }}
     yum: state=latest name={{ package }}
     ignore_errors: True
     when: clean_result | success
   - name: start apache
     service: name=apache state=started
     when: apache_restart | bool
{% endhighlight %}

And that's it.

### The Downsides

####No Counted Failures
There is one major downside to this method. You will never have any failures in your results.  It'll pretty much always look like this:

{% highlight text %}
ok=4   changed=2    unreachable=0    failed=0
{% endhighlight %}

Each individual task will still print a failure, but if someone unfamiliar with your script were to see the results, they would never know it broke. That task failure will look like this:

{% highlight text %}
TASK: [update package]
*********************************
failed: [hostname] => {"changed": false, "failed": true, "rc": 0, "results": []}
msg: No Package matching '<package-name>' found available, installed or updated
{% endhighlight %}

####What If There's Lots of Tasks?
This is clearly just a pain if you have a ton of tasks. It could be simplified by naming all of the registered variables the same thing. Instead of calling it `clean_result`, just call it `result`. Then you can copy and paste the same `register` and `when` into each task. `result` will be re-written each time with the newest results. It's not perfect but is manageable.

###Conclusion
This is an imperfect solution to a shortcoming of Ansible as it exists now. Sometimes it's just necessary to keep a server running or to do any other clean up task. Please comment if anyone finds a more ideal solution, as I would love to know it.

