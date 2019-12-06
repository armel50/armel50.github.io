---
layout: post
title:      "Deploy a Single Page Application on Heroku"
date:       2019-12-06 21:38:14 +0000
permalink:  deploy_a_single_page_application_on_heroku
---


<h2>Introduction</h2>
<p>Money coach is an application designed to help users manage their money better. With such an application, a user can keep track of how much money is spent on certain products, set goals with deadlines, and even interact with the stocks market. <br>As most popular web applications, Money Coach uses features such as javaScript, Chart.js, and Ajax to keep the user's experience more enjoyable.<br> Unfortunetaly, deploying such an application on heroku seems much harder than it looks. Thus throughout the rest of this blog, we will uncover one of the best tricks to overcome any obstacle which could prevent the deployment of such applications on Heroku. </p>
<h2>All you need to know</h2>
<p>Assuming that you're already done building your application and you now ready to deploy it; the following trick will help you deploy your app on Heroku.<br> Deploying just a Html file on heroku is a bit tricky. In fact, Heroku does not recognize Html as one of its default buildpacks. <br> Fortunately for us, it is very easy to overcome such an obstacle. <br>
<ul>
<li>Head to the root directory of your project that contains index.html (the main HTML page).</li><br>
<li>In this directory, run touch composer.json to create a file: composer.json.</li><br>
<li>Within the file, add the following line: {}</li><br>
<li>In the same directory, run touch index.php to create a file: index.php.</li><br>
<li>Within the file, add the following line: <?php include_once("index.html"); ?></li><br>
<li>Now, commit and push these two new files to your repository. You can also use the Heroku command git push heroku master .</li><br>
</ul>
... And voila, you can now brag about your application at your friend's party!
</p>

