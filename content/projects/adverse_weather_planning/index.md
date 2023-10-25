---
title: "Cadeler A/S: Adverse Weather Planning tool"
subtitle: ""
date: 2023-10-23T17:55:54+02:00
draft: false
description: "" # important for proper SEO!!
summary: "" 

tags: []
categories: []
series: ""
weight: 0

featuredImage: "pacific-orca-sept-2012-large_blurred.jpg"
featuredImagePreview: "preview_with_logo5.png"
# featuredImagePreview: "preview.png"
# 800x240 rendered, dus moet width x 0.3 is height

math: false
lightgallery: false # only set true if linked images in post
toc: false
---

[Cadeler A/S](https://www.cadeler.com/) has been so kind to assign me my first project. The goal was to streamline their planning- and reporting process related to adverse weather conditions during operations with their jack-up vessels, used in the construction and commissioning of offshore windfarms. 
<!--more-->

The final product has the following top-level functionality:

<style>
#pdf_img {
    margin: 0 7.5px;
}
#mermaid_mobile {
    display:none;
}
@media only screen and (max-width: 960px) {
    #mermaid_mobile {
        display:inherit;
    }
    #mermaid_desktop {
        display:none;
    }
}
</style>

<div id="mermaid_desktop">
{{< mermaid >}}
graph LR;
    A(Graphical User Interface) --> | User input | C(Weather window calculations <br/>according to DNV regulations)
    C --> report(Automated report generation)
    report --> word(<img src='Microsoft_Office_Word.png' width='55px' height='50px' > )
    report --> pdf(<img src='Adobe_PDF-removebg.png' width='40px' height='50px' id='pdf_img' /> )
{{< /mermaid >}}
</div>

<div id="mermaid_mobile">
{{< mermaid >}}
graph TD;
    A(Graphical User Interface) --> | User input | C(Weather window calculations according to DNV regulations)
    C --> report(Automated report generation)
    report --> word(<img src='Microsoft_Office_Word.png' width='55px' height='50px' > )
    report --> pdf(<img src='Adobe_PDF-removebg.png' width='40px' height='50px' id='pdf_img' /> )
{{< /mermaid >}}
</div>

The GUI has the following extra functionalities:

- Complete input verification. The tool actively directs the user to faulty or incomplete inputs, also providing clear textual instructions through message boxes.
- Tooltips per input field, making the tool largely self-explained up to the level of the technical specifics.
- Mechanics to (automatically) save and load the complete state of GUI.
- The possibility to save (and delete) standard scenarios, to be straighforwardly inserted from within the GUI for an increased efficiency and ease of the workflow.
- Mechanics to share the user-defined save files and standard scnearios with other users. Likewise, the complete tool can be exported and automatically installed for receiving users.
- The tool comes with a manual for users and for developers.
<br/>
<br/>

{{< image src="AWPtool_projectconstants.png" linked=false caption="The 'Project constants' tab where general project constants, options, and inputs specific to report generation can be supplied. For privacy reasons, examples of the rendered outputs and tabs to provide the weather window inputs and are not shown in this post." >}}

The GUI is written in python using built-in Tkinter and [CustomTkinter](https://github.com/TomSchimansky/CustomTkinter), to obtain a clean and modern look. Any future GUI projects will however be written using [PySide](https://www.qt.io/qt-for-python), the LGPL-licensed official python binding of Qt. The tool's backend and GUI-integration are written using well-known python libraries such as numpy, matplotlib and PIL. Report generation is based on a template file, changed and imputed with the tool's inputs and outputs using [python-docx](https://github.com/python-openxml/python-docx). The tool is exported as a single-file executable using [PyInstaller](https://pyinstaller.org/en/stable/).





