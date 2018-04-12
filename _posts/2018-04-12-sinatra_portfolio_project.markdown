---
layout: post
title:      "Sinatra portfolio project"
date:       2018-04-12 05:43:53 +0000
permalink:  sinatra_portfolio_project
---


I was recently chatting with a friend of mine about his art practice and the difficulty he had with keeping an inventory for all his artwork and the materials he used to create it. This is when I thought a Sinatra application could be a perfect solution. He could log in as a user, add artworks/materials to the database, create relationships between his artworks and materials (i.e. see which materials were used in an artwork and which artworks were created with a specific material), and edit or delete any item. 

The first step was to create artworks & materials and be able to view them but one of the things that make this app what it is is the ability to associate both materials and artworks. An artwork not only has many materials but materials can also have many artworks. 

It was evident from the start that things would be much harder to navigate for the user if there was no way of categorizing items. I felt it was important enough for any item added to include a category. However, I still included an option to allow the user to view items by most recently added if so desired.

My primary models include the user, artwork, material and category:
```
class User < ActiveRecord::Base
	has_secure_password
	has_many :materials
	has_many :artworks
	has_many :categories
	validates_presence_of :username, :password
end

class Artwork < ActiveRecord::Base
	has_many :artwork_materials
	has_many :materials, :through => :artwork_materials
	has_many :artwork_categories
	has_many :categories, :through => :artwork_categories
	belongs_to :user
	
		extend CarrierWave::Mount
	mount_uploader :file, Uploader

  def slug
  	title.parameterize.truncate(80, omission: ' ')
  end

  def self.find_by_slug(slug)
    Artwork.all.find{|artwork| artwork.slug == slug}
  end
end

class Material < ActiveRecord::Base
	has_many :artwork_materials
	has_many :artworks, :through => :artwork_materials
	has_many :material_categories
	has_many :categories, :through => :material_categories
	belongs_to :user
	
		extend CarrierWave::Mount
	mount_uploader :file, Uploader

  def slug
  	name.parameterize.truncate(80, omission: ' ')
  end

  def self.find_by_slug(slug)
    Material.all.find{|material| material.slug == slug}
  end
end

class Category < ActiveRecord::Base
	has_many :material_categories
	has_many :materials, :through => :material_categories
	has_many :artwork_categories
	has_many :artworks, :through => :artwork_categories
	belongs_to :user
	
	  def slug
    name.parameterize.truncate(80, omission: ' ')
  end

  def self.find_by_slug(slug)
    Category.all.find{|category| category.slug == slug}
  end
end
```

If you take a look at my artwork and material models you may notice a gem called CarrierWave. I won’t go into detail but this gem along with Fog and ImageMagik allows the user to add and view images. For this instance I used Amazon S3 Buckets as a repository for my images. Although this falls outside of project requirements I felt it was imperative to feature images in my app. For more about using CarrierWave with Sinatra see these very handy guidelines: https://www.softcover.io/read/27309ccd/sinatra_cookbook/gallery

The overall design for this application doesn’t allow for interaction between users as it was intended to be used privately. So the only artworks and materials the user sees are their own. In order to achieve this I needed to build a couple helper methods that only select items from the current user. These helper methods also provide an array of either material categories or artwork categories which are used in to determine the appropriate category items to display (btw there is probably a much more efficient way of doing this…)

```
def category_by_artwork
  @artwork_category = []
  @current_user.artworks.each do |artwork|
    artwork.categories.collect do |category|
      @artwork_category << category.id
    end
  end
end

<% Category.order("id DESC").all.each do |category| %> 
  <%if @artwork_category.uniq.include? category.id%>
    "do stuff with category and category.artworks in here"
  <%end%>
<%end%>
```

There are a few more features I would like to eventually add such as displaying by most recently updated, pagination or dynamic content loading, and a search function. But even in its current state I’m very much looking forward to putting this app to good use. 


