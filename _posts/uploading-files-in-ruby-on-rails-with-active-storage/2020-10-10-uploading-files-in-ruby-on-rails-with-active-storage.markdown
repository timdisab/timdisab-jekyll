---
layout: post
title: Uploading Files In Ruby on Rails 6 With Active Storage
excerpt: This is a meta description
permalink: uploading-files-in-ruby-on-rails-with-active-storage
tags: [ruby on rails, active storage]

---

The ability to upload files to a web application is a fairly common feature.  With the arrival of Rails 5, Active Storage was added as an addition to the Rails code base.  Prior to Active Storage, file uploading functionality was added to Rails applications through the addition of Ruby gems—most notably [CarrierWave](https://github.com/carrierwaveuploader/carrierwave){:target="_blank"}, [Shrine](https://github.com/shrinerb/shrine){:target="_blank"} or [Paperclip](https://github.com/thoughtbot/paperclip){:target="_blank"}.

I recently spent a couple of days trying to get file uploads to work as a full CRUD action in Rails with Active Storage.  To get a sense of how Active Storage works, I did a few tutorials and built a couple of small apps.  As with many tutorials, there were a couple of edge cases that the tutorials didn’t cover so I decided to write this.  If you’re planning on adding file uploads to your Rails app, hopefully this will help you in some way.

## Uploading Files To a Basic Blog Post in Rails

To demonstrate how Active Storage works, let’s build a simple blog with attachments.  We’ll call this example **active_blog**.  Let’s create our blog:

{% highlight terminal %}
$ rails new active_blog
{% endhighlight %}

Next, we need to install Active Storage since it doesn’t get installed automatically when a new Rails app is created.  Let’s cd into our active_blog directory and run the installer for Active Storage:

{% highlight terminal %}
$ cd active_blog
$ rails active_storage:install
{% endhighlight %}

Let’s next generate our basic blog via scaffolding with a title and a body:

{% highlight terminal %}
$ rails g scaffold Post title:string body:text
{% endhighlight %}

Now run a migration:

{% highlight terminal %}
$ rails db:migrate
{% endhighlight %}

Just to make sure that everything is okay, let’s boot the Rails server just to make sure that our basic blog is there:

{% highlight terminal %}
$ rails s
{% endhighlight %}

Once the server is booted, if we go to <http://localhost:3000/posts> we should see our basic blog index page:

![empty posts index page](/posts_index_ss1.png)

Let’s add a couple of simple posts so our blog doesn’t look so empty:

![index page with posts](/posts_index_w_posts_ss2.png)

## Adding Attachments to Our Posts With an Upload Form

Now that we have a basic blog post via scaffolding, let’s dive in to adding upload functionality via Active Storage.

First, let’s add the database logic to the Post model.  With Active Storage, we can choose to attach one file per Post or many files per Post.  Depending on which one we want, the statement we add to the Post model is slightly different:

{% highlight ruby %}
# /app/models/post.rb

class Post < ApplicationRecord
  has_one_attached :header_image   # Use has_one_attached for only one file allowed
  has_many_attached :files         # Use has_many_attached for multiple files allowed
end
{% endhighlight %}

Next, we need to add the attachment as a permitted parameter in our controller. Note that there is a difference in syntax if we are allowing one attachment vs. many:

{% highlight ruby %}

# /app/controllers/posts_controller.rb

# Use a symbol without brackets for one attachment

def post_params
  params.require(:post).permit(:title, :body, :header_image)
end

# Use a Ruby symbol with brackets (array) for many attachments

def post_params
  params.require(:post).permit(:title, :body, files: [])
end
{% endhighlight %}

After adding the params to the Posts controller, we next need to add the form fields for our uploader to the post form:

{% highlight erb %}
#  use this for one attachment

 <div class="field">
   <%= form.label :header_image %>
   <%= form.file_field :header_image %>
 </div>

 # use this for many attachments, note we need multiple: true

 <div class="field">
   <%= form.label :files %>
   <%= form.file_field :files, multiple: true %>
 </div>
{% endhighlight %}

Note that we need to add the **multiple:true** statement for upload field that can add many files.  With both our blog post header image upload field as well as our multiple files upload field, the complete form code would look like:

{% highlight erb %}
 <%= form_with(model: post, local: true) do |form| %>
   <% if post.errors.any? %>
     <div id="error_explanation">
       <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

       <ul>
         <% post.errors.full_messages.each do |message| %>
           <li><%= message %></li>
         <% end %>
       </ul>
     </div>
   <% end %>

   <div class="field">
     <%= form.label :title %>
     <%= form.text_field :title %>
   </div>

   <div class="field">
     <%= form.label :header_image %>
     <%= form.file_field :header_image %>
   </div>

   <div class="field">
     <%= form.label :body %>
     <%= form.text_area :body %>
   </div>

   <div class="field">
     <%= form.label :files %>
     <%= form.file_field :files, multiple: true %>
   </div>

   <div class="actions">
     <%= form.submit %>
   </div>
 <% end %>
{% endhighlight %}

And the Post form will render in the browser like so:

![new post form with file uploads](/posts_form_ss3.png)

## Displaying Uploaded Files

To view our uploaded files, we need to access them on the Posts show page.  In its simplest form, we can simply link to the file by iterating through the files and linking to them.  This would be for our many files upload:

{% highlight erb %}
 # /app/views/posts/show.html.erb

 <% @post.files.each do |file| %>
   <%= link_to file.filename, rails_blob_path(file, disposition: :attachment) %>
 <% end %>
{% endhighlight %}

**Note:** the <kbd>disposition: :attachment</kbd>  parameter downloads the file when it is clicked. If you want to open it in the browser, make the statement <kbd>disposition: :inline</kbd>.

If we upload our files, save the Post and look at the Posts show page, we will the links to our files.  In this case, these are images but it could be any file type:

![show post with attachments](/post_show_with_attachments_ss4.png)

If we want to actually show the images and re-size them, we can update our gems with the [MiniMagick](https://github.com/minimagick/minimagick){:target="_blank"} gem and display our uploads in different sizes as in a photo gallery.  There are a lot of configuration options such as displaying PDFs, videos and other types of files.  Explore the documentation if this interests you (it’s a wide topic).

In the code below, we’ve installed MiniMagick and we’re calling for the header image to have a header image with dimensions of 600×500 pixels. For the attachments, we’re looping through them again but this time we’ve added some logic that tells Rails to view the attachments as images if possible, preview them if not and if all else fails to simply link to them:

{% highlight erb %}
 # /app/views/posts/show.html.erb

 <p id="notice"><%= notice %></p>

 <h1>
   <strong>Title:</strong>
   <%= @post.title %>
 </h1>

 <p>
   <%= image_tag @post.header_image.variant(resize: "600x500") %>
 </p>

 <p>
   <strong>Body:</strong>
   <%= @post.body %>
 </p>

 <% if @post.files.any? %>
   <h3>Attachments:</h3>
   <% @post.files.each do |file| %>
     <% if file.variable? %>
       <%= image_tag file.variant(resize: "400x400") %>
     <% elsif file.previewable? %>
       <%= image_tag file.preview(resize: "400x400"), rails_blob_path(file, disposition: :attachment) %>
     <% else %>
       <%= link_to file.filename, rails_blob_path(file, disposition: :attachment) %>
     <% end %>
   <% end %>
 <% end %>

 <%= link_to 'Edit', edit_post_path(@post) %> |
 <%= link_to 'Back', posts_path %>
 {% endhighlight %}

![show post with images](/post_show_images_ss5.png)

## Editing Uploaded Files

For our multiple files attached, if you edit them and upload new files, the default behavior is to overwrite the existing images and replace them with new ones.  This is fine if that’s what you’re looking to do.  But what if you simply want to add additional files to the existing ones you’ve already uploaded?

Under the default Active Storage configuration, you can’t do this.  You’d have to select the new files you want to upload in addition to the existing ones which would be quite cumbersome to do each time.

There’s a fix for this.  Under the _config/environments/development.rb_ file add the following line:

{% highlight ruby %}
 # config/environments/development.rb

 config.active_storage.replace_on_assign_to_many = false
 {% endhighlight %}

 With this change, we can add new images to our existing post and now our existing images will persist:

![show post with added images](/ss6_alternate.png)


## Deleting Uploaded Files

Since we only have one header image per post, we never really need to delete the image.  We can switch out the header image by editing it and uploading a different image.

For our many attached files, we may want to delete one or even several of them at some point.  Because deleting Active Storage attachments falls outside the default conventions for RESTful actions in Rails, we need to add both a new action to the posts controller as well as a new route to the **routes.rb** file.

First, let’s add a new action to our posts controller underneath the destroy method but above all of the private methods in the controller.  This action is called **delete_file** and contains the **purge** method that is necessary for deleting Active Storage attachments from the database:

{% highlight ruby %}
 # app/controllers/posts_controller.rb

 def delete_file
   file = ActiveStorage::Attachment.find(params[:id])
   file.purge
   redirect_back(fallback_location: posts_path)
 end
 {% endhighlight %}

 Next, we need to add a member route inside of the **routes.rb** file for the **delete_file** action.  This will create a route called **delete_file_post** that we can then reference in the view:

 {% highlight ruby %}
  # config/routes.rb

  Rails.application.routes.draw do
    resources :posts do
      member do
        delete :delete_file
      end
    end
  end
{% endhighlight %}

Finally, in the form partial for our posts, we need a way to reference each individual file so that it can be deleted.  In the form partial, we’ll add a section to show the attachments for each post as well as a link next to each attachment that will route to our **delete_file_post** action in the _posts_controller.rb_:

{% highlight erb %}
 <strong>Attachments:</strong>
 <ul>
 <% @post.files.each do |file| %>
   <li>
     <%= link_to file.filename, rails_blob_path(file, disposition: :attachment) %>
     <%= link_to 'Remove', delete_file_post_url(file.id), method: :delete, data: { confirm: 'Are you sure?' } %>
   </li>
 <% end %>
 </ul>
{% endhighlight %}

So our final rendered form will render in the browser like so with a **Remove** link next to each attachment for our many attached files:

![edit post with delete images](/edit_post_with_delete_attachments_ss7.png)

## Conclusion

In this tutorial, we have been able to generate a full set of CRUD actions for Active Storage attachments in a simple Rails app for both single and multiple file uploads.  Uploads can get quite a bit more complex in terms of how you want to display different kinds of files in the browser and whether or not you want to add drag and drop uploading.  But the functionality in this post covers most simple use cases.

## Additional Resources

If you’re looking for additional information on using Active Storage or how it works, I’ve found the following tutorials to be great resources:

[The Rails Guides Active Storage Overview](https://edgeguides.rubyonrails.org/active_storage_overview.html){:target="_blank"}

[Using Active Storage in Rails 6](https://pragmaticstudio.com/tutorials/using-active-storage-in-rails){:target="_blank"} by Mike Clark at the Pragmatic Studio

[File Uploading with ActiveStorage in Rails 5.2](https://gorails.com/episodes/file-uploading-with-activestorage-rails-5-2){:target="_blank"} by Chris Oliver at GoRails

[Ruby on Rails Drag and Drop Uploads with Active Storage, Stimulus.js and Dropzone.js](https://web-crunch.com/posts/rails-drag-drop-active-storage-stimulus-dropzone){:target="_blank"} from Andy Leverenz at Hello Rails
