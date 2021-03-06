# sinatra-thumbnails

Generate, serve cached thumbnails in Sinatra. Good for simple file-based CMS's.

## Minimal example

Consider a small sinatra app that serves a mix of markdown and jpeg content,
loosely arranged as follows.

```
portfolio/
 Gemfile
 app.rb
 views/
    layout.haml
    gallery.haml
 content/
   some_gallery/
     1.jpg           # a pic
     test.jpg        # another pic
     desc.mdown      # a description of some_gallery
   another_gallery/
     yap.jpg         # yet another pic
     desc.mdown      # description of another_gallery
```

I use bundler, so in my `Gemfile`

```ruby
gem 'sinatra-thumbnails', :require => "sinatra/thumbnails"
```

I have a `app.rb` like

```ruby
require 'rubygems'
require 'bundler'
Bundler.require

module ContentHelpers
  def content_path(file)
    File.join("content",params[:gallery], file)
  end
end

helpers do
  include ContentHelpers
end

get "/content/*" do
 file = "content/" + params[:splat].first
 send_file file
end

get "/:gallery" do
  # A gallery has markdown description snippet
  #
  file =  content_path("desc.mdown")
  if File.exists? file
    @description = Maruku.new(File.read(file)).to_html
  end

  # ... and a bunch of jpegs...
  #
  @images = Dir.glob(content_path("*.{jpg,jpeg,JPG,JPEG}"))
  haml :gallery
end
```

This is the `gallery.haml` partial, notice the call to `thumbnail_url_for`:

```
#gallery
  #description
    ~ @description
  #images
    - @images.each do |image|
      .asset
        %a{:href => image}
          %img{:src => thumbnail_url_for(image, "200x200")}
```

That's it.

## What it does / why this works

When the browsers loads `http://www.myportfolio.com/some_gallery` there's the
following tag

```
<img src='public/thumbnails/200x200/content/some_gallery/test.png?original_extension=jpg' />
```

in the middle of the HTML I get. When a request for that particular URL is comes
in, the ImageMagick's `convert` creates the `test.png` thumbnail from `test.jpg`
ImageMagick and the directory structure becomes:

```
portfolio/
  ...
  content/
    ...                  # same as before
  public/
    thumbnails/
      200x200/
        content/
          some_gallery/
            test.png
```

The next time the very same URL is loaded, and unless `test.jpg` has changed in
the meantime, the following path is served instead, without query string and `public/`:

```
<img src='thumbnails/200x200/content/some_gallery/test.png' />
```

##  Crop to Fit

Thumbnails will maintain their aspect ratio. In order to make crop the image to fit a specific thumbnail size, append -crop to the format.
For example, for following will create a square thumbnail with edges cropped off.

```
<img src='public/thumbnails/200x200-crop/content/some_gallery/test.png?original_extension=jpg' />
```

## Contributing to sinatra-thumbnails

Fell free to fork and submit pull requests.

## Copyright

Copyright (c) 2011 João Távora. See LICENSE.txt for
further details.

