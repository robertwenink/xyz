---
title: "About this website"
subtitle: ""
date: 2023-08-01T17:37:43+02:00
lastmod: 2023-08-01T17:37:43+02:00
draft: false
description: ""

tags: []
categories: []

featuredImage: ""
featuredImagePreview: ""
---
<script>
function adjustClearProperty() {
  var ul = document.querySelector("ul");
  var image = document.querySelector(".wrap-around img");
  var text = document.querySelector("#fronttext");

  var ulHeight = ul.offsetHeight;
  var imageHeight = image.offsetHeight;  
  var textHeight = text.offsetHeight;
  var diff = imageHeight - textHeight;
  console.log(ulHeight, diff)

  if (diff < ulHeight * 1 / 2) {
    ul.style.clear = "left";
  } else if (diff > ulHeight) {
    ul.style.clear = "none";
  }
}

document.addEventListener("DOMContentLoaded", function() {
  adjustClearProperty(); // Initial adjustment on load

  window.addEventListener("resize", function() {
    adjustClearProperty(); // Adjust on window resize
  });
});
</script>

## About this website
<!-- alt="Me on top of a mountain/hill in the black forest, Germany." caption="Me on top of a mountain/hill in the black forest, Germany." -->
<div class="wrap-around">
{{< image src="pf.tif" width="45%" alt="Me on top of a mountain/hill in the black forest, Germany." >}}

<div id="fronttext">
I specialise in the fusion between the fields of Computer Science and Maritime Engineering, possessing a master's degree in both. 

In 2022, I started freelancing with a most self-explanatory company name:
</br></br>
<i style="white-space:nowrap;">Robert Wenink - </i>
<i style="white-space:nowrap;">Maritime Engineering &</i>
<i style="white-space:nowrap;">Computer Science.</i> 
</div>
<!-- padding om de ul binnen de div te houden -->
<div id="ul" style="padding:1px">

- To read more about me, my skills and curriculum vitae, see the [About](/about/) section.
- A project portfolio is found under [Projects](/projects/).
- A (personal) tech blog is maintained in [Blog](/posts/).

Want to hire me, or have any suggestions? 
<span></br>Contact me: <robertwenink@gmail.com>. </span>

</div>
</div>

<hr>

## Projects