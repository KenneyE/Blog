---
layout: post

title: Finding a Rails Model That Has All Associations
subtitle: "How to filter on a Rails model where all associations exist."
cover_image: blog-cover.jpg

excerpt: "For a project I'm working on, I was trying to build a check-box filter, where each check-box represents a relation in a has_many through association, and each new checked box further reduces the results. This is my solution."

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---

##Rails: Find Models with ALL Associations

### The Problem
For a project I'm working on, I was trying to build a check-box filter, where each check-box represents a relation in a has_many through association, and each new checked box further reduces the results.

For example, I've got a bunch of hospitals each with a set of features. Each check-box corresponds to a feature, and with each newly checked box, the number of hospitals returned should decrease, only return hospitals with ALL of the matching features.

I have an app with the following models:

- Hospital
- HospitalFeatures
- Features

with HospitalFeatures being the join table between the two primary models. Here are the associations:

{% highlight ruby %}
class Hospital < ActiveRecord::Base
  has_many :hospital_features
  has_many :features, through: :hospital_features
end
{% endhighlight %}

{% highlight ruby %}
class HospitalFeature < ActiveRecord::Base
  belongs_to :hospital
  belongs_to :feature
end
{% endhighlight %}

{% highlight ruby %}
class Feature < ActiveRecord::Base
  has_many :hospital_features
  has_many :hospitals, through: :hospital_features
end
{% endhighlight %}

I have a check-box form that lists all of the features available.

{% highlight erb %}
<%= label_tag("helipad", "Helipad") %>
<%= check_box_tag("features[]", "helipad") %>

<%= label_tag("telesurgery", "Telesurgery") %>
<%= check_box_tag("features[]", "telesurgery") %>

<%= label_tag("emergency room", "Emergency Room") %>
<%= check_box_tag("features[]", "emergency room") %>
{% endhighlight %}

### The Solution
So if you check "helipad" and "telesurgery", it should return only hospitals with both "helipad" AND "telesurgery" rather than any hospital with either "helipad" OR "telesurgery".

When I set out to do this, I assumed it would be trivial in Rails. Turned out not to be the case. Here's how I got it done.

First you will need to join your tables...
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
{% endhighlight %}
At this point, you've just got a big table with everything in it.

So you filter for where the feature name is in the array of searched features:
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where('features.name in (?)', searched_features)
{% endhighlight %}
Where `searched_features` is something like `["telesurgery", "helipad"]`.

So if you want 2 features, and a hospital has both, you will get 2 copies of that hospital. So you can group on hospital.id
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where('features.name in (?)', searched_features)
  .group('hospitals.id')
{% endhighlight %}
At this point, you will have all the hospitals that had ANY of the searched features rather than ALL of them. To fix that, you make sure that `COUNT(*)` is equal to the number of searched features. The final query being:
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where('features.name in (?)', searched_features)
  .group('hospitals.id')
  .having('COUNT(*) = ?', searched_features.length)
{% endhighlight %}
And there you have it. `filtered_hospitals` will be only the hospitals that have an association for every one of the desired features.

###My Original Attempts
If you're still reading, that means you didn't just copy and paste that last snippet of code and want to actually learn why more obvious methods might not work. That's awesome.

####First Go
The move obvious and simplest attempt would be something like:
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where( features: {name: searched_features} )
{% endhighlight %}
This does the same join as the final solution, but looks for features where `feature.name` is in `searched_features`. It's the same as:
{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where( 'features.name in (?)', searched_features )
{% endhighlight %}
This does the exact thing I was trying to avoid, which returns any hospital that has ANY searched feature. So adding features increases your number of results rather than narrowing it down.

####Second Go - Bad SQL
My next attempt was to work with somewhat more raw SQL. I'd build a query string to evaluate based on each feature in the `searched_features` list.

{% highlight ruby %}
query_string = ("features.name = ? AND " * searched_features.length)[0..-6]
{% endhighlight %}
This just repeats "features.name = ? AND" as many times as there are features, and then removes the final AND. Not the cleanest, admittedly. Then I would run the following query:

{% highlight ruby %}
filtered_hospitals = Hospital.joins(:features)
  .where(query_string, searched_features)
{% endhighlight %}
This just doesn't work at all. When you join the hospitals and features table, you end up with multiple rows of the same hospital. One for each associated feature. This query would look for a single row, with all of the features. That's obviously impossible as each row for a given hospital can only have one feature. Bad attempt.

###Conclusion
This is just a query that seems like it would be very common and like Rails would have a built-in query for just this. Turns out that's not the case and that it's actually harder to find a clear solution than I expected. Once you see it though, it makes perfect sense and I think this will be a useful pattern to remember down the road.