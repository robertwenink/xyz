---
title: "Tkinter: how export a widget as image"
subtitle: ""
date: 2023-10-03T17:05:25+02:00
draft: false
description: ""
summary: "" 

tags: [GUI, tkinter]
categories: [programming]
series: ""
weight: 0

featuredImage: ""
featuredImagePreview: ""

---
Tkinter, is a Python standard library and interface to the Tcl/Tk GUI toolkit (Tcl is playfully pronounced as 'tickle').
Depending on the usage, Tkinter provides an easy-to-use and fast library for creating GUIs, and can presently be made modern and beautiful using the [customtkinter](https://github.com/TomSchimansky/CustomTkinter) package.
However, there is no native support to export a rendered tkinter.Frame or tkinter.Toplevel to image format. This post describes a workaround to that problem.

# Capture the widget
To create an image from the rendered GUI widget, we can use the straighforward option of taking a screenshot.

```python
from PIL import Image, ImageGrab

def capture_widget(widget):
    """Take screenshot of the passed widget"""
    
    widget.update()
    widget.focus()

    x0 = widget.winfo_rootx()
    y0 = widget.winfo_rooty()
    x1 = x0 + widget.winfo_width()
    y1 = y0 + widget.winfo_height()

    img = ImageGrab.grab((x0, y0, x1, y1))
    return img
```

<!--more-->
{{< highlight-inline python "widget.update()" >}} makes sure that the widget is updated and rendered completely. Otherwise, the dimensions provided by for instance {{< highlight-inline python "widget.winfo_rootx()" >}} might not be correct. The {{< highlight-inline python "widget.update()" >}} function is blocking and thereby relatively time-consuming. So, if you know the widget has been rendered before, you can omit this step.

If all went well, we now have captured a screenshot of the rendered widget. The resolution of the screenshot is directly dependent on the rendered size of the widget and thereby limited by the display's effective screensize. This limiting screensize often is 1080p and might even include a 125% scaling. The quality of the screenshot is thereby far from the infinite zoom quality of .svg files one might desire. 

To lessen the symptoms of this problem, I apply two image manipulation steps:
- [Resize and interpolate the image](#resize-and-interpolate-the-image)
- [Use an image palette](#reducing-the-image-size-by-using-a-palette) to drastically reduce the image filesize by limiting the number of used colors while retaining quality (in case of schematics or graphs, do not use this for pictures!)

## Resize and interpolate the image
```python
# capture the widget
img = capture_widget(widget)

# resize and interpolate (resample)
scale = 2
img = img.resize((int(img.size[0] * scale), int(img.size[1] * scale)), resample = Image.LANCZOS)
```

Resizing and interpolating the image increases the 'zoomable' quality of the widget screencapture. However, no new real image information is created, and so the quality-improvement achievable through interpolation is capped. From my trial-and-error experience the best balance between increased perceived quality and filesize is found for {{< highlight-inline python "scale=2" >}}.

## Reducing the image size by using a palette
Although the image quality has now improved, this comes at the cost of a {{< highlight-inline python "scale * scale" >}} increase in the number in pixels and thus the filesize. To counter this effect, I convert the image to use a palette, whereby each pixel is represented using a pointer to a 'color palette'. 

```python
# convert img to palette to save file space
img = img.convert('P',
    palette=Image.ADAPTIVE, # Let PIL pick the best fitting palette
    colors=256, # Amount of colors for the ADAPTIVE palette, 256 is default and max 
    )
```

This conversion, for my use case and sytem, reduces the size of 3 images from a total of 2642kb to 871kb at the cost of 0.4s per processed image at 1080p with no to minimal visual degradation of the image quality. Indeed, a 8-bit (up to 256) numeric pointer is 3 times as small as a 24-bit RGB color (typically represented using a hexidecimal number of length 6) per pixel.

The other possible option for pallette, {{< highlight-inline python "pallet=Image.WEB" >}}, is not well-documented in what it does besides not using the colors parameter. The result is both larger in file-size and of lesser quality, with our without dithering applied. I therefore do not recommende using {{< highlight-inline python "pallet=Image.WEB" >}}.

# Use the image in further processing
As a last optional step, when the image is not saved as-is but further used in a processing pipeline, it should be saved to an bytestream that can be used by other tools instead of saving it to a temporary image file.

```python
import io
import img2pdf

# create img bytestream
image_stream  = io.BytesIO()
img.save(image_stream, format='png')

# and for instance save to an A4 pdf page
pdf_byte_arr = io.BytesIO()
layout_fun = img2pdf.get_layout_fun((img2pdf.mm_to_pt(297),img2pdf.mm_to_pt(210)))
pdf_byte_arr.write(img2pdf.convert(img_byte_arr.getvalue(), layout_fun=layout_fun))

# do some more pdf / image magic
... 
```

While saving a schematic image, remember to save the image as .png and not as .jpg! 

The jpg compression is done per block of 8x8 pixels using the information available in that block and so will result in compression artefacts if the block is not composed of one color. These artefacts for instance become apparent in images with large single-color areas containing text or lines, where the text or lines become blurry and the single-color areas non-uniform. 

The png compression on the other hand recognises the repeated pattern of the single-color area and as such compresses the image more dynamically, without artefacts. A not well-known nor well-found fact is that due to this png images can even be of smaller filesize even though they are lossless, when compressing schematic images.