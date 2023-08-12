---
layout: page
title: About me
description: About me
published: true
permalink: about
icon: fa-solid fa-address-card
order: 10
---

My name is Michele and I’m a Principal Engineer at Amadeus.

You are reading a low frequency blog about programming, technologies, software architecture, big data, languages. I am usually busy with C++, Spark, Scala, Python, not enough cloud and perhaps too much software architecture.

You can contact me in English, French, Italian or Spanish, preferably [via email](javascript:sendEmail();).

The code for this blog is on github.

All views expressed here are my own.

Thanks for reading.

<!-- --------------------------------------------------------------------------------------------------- -->
<!-- Some code to avoid putting the email in clear, very much inspired by the trick used by Chirpy Theme -->
<!-- --------------------------------------------------------------------------------------------------- -->
{% assign email = site.social.email | split: '@' %}
<script>
  // JavaScript code to run when the link is clicked
  function sendEmail() {
    var emailParts = ['{{ email[0] }}', '{{ email[1] }}'];
    var emailAddress = emailParts.join('@');
    location.href = 'mailto:' + emailAddress;
  }
</script>