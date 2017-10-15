---
layout: misc
title: Contact
---

This theme is completely free and open source software. You may use it however you want, as it is distributed under the [MIT License](http://choosealicense.com/licenses/mit/). If you are having any problems, any questions or suggestions, feel free to [tweet at me](https://twitter.com/intent/tweet?text=My%question%about%Millennial%is:%&amp;via=paululele), or [file a GitHub issue](https://github.com/lenpaul/Millennial/issues/new).

<form id="contactform" method="POST">
    <label for="name">Your name</label><br>
    <input type="text" name="name" placeholder="Name" required><br><br>
    <label for="_replyto">Your email</label><br>
    <input type="email" name="_replyto" placeholder="example@domain.com" required><br><br>
    <label for="message">Your message</label><br>
    <textarea name="message" rows="5" cols="50" placeholder="Message" required></textarea>
    <input type="hidden" name="_subject" value="[throughaglass.io] new contact!" /><br><br>
    <input type="text" name="_gotcha" style="display:none" />
    <input type="submit" value="Send">
</form>
<script>
    var contactform =  document.getElementById('contactform');
    contactform.setAttribute('action', 'https://formspree.io/' + 'vitor191291' + '@' + 'gmail' + '.' + 'com');
</script>
