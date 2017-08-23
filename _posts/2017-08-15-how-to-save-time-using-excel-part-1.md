---
layout: post
title: "How to save time using excel(Part 1): Record Macro - No coding involved"
excerpt: How to do things faster with excel
summary: "This post is written specifically for people who uses excel a lot. There are a lot of tasks which are always repeated again and again when you work with excel. What if you can create a keyboard shortcut to all these repeatable tasks? I will teach you how to create your own one(my repetitive tasks might be different from yours). First part of the series of two is to to teach how to record the repeatable tasks using macros."
categories: Tutorials
tags: [Excel, VBA, Macros]
comments: true
---

This post is written specifically for people who uses excel a lot. There are a lot of tasks which are always repeated again and again when you work with excel. What if you can create a keyboard shortcut to all these repeatable tasks? I will teach you how to create your own one.(my repetitive tasks might be different from yours) First part of the series of two is to to teach how to record the repeatable tasks using macros.

**Enabling the Developer Ribbon**

This is something that comes out since excel 2010 where you cannot see the record macro(which is the means of us creating the automation). This is available in the Developer Ribbon only. So to get that out, you need to go to File tab->Options->Customize Ribbon->Main Tabs->check the Developer-> Click OK.
<img src="{{ site.baseurl }}/images/dev_ribbon.jpg" alt="">

**Start recording macro**

Go to the Developer Ribbon and click Record Macro.<br>
<img src="{{ site.baseurl }}/images/record_macro_way.jpg" alt="">
<a name = "keyboardshortcut"></a>
You will see the below window. Key in your macro name(optional unless you want some means to keep track of the names of your shortcut, if not keep as default). 
<img src="{{ site.baseurl }}/images/macro1.jpg" alt="">

Click on the empty box below the shortcut key word to move the cursor there. Type the letter that you want as a shortcut key to run this macro (inputting k will mean this macro will be activated by Ctl+k). If you press the Shift key while pressing the letter, this will add a Shift to shortcut (inputting k while holding down Shift means Ctl+Shift+k).<br> 
<img src="{{ site.baseurl }}/images/macro2.jpg" alt="">

I recommend to add a Shift key to your shortcut as a lot of the default windows keyboard shortcuts are with Ctl only (e.g. Ctl+z will be undo Ctl+v is paste). Overriding them with macro activation might cause confusion to you especially if you use the keyboard shortcut often.


Most important part of this is to select where to store macro in. Select Personal Macro Workbook. This will allow you to reuse all the macros you have created in all your excel spreadsheet.<br>
<img src="{{ site.baseurl }}/images/macro3.jpg" alt="">

Then click Ok to start recording.

**Save your recording**

Now you are on your own to create. For example, what I will do is to simply change the font size  from 11 to 12. I will do it via Font section in Home Ribbon. Once done, go to the Developer Ribbon and click on Stop Recording. Voila, your first automation is done!<br>
<img src="{{ site.baseurl }}/images/stoprecording.jpg" alt="">

Test out the macro. Like for my case i have a macro created with Ctl+Shift+k to change font to size 12. So i can set font of a particular cell or group of cell to 8 then press Ctl+Shift+k and hey presto it changed to size 12. 

Now you need to save the Personal Macro Workbook so that you can reuse the recorded macros over and over even if you reopen another spreadsheet. To do this, save all your excel spreadsheet and close the excel application, you will be prompt with the below.<br>
<img src="{{ site.baseurl }}/images/savemacrowb.jpg" alt="">

Click on Save and you are done. Now your recorded macro will be available for use in excel with your assigned keyboard shortcut.

The 2nd part of this is more technical as it would require some coding to be done. But nothing too difficult as VBA(excel coding language) can be written in a very layman manner and in addition the coding will be very minimal. But there will be much more things you can do reading the [next post]({{ site.baseurl }}{% post_url 2017-08-16-how-to-save-time-using-excel-part-2 %})(some things cannot be recorded but can be amended from what macro has recorded and modified via some code).