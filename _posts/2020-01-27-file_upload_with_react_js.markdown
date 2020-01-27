---
layout: post
title:      "File Upload With React Js"
date:       2020-01-27 20:05:12 +0000
permalink:  file_upload_with_react_js
---

<h2>Introduction</h2>
Nowadays, every popular website such as Facebook, Instagram, Twitter gives the option to upload files.
While this may sound easy, saving those files to a database can be tedious and painful at the same time. In this blog, I will share how to uploads any files (primarly pictures) to a database. 

<h2>Code in React </h2> 
```
import React, { Component } from 'react'
import {connect} from 'react-redux'
import add_place from '../../actions/send_image'

 class New extends Component {
    constructor(){
        super()
        this.state = {
          
            image: ' '

        }
        this.div_ref = React.createRef()

    }


  
    handleFileChange = event => {
        
        event.persist()
        this.setState(state => ({image: event.target.files[0] })  )
    }



    handleSubmit = (event) => {
     
        this.props.send_image(this.state)

        event.preventDefault()

    }

    render() {
        return (
            <div>
                <h1>This is to add an image</h1>
                <form onSubmit={this.handleSubmit} >
                    

                

                    <div class='images' ref={this.div_ref}>
                        <label>Images</label>
                        <input type= 'file'  name='image' onChange={this.handleFileChange} accept="image/x-png,image/gif,image/jpeg" />
                    </div>


                    <button>Add Image </button>
                </form>

            </div>
        )
    }
}

const mapToDispatch = dispatch  => ({
    send_image: image => dispatch(send_image(image))
})
export default connect(null, mapToDispatch)(New)
``` 

The code above is a simple implementation of a file input. we first  write the input tag, and add an event handler to take care of it's changes. This is a pretty straightforward process therefore the challenge does not reside in this part of the code.
<h2>The Backend</h2>
In the backend, I set up a Ruby Api from scratch that I use to fetch my data. However this process should not be too different with any other Api. 

<h3>The Table</h3>
now that we have a Ruby Api, we can set up a simple image table.
```
class CreateImages < ActiveRecord::Migration[5.2]
  def change
    create_table :images do |t|
      t.string :url
      t.integer :place_id

      t.timestamps
    end
  end
end
```
We can give any attribute we want to this table, but we have to make sure that we have an attribute to store an url. 
<h3>Active Storage</h3>
After those simple steps, let's make sure we activate <a herf=https://edgeguides.rubyonrails.org/active_storage_overview.html''>Active Storage</a>. This is a built-in feature of rails that will let us store files on local storage and cloud services(amzon s3, Google Cloud).
run this code in your terminal: <code>rails active_storage:install</code>

run <code>rails db:migrate</code> after those steps.
We still have to define how to send the file to our controller.
<h2>Data Formating</h2>
If you still remember, when we submit the form our images will be submited with a function called <code>send_image()</code>. Let's now take a look at this function.
```
export default function send_image(data){
    return dispatch =>{
    
          const fd = new FormData()
      
            fd.append('image',data.image)

       
        const params = {
            method: 'POST',
      
           body:fd
        }
        fetch('http://localhost:3000/places/',params)
        .then(resp => resp.json())
        .then(json => {
                           dispatch({type:'ADD_IMAGE', places: json.places})
         })

  
}
```
If you're familiar with the fetch Api, you might be used to definning headers in your request. This is not going to serve you at all in this case. Also, note that instead of using <code>JSON.stringify</code> I used <a href='https://developer.mozilla.org/en-US/docs/Web/API/FormData/Using_FormData_Objects'>FormData to send the request.</a>. Using FomData will make it easier to get files to the controller.
Now let's see what our controller looks like.
<h2>Controller</h2>
```
class ImagesController < ApplicationController
    

    def create 
                
                image = Image.new()
								image.file = params.require(:image)
                if image.file.attached? 
                    image.url = url_for(image.file)
                    image.save

                end
           
        end
        render json: {image: image} 
    end
end
```
Here, to be able to access the image later on we use some 'rails magic' <a href ='https://guides.rubyonrails.org/routing.html'>url_for</a>.  url_for will give us a url that we can store in the database.
<h2>Conclusion</h2>
That's it for this tutorial. Thanks again for readind and happy hacking!
