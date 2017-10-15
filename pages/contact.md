---
layout: misc
title: Contact
---

You can find me at [Linkedin](https://linkedin.com/in/joao-osilva), [GitHub](https://github.com/joao-osilva) or [Email](mailto:vitor191291@gmail.com).

If you have any questions or suggestions, feel free to send me a message using the form below.

<div class="form-style">
<form id="contactform" method="POST">
    <label for="name">Your name</label><br>
    <input type="text" name="name" placeholder="Name" required><br>
    <label for="_replyto">Your email</label><br>
    <input type="email" name="_replyto" placeholder="example@domain.com" required><br>
    <label for="message">Your message</label><br>
    <textarea name="message" rows="5" cols="50" placeholder="Message" required></textarea>
    <input type="hidden" name="_subject" value="[throughaglass.io] new contact!" /><br>
    <input type="text" name="_gotcha" style="display:none" />
<label>
    <input type="submit" value="Send">
</label>
</form>
</div>
<script>
    var contactform =  document.getElementById('contactform');
    contactform.setAttribute('action', 'https://formspree.io/' + 'vitor191291' + '@' + 'gmail' + '.' + 'com');
</script>
