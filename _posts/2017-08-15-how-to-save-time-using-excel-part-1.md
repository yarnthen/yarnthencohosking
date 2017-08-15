---
layout: post
title: "How to save time using excel(Part 1): Record Macro - No coding involved"
excerpt: How to do things faster with excel
summary: "This post is written specifically for people who uses excel a lot. There are a lot of tasks which are always repeated again and again when you work with excel. What if you can create a keyboard shortcut to all these repeatable tasks? I will teach you how to create your own one(my repetive tasks might be different from yours)"
categories: Tutorials
tags: [Excel, VBA, Macros]
comments: true
---

This post is written specifically for people who uses excel a lot. There are a lot of tasks which are always repeated again and again when you work with excel. What if you can create a keyboard shortcut to all these repeatable tasks? I will teach you how to create your own one(my repetive tasks might be different from yours)

**Enabling the Developer Ribbon**

This is something that comes out since excel 2010 where you cannot see the record macro(which is the means of us creating the automation). This is available in the Developer Ribbon only. So to get that out, you need to go to File tab->Options->Customize Ribbon->Main Tabs->check the Developer-> Click OK.
<img src="{{ site.urlimg }}/dev_ribbon.jpg" alt="">

**Start recording macro**

Go to the Developer Ribbon and click Record Macro.<br>
<img src="{{ site.urlimg }}/record_macro_way.jpg" alt="">

You will see the below window. Key in your macro name(optional unless you want some means to keep track of the names of your shortcut, if not keep as default). 
<img src="{{ site.urlimg }}/macro1.jpg" alt="">

Click on the empty box below the shortcut key word to move the cursor there. Type the letter that you want as a shortcut key to run this macro (inputting k will mean this macro will be activated by Ctl+k). If you press the Shift key while pressing the letter, this will add a Shift to shortcut (inputting k while holding down Shift means Ctl+Shift+k).<br> 
<img src="{{ site.urlimg }}/macro2.jpg" alt="">

I recommend to add a Shift key to your shortcut as a lot of the default windows keyboard shortcuts are with Ctl only (e.g. Ctl+z will be undo Ctl+v is paste). Overriding them with macro activation might cause confusion to you especially if you use the keyboard shortcut often.


Most important part of this is to select where to store macro in. Select Personal Macro Workbook. This will allow you to reuse all the macros you have created in all your excel spreadsheet.<br>
<img src="{{ site.urlimg }}/macro3.jpg" alt="">

Then click Ok to start recording.

**Save your recording**

Now you are on your own to create. For example, what I will do is to simply change the font size  from 11 to 12. I will do it via Font section in Home Ribbon. Once done, go to the Developer Ribbon and click on Stop Recording. Voila, your first automation is done!
<img src="{{ site.urlimg }}/stoprecording.jpg" alt="">

Test out the macro. Like for my case i have a macro created with Ctl+Shift+k to change font to size 12. So i can set font of a particular cell or group of cell to 8 then press Ctl+Shift+k and hey presto it changed to size 12. 

The 2nd part of this is more technical as it would require some coding to be done. But nothing too difficult as VBA(excel coding language) can be written in a very layman manner and in addition the coding will be very minimal. But there will be much more things you can do reading the next post (some things cannot be recorded but can be amended from what macro has recorded and modified via some code).