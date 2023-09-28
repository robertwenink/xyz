---
title: "Home" # Robert Wenink - Maritime Engineering & Computer Science
subtitle: ""
date: 2023-08-01T17:37:43+02:00
draft: false
description: ""

tags: []
categories: []

featuredImage: ""
featuredImagePreview: ""
---

<!-- <script>
function adjustClearProperty() {
  var ul = document.querySelector("ul");
  var image = document.querySelector(".wrap-around img");
  var text = document.querySelector("#fronttext");

  var ulHeight = ul.offsetHeight;
  var imageHeight = image.offsetHeight;  
  var textHeight = text.offsetHeight;  
  var companyHeight = document.querySelector("#centered_company");
  var companyHeight = companyHeight === null ? 0 : companyHeight.offsetHeight;
  
  var diff = imageHeight - textHeight - companyHeight;
  console.log(ulHeight, diff)

  if (diff < ulHeight * 1 / 2) {
    ul.style.clear = "left";
  } else if (diff > ulHeight) {
    ul.style.clear = "none";
  } else {
    ul.style.clear = "none";
  }

}

document.addEventListener("DOMContentLoaded", function() {
  adjustClearProperty(); // Initial adjustment on load

  window.addEventListener("resize", function() {
    adjustClearProperty(); // Adjust on window resize
  });
});
</script> -->

# About me and this website

<div class="wrap-around">
  <div id="my-name-is-before">
  
  My name is Robert Wenink. I specialise in the fusion between the fields of Computer Science and Maritime Engineering, possessing a master's degree in both.
  </div>
  <div id="hire-text-before">

  You can hire me for projects related to process automation or code optimisation.
  </div>

{{< image src="images/pf.tif" id="pf_home" alt="Me on top of a mountain/hill in the black forest, Germany."  linked=false >}}

<div id="fronttext">

  <div id="my-name-is-after">

  My name is Robert Wenink. I specialise in the fusion between the fields of Computer Science and Maritime Engineering, possessing a master's degree in both.
  </div>

  <div id="hire-text-after">

  You can hire me for projects related to process automation or code optimisation.
  </div>

</div>
<!-- padding om de ul binnen de div te houden -->
<div id="ul" style="padding:1px;">

- To read more about me, my skills and curriculum vitae, see the [About](/about/) section.
- A project portfolio is found under [Projects](/projects/).
- A (personal) tech blog is maintained in [Posts](/posts/).
</div>

<div id="hire-me">

Want to hire me, or have any suggestions?

<span id="contact-me">Contact me: <robertwenink@gmail.com>.</span>

</div>
</div>

# Projects