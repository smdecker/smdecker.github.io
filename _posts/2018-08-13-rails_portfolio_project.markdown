---
layout: post
title:      "Rails Portfolio Project"
date:       2018-08-13 04:30:57 +0000
permalink:  rails_portfolio_project
---


For my rails app I decided to make an online radio platform. Essentially, users can browse the site for shows and its episodes, comment on and ‘favorite’ those episodes and explore associated genres. The actual creation of shows/episodes is restricted to admin only. (I’ll go into more detail about the ‘favorite’ function and genre tags below)

I ended up using Devise and Google oAuth. However, it was difficult to find documentation on using Devise with Google oAuth. Eventually I came across a couple of helpful resources that I used together to accomplish this task. If anyone is interested in using Devise and Google oAuth, here are the resources that saved me some time and frustration: https://www.interexchange.org/articles/engineering/lets-devise-google-oauth-login/ & https://github.com/zquestz/omniauth-google-oauth2

**The root of my app displays three sections:**
- Recommendations that are determined by admin ‘favorites’
- Recently aired - simply displays the last 15 episodes in the order which they were created
- Popular episodes - displays the 10 most favorited episodes


**Breaking down the favorite function** 

The favorite function involves three models, User, Episode and the join model FavoriteEpisode.
```
class User
  has_many :favorite_episodes
  has_many :favorites, through: :favorite_episodes, source: :episode
end

class Episode
  has_many :favorite_episodes
  has_many :favorited_by, through: :favorite_episodes, source: :user
end

class FavoriteEpisode
  belongs_to :episode
  belongs_to :user
end
```

In order to connect the associations in the FavoriteEpisode model I provided :favorites with the source: :user and :favorited_by with the source: :user. Essentially renaming :episode and :user associations. :favorited_by will return a list of users the episode has been favorited by and :favorites will return a list of episodes the user has favorited.

The episodes controller contains a favorite method that pushes the appropriate @episode into the current_user.favorites when “favorited” and deletes that episode when “unfavorited”. The redirect_to request.referrer just sends the user back to where the request came from. In this case the episode page.
```
def favorite
	if params[:type] == "favorite"
		current_user.favorites << @episode
		redirect_to request.referrer

	elsif params[:type] == "unfavorite"
		current_user.favorites.delete(@episode)
		redirect_to request.referrer
	end
end
```

Finally in the episode view, if the user is signed in they will be able to favorite/unfavorite an episode with a link(styled as a button) with a put method and a type: of either unfavorite or favorite. If the user is not signed in the link will take them to a the sign up/login page. The put action must also be nested in the routes under episodes resources like so: put :favorite, on: :member
```
  <% if user_signed_in? %>
    <% if current_user.favorites.include?(@episode) %>
      <%= link_to favorite_show_episode_path(@episode, type: "unfavorite"), method: :put do %>
        <div class="favorite-button" style="background-color:#6f6f6f">
          <p>favorited</p>
        </div>
     <% end %>
    <% else %>
      <%= link_to favorite_show_episode_path(@episode, type: "favorite"), method: :put do %>
        <div class="favorite-button">
          <p>favorite</p>
        </div>
      <% end %>
    <% end %>
  <% else %>
    <%= link_to new_user_session_path do %>
      <div class="favorite-button">
        <p>favorite</p>
      </div>
    <% end %>
  <% end %>
```


**Genre tags**

Another feature of this app is the genre tags. In my new episode form a collection_check_boxes is provided for the admin to either select/create a new genre or do both. 
```
<div class="select-box">
	<%= f.collection_check_boxes(:genre_ids, Genre.genre_order, :id, :name) do |genre| %>
		<%= genre.label class: "label-checkbox" do%>
		 <%= genre.check_box + genre.text %>
		<%end%>
	<% end %>
</div>
```

The models are fairly straight forward, episode has_many genres through: :episode_genres and genre has many episodes through: :episode_genres. The genre_attributes method in the episode model takes care of any genre selected or created:
```
def genres_attributes=(genre_attributes)
	genre_attributes.values.each do |genre_attribute|
		if !genre_attribute[:name].blank?
			genre = Genre.find_or_create_by(genre_attribute)
				self.genres << genre
		end
	end
end
```
The selected genres are then displayed on the episode show page and links to a genre show page that lists all episodes playing the particular genre.

I used a few other handy gems in this app including [friendly_id](https://rubygems.org/gems/friendly_id/versions/5.1.0 ) for pretty urls, [paperclip](https://rubygems.org/gems/paperclip) for image file uploads and [ransack](https://rubygems.org/gems/ransack) for some search functionality. 

But of course it couldn’t be considered a radio station without any audio functions, but hopefully these will be added once I complete the Javascript section.
