---
layout: page
title: Contact Me
description: Have questions? I have answers.
background: '/img/bg-contact.jpg'
form: true
---

<p>Want to get in touch? Fill out the form below to send me a message and I will get back to you as soon as possible!</p>
<!-- <form name="sentMessage" id="contactForm" novalidate> -->
<form id="form1" action="https://atechcontactformserver.azurewebsites.net/Message/Submit" method="post" enctype="application/x-www-form-urlencoded">
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Name</label>
      <input type="text" class="form-control" placeholder="Name" name="name" required data-validation-required-message="Please enter your name.">
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Email Address</label>
      <input type="email" class="form-control" placeholder="Email Address" name="email" required data-validation-required-message="Please enter your email address.">
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Message</label>
      <textarea rows="5" class="form-control" placeholder="Message" name="message" required data-validation-required-message="Please enter a message."></textarea>
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <br>
  <div id="success"></div>
  <div class="form-group">
    <button type="submit" class="btn btn-primary">Send</button>
    <!-- <div><input type="submit" value="Submit" class="btn btn-primary"></div> -->
  </div>
</form>

<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<script>
$('#form1').on('submit', (e) => {
alert('Hello, World');
});
</script>
