---
layout: post
title:      "MegaTravelShare is there!!!!!"
date:       2019-10-09 22:56:05 +0000
permalink:  megatravelshare_is_there
---

After long hours of coding session, I finally got done with the MegaTravel Share website. <br>  
A website That let the users share their travel experiences, Mega Travel Share resembles the Instagram web site but it's essence still remains in the travel domain. <br>
Given such a short amount of time, there are things that could have been done, but got left out. With such constraints, I only worked on the essentials: 
<ul> 

<li>User
<ul>
<li>User can Log in and log out with the has_secure_password macro</li>
</ul>
</li>

<li>Post
<ul>
 <li>A user has many posts that he or she can create, view, edit, and delete.</li> 
 </ul>
 </li>
  
<li> Like
<ul> 
<li>A user can like a post, also dislike a post if he or she already like it.</li>
</ul>
</li>
 
 <li> Comment
 <ul>
 <li>A user has many comment</li>
 <li>A user can comment a post and delete his or her own comments</li>
 </ul>
 </li>
 
 <li> Follow
 <ul>
  <li>A user can have many followers and people that he or she follows. </li> 
	 <li>A user can unfollow and followback someone</li>
 </ul>
 </li>
</ul>  
That was a relatively easy web site to buid. However, I did encouter some challenges on the way. <br> 
Maybe the most challenging part of this website was the "Follow" feature.  let's take a look at the User and the Friendship model.<br><br> 
User model. 
<hr>
<code>
     class User < ActiveRecord::Base
		 
    has_many :posts 
    has_many :comments
    has_many :likes
		
    has_many :follower_relations, foreign_key: :following_id, class_name: "Friendship"
		
	has_many :followers, through: :follower_relations, source: :follower
	
	
	
	
		
    has_many :following_relations, foreign_key: :follower_id, :class_name => "Friendship" 
		
    has_many :followings, through: :following_relations,  source: :following
		
 
    has_secure_password
		
    end
</code> 
<br><br>
Friendship model.
<hr> 
<code>
    class Friendship < ActiveRecord::Base
				 
    belongs_to :follower,foreign_key: :follower_id, :class_name => "User"
    belongs_to :following, foreign_key: :following_id, :class_name => "User" 
									 
    end
</code>
<br> 


If it looks confusing to you do not worry. I had the same face on when I first saw this method: <a href="https://learn.co/tracks/online-software-engineering-structured/orms-and-activerecord/associations/video-review-aliasing-activerecord-associations">Aliasing</a> 

Without too many details, the Friendship model is used as joining table to join a user with another user. 
All that would be impossible without the Aliasing method. <br> 
I had so much fun, working on this website and I feel like it will never be completed because there are so many functionality to add. However,this web site is more than "good enough" to pass the flatrion requirements.(^_^)



 


