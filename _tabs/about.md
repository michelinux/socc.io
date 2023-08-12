---
layout: page
title: About me
description: About me
published: true
permalink: about
icon: fa-solid fa-address-card
order: 10
math: true
---

My name is Michele Costantino Soccio and I’m a Principal Engineer at [Amadeus](https://amadeus.com/en). You are reading a low frequency blog about programming, technologies, software architecture, big data, languages. I am usually busy with C++, Spark, Scala, Python, not enough cloud and too much software architecture, or the other way around.

If you wish to contact me, do not refrain from [dropping me an email <i class="fa-regular fa-envelope"/>](javascript:sendEmail();) in English, French, Italian or Spanish.

The code for this blog is on [GitHub <i class="fa-brands fa-github"></i>](https://github.com/cotes2020/jekyll-theme-chirpy). All views expressed here are my own.

Thanks for reading,<br/>
Michele.


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