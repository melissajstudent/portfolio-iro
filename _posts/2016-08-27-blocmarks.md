---
layout: post
title: BlocMarks Project Case Study
thumbnail-path: "img/MarkIt_index.png"
---
Blocmarks is a Bloc project where the developer creates a site that allows users to manage and share bookmarks.

Explanation
-----------
Site functionality includes:

* Community aggregation of bookmarks (URLs)
* URLs organized by Topic
* View bookmarks by image and page title
* View only my favorites and submitted bookmarks
* User has the ability to like (or favorite) any bookmark
* User has the ability to edit and delete any topic or bookmark she creates

Problem
-------
As the first Bloc project, this was the first un-guided introduction to the Rails environment and how information is transferred between models, controllers, and views. The features required implementation of:

* user authentication
* CRUD operations on topics and bookmarks that were related
* ability to view images and site meta-data given a URL

Solution
-------
Solving the need for CRUD functionality on topics and bookmarks was a straight-forward implementation of a one-to-many relationship between those two tables and the user table.

Logic                 | Schema
-----                 | ----
Topic  <br> * has many bookmarks  <br> * belongs to one user | `t.index ["user_id"], name: "index_topics_on_user_id", using: :btree`
Bookmark  <br> * has one topic  <br> * belongs to one user | `t.index ["topic_id"], name: "index_bookmarks_on_topic_id", using: :btree` <br> `t.index ["user_id"], name: "index_bookmarks_on_user_id", using: :btree`
   | `add_foreign_key "bookmarks", "topics" ` <br> `add_foreign_key "likes", "bookmarks"` <br> `add_foreign_key "likes", "users"` <br> `add_foreign_key "topics", "users"`

Implementation of user authentication and image and metadata display from a URL were accomplished through gems. The gems proved valuable for quickly implementing the functionality but they sometimes introduced complexity into the code that wasn't immediately obvious.

Gems used include:

* Devise - user authentication
* Embedly - display images, title, description, etc from a URL
* Pundit - policy enforcement

### Devise
This gem was challenging because of how deeply embedded it is in the app and it's not obvious where the integrations take place. For example, making the user confirmable required additional code in test scripts to provide confirmed at data. Missing this caused tests to fail although the underlying code was correct.

Devise also adds it's own views. Fortunately, these views are customizable by downloading and editing their code.

### Pundit
This gem allows the developer to specify access levels. So, for example, one could allow an admin to do anything whereas a typical member, could only manipulate her own submissions. This was easily enforced by adding `authorize <instance variable>` to the desired method in the controller file.

### Embedly

This was a very easy gem to implement requiring code in only two places (after adding the gem to Gemfile and `require embedly` to config/application.rb).

**app/helper/application_helper.rb**

```
module ApplicationHelper

require 'embedly'

  def display(url)
    embedly_api =Embedly::API.new :key => ENV['EMBEDLY_KEY']
    obj = embedly_api.oembed :url => url
    image_tag((obj.first.thumbnail_url).to_s.html_safe, size: "240x150", class: 'card-img-top')
    #(obj.first.html).to_s.html_safe --> see all page info
  end

  def page_title(url)
    embedly_api =Embedly::API.new :key => ENV['EMBEDLY_KEY']
    obj = embedly_api.oembed :url => url
    obj.first.title
  end
end
```

The code to call the methods from various views is as follows.

* To display the first thumbnail of within the URL, `<%= display(bookmark.url) %>`<br>
* To display the URL page title and open the URL in a new tab, `<%= link_to page_title(bookmark.url), bookmark.url, target: '_blank' %>`

Results
-------
The results of the above solution implementation was a simple-to-use site.

Page Name    |   Screenshot
---------    |   ----------
Sign In      |   ![]( {{ site.base }}/img/MarkIT_index.png)
Sign Up      |   ![]( {{ site.base }}/img/MarkIt_sign_up.png)
Post Sign-In |   ![]( {{ site.base }}/img/MarkIt_all_bookmarks.png)
My Bookmarks |   ![]( {{ site.base }}/img/MarkIt_my_bookmarks.png)

Conclusion
----------
Overall, BlocMarks is a good introductory project in that it introduces you to core Rails concepts yet is simple enough for the earliest learner to achieve success. It also allows for multiple expansion opportunities. A few that I may implement if time permits include:

* use of modals to add and edit topics and bookmarks
* pagination
