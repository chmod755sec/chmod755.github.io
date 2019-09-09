## General Info
If you want to do a writeup, you'll need to message me (JaguarZZ) in discord, so that I can make you a contributor.. Once you're a contributor, you'll be able to upload files.

The first thing I'm going to talk about is how to do a pull request, so that you can properly link your images. If you do your writeups, and somehow have the img's in that writeup stored in a folder, then skip to "Imgs pull req", but if your're like me, and take snippets, then paste it into the writeup (editor), then follow the below.

## Extracting text & imgs
From what I've noticed, the easiest way of taking a pasted photo from an editor, and turning it into a file, is to first convert your writeup to a PDF, then use a online extractor for the images & the text.. The online img extractor will name the imgs from 0-x amount, so it takes care of where your img were at in the writeup. Here are the links to extract the imgs & text: text) https://pdftotext.com/, imgs) https://www.extractpdf.com/.

## Imgs pull req
Now that you've got the imgs in a folder, and the text in a file, we can move onto doing the pull req. The first thing you'll need to do, is go to the assets folder in the repos, then from there click on the folder of the box you're working on.. Click the “upload” button, and copy the imgs you'll be using into the folder. (if the box your working on isn't there, then create it, and do the above)

## Linking imgs to text & formating
Next thing to do is create a file in the “_posts” folder, with the date and box name: “year-month-data-boxname.md”. There will be some other files in that folder, so you can just look at them, and you should understand the naming scheme. Next thing to do is copy the text from your writeup and insert it into the file you've just created. Now the final thing to do, is to link your imgs & do some formating. To link your images, you'll just need to use this little snippet of code, with a minor change: 
```
![Image](/assets/BOXNAME/NAME_OF_THE_IMGS_000.jpg)
```
The naming scheme of the image should look like this: “Writeup_for_Bastion-000.jpg”--thats depending on the name of the PDF file. You should be able to insert that snippet of code where ever you had a image, and with how the online extractor extracts the images, it should all be in order from 000-0xx.

## Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)

## One last thing
At the top of your writeup, please insert the following:
```
---
layout: post
title:  "boxname writeup by username"
---
```
The above must be at the top of the writeup
