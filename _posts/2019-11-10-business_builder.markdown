---
layout: post
title:      "Business Builder"
date:       2019-11-10 23:09:24 -0500
permalink:  business_builder
---

<h2>Introduction</h2>
<p> My third Flatiron Project was the most challenging of all my previous projects. Fortunately, there are many people who had already discussed some aspect of my project. Something that made the creation of "Business Builder" easier than what it was supposed to be. </p>
<h2> Short Description</h2>
<p>Business Builder is an app that can give start-ups an idea of the structure of their business. It also serve as a platform to define projects, goals, and communicate information among members of a business.</p>

<h2>Demo</h2>
<div class = "container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/1u2r43YbhL4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

<h2>Tutorial part I</h2>
<p>As one can see on the video, some points have been left out. However it was not because "As programmers, we are lazy," but rather those points do not require as much research as the one I'm about to disscuss now.</p>
<h3>Action Cable</h3>
<p>If you watched the video till the end, you will be able to tell that the star of my application was a real time chat app. real time application can be built through action cable, which is Rail's built-in library with both server-side and client-side components to push data through websockets.In the case you're wondering how to make such a thing work, I'm about to walk you through the different steps.</p>
<h3>Steps</h3>
<ul>
<li>In the Gemfile</li>
<code>gem "bcrypt"</code>

<p>This gem can be used for authentication purpose; it requires the model wich will need to be authenticated to have an attribute of <code>password_digest:</code><br> <a href = "https://github.com/codahale/bcrypt-ruby/">bcrypt</a></p>

<li>Create a User model</li>
<code>rails g model user username:string email:string password_digest:string</code>
<br>
<br>

<li>Create a ChatRoom model</li>

`rails g model title:string `
<br>
<br>

<li> Create a Message model</li>

`rails g model message content:string user_id:integer chat_room_id:integer   attachment_data:text file_name:string`
	
<br> 
<br>
	
<li>
	Associtations
<ul>
	<li>app/models/user.rb</li>
	
	```
	class User < ApplicationRecord
		 has_many :messages
  end
	```

<br>
<br>
	
<li>app/models/chat_room.rb</li>

```
class ChatRoom < ApplicationRecord
       has_many :users
       has_many :messages, trhough: :user
end
```


<br>
<br>
		
<li>app/models/message.rb</li>
```
class Message < ApplicationRecord <br>
      belongs_to :chat_room <br>
      belongs_to :user<br>
end
```

<br>
<br>
		
		
</ul>
</li>
<p>Now that we are done with the associations we need to implement the action cable feature in our app.</p>
<li>Genarate a channel </li>
<code>rails g channel chat send_message  </code>
<br>
<p>In the previous code, <code>chat</code> is the name of the channel, and <code>send_message</code> is a function that we will use to send messages. Note that those names are customizable.</p>

<li>In your file javascripts/cable.js make sure to have this:</li> 
<br>

```
       (function() {
                  this.App || (this.App = {});

                 App.cable = ActionCable.createConsumer();

           }).call(this);	
```

<br>
<p>this code is telling your app to enable action cable. In your application, you may have "cable.coffee" instead of "cable.js". You can change the file into a .js and use <a href = "http://js2.coffee/">js2.coffee</a> to convert the .coffee code into .js and vice versa.</p>

<li>Now go to your routes.rb file</li>
<p>Add this code: <code>  mount ActionCable.server => "/cable"</code></p>
<p>I know that's a lot of information but bear with me.</p>
<li>Let's now add some code on the client-side in app/javascripts/channels/chat.js</li>

```
Query(document).on('turbolinks:load', function(){

                    var messages;
										
                    messages = jQuery('#messages');

    messages_to_bottom =  function(){ $("div.ui.segment.scroller").scrollTop(messages.prop("scrollHeight")) };
  
                                               messages_to_bottom();

                                             return App.global_chat = App.cable.subscriptions.create({
                                                                                                                           channel: "ChatChannel",
                                                                                                                           chat_room_id: messages.data('chat-room-id')
                                                                                                                        }, {
                                                                                                                               connected: function() {console.log( messages) },
                                                                                                                               disconnected: function() {},
                                                                                                                               received: function(data) {  
                                                                                                                               messages.append(data)
                                                                                                                           },
                                                                                                                  send_message: function(message, chat_room_id,file,file_name) {
                                                                                                                                                             return this.perform('send_message', {
                                                                                                                                                              message: message,
                                                                                                                                                             chat_room_id: chat_room_id,
                                                                                                                                                             file: file,
                                                                                                                                                             file_name: file_name
                                                                                                                                                             });
                                                                                                                               }
                                                                                                                                });
                                                                                                                             });
	```
	

<p>The connected method will run when the user subscribes to a channel. <br> The disconnected method will run when the user disconnect from a channel and the the received method will run when the server-side sends data to the client-side.<br> Also notice that we have a "send_message"; This methode will send data to the back-end in order to be processed.</p>
<p>function like message_to_bottom() will be explained later. Do not worry about that for now.</p>
<p>We still need to implement the show page for the chat_room</p>
<li>In you chat_rooms_controller.rb file have this: </li>
<br>

```
               def show 
		
                      @message = Message.new
				
				              @chat_room = ChatRoom.find_by(id: params[:id])
				
                      @messages = Message.where("chat_room_id = :chat_room",chat_room: @chat_room.id)
				
                end
```

<li>In your app/views/chat_rooms/show.html.erb</li>
<br> 

       ``` 
						<div class = "messages ui container "  id = "messages"  data-chat-room-id="<%= @chat_room.id %>">
                   <%= render @messages %>
              </div>			 
					 ```
								 
<p>The attribue  "data-chat-room-id" will help us have multiple chat_rooms to stream from. without it, all the users would be subscried to the same channel and people who are not supposed to read certain messages could have access to them. Obviously this is not what we want.</p>
<p>Now we need a form to submit the message</p>
<li>Update your  app/views/chat_rooms/show.html.erb like so </li>
<li>Set up a partial "_message" in the messages folder</li>
```
          <div class = "message ui raised segment">
    <%= message.content%><br>
    <% if message.attachment_data %>
        <span><%= link_to "Download #{message.file_name}", message.attachment_data, download: "" %> <i class="download icon"></i></span>
    <%end%>
   
    <%if current_user(session) == message.user%>
        <%= link_to "Delete", "/chat_rooms/#{@chat_room.id}/messages/#{message.id}", method: :delete %>
    <% end %>
 </div>
  <small class = "ui right aligned">By <%= message.user.name %> on <%= message.created_at.strftime("%A, %d %B at %k:%M %P")%></small>
```

	 ```       
					[...]
				
        <div class = "ui container">
				

	       <%=form_for @message ,url: "/chat_rooms/#{@chat_room.id}/messages", multipart: true do |f|%>
				 
            <div >
 
          <%= f.text_field :content, placeholder: "Enter a message"%>
		

			
				<label for = "message_attachment" ><i class="cloud upload icon"></i> Upload<label>
				
				<%=f.file_field :attachment, class: "hidden menu"%>
				
		  </div>
							<%= f.submit "Send message",class: "hidden menu" %>
					<%end%>
		```			

</div>	 

<p>You do not have to reproduce exactly that html in you code, but I thought that some elements did not have to appear on the page. Thus I used class = "hidden menu" wich is a class defined in my css code. "hidden menu" will set the display of any element to be none.</p>
<p>With that being said, we still need a way to submit this form and handle it without the intervention of any controller. Yes this is doable. We are gonna use some jQuery!</p>

```
    $(document).ready(function() {
    $('#new_message').submit(function(e) {
    var $this, textarea;
    $this = $(this);
    textarea = $this.find('#message_content');
    if ($.trim(textarea.val()).length > 1  ) {
    if ($("input#message_attachment").get(0).files[0]){
    
        reader = new FileReader() ;
        file_name = $("input#message_attachment").get(0).files[0].name;
    
        reader.addEventListener("loadend", function() {
    
        App.global_chat.send_message(textarea.val(), $("#messages").data('chat-room-id'),reader.result,file_name);
    });
        reader.readAsDataURL($("input#message_attachment").get(0).files[0]);
        

    }else {
        App.global_chat.send_message(textarea.val(), $("#messages").data('chat-room-id'));
    }

    e.preventDefault();
        return false;
    }
            

    });
    });
		```
<p>App.global_chat has been defined earlier in  app/javascripts/channels/chat.js<br> This file is basically saying that upon submission of the form, send some data to the method "send_message"</p>
<p>Almost there! before we get to the part where we can create a message, we need to make the user available to action cable. In fact, helpers methods such as current_user and session are not available in action cable. Some people recommend using devise to make the user available. <a href = "https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/" >action cable with devise</a>. But here is the thing, when you read a blog to add new features to your app, you need to find a way to adapt the code on the blog to your app not the other way around. <br>I had already set up my authentication from scratch and I did not want any major change. So I found a customized way to make a current_user available for action cable. </p>
<p>So wherever you are creating a session[:user_id], you can pretty much add the following code <code>            cookies.encrypted[:user_id] = @user.id
</code></p>
<p>We need to make sure that action cable use the current_user method. </p>
<li>In your app/channels/application_cable/connection.rb, have the following code: </li>
                module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user
```
    def connect
   
      self.current_user = find_verified_user
      logger.add_tags 'ActionCable', current_user.email
    end
    private
    def find_verified_user
      if  cookies.encrypted[:user_id]
         User.find_by(id: cookies.encrypted[:user_id])
      else
        reject_unauthorized_connection
      end
    end
  end
end
```

<p>Now we are ready to create our message.</p>
<li>In your file app/channels/chat_channel.rb</li>
```
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_#{params['chat_room_id']}_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
  def send_message(data)
   
    if data["file"] && !data["file"].empty?
      # binding.pry
      message = current_user.messages.build(content: data['message'], chat_room_id: data['chat_room_id'])
      message.attachment_data= data['file']
      message.file_name = data['file_name']
      message.save
      # binding.pry
    else
      current_user.messages.create!(content: data['message'], chat_room_id: data['chat_room_id']) 
    end
  end
end
```
<p>Here we are creating our messages. We are also defining its chat_room for and its user</p>
<p>We will need to use broadcast job to make changes in the background </p>
<li>Run the following code in the terminal: `rails g job MessageBroadcast`</li>
<p>Now let's go back to our message model app/models/message</p>
```
[...]
    after_create_commit{
        MessageBroadcastJob.perform_later(self)
    }
```
<p>This macros will send the message to the job wich will render it to our view.</p>

<li>Now let's go to our file: app/jobs/message_broadcast_job.rb</li>
```
class MessageBroadcastJob < ApplicationJob
  queue_as :default

  def perform(message)
     ActionCable.server.broadcast "chat_#{message.chat_room.id}_channel", render_message(message)
        # binding.pry
  end

  private

  def render_message(message)
    ApplicationController.render message
  end

end
```
<p>The fuction perform will send data to the front-end.<br> The following code `ApplicationController.render message`
will render the message with the html already set up in the partial "_message.rb"
</p>
</ul>
 
 <h2>Conclusion</h2>
 <p>This project made me learn so much. I'm so happy to finally see the results after hours and hours of code.</p>
 <h2>Resources</h2>
 <ul>
 <li>
 <a href = "https://www.youtube.com/watch?v=tyNepRO_ERc&t=1368s">Rails 5 ActionCable Talk: Realtime the Rails Way</a></li>
 
  <li><a href = "https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/">Site point blog</a></li>
	<li><a href = "https://www.youtube.com/watch?v=FrRBCkKGK7Y&t=1248s">Your First Action Cable Chat App | Part 1 - The Setup | Ruby On Rails</a></li>


 </ul>



