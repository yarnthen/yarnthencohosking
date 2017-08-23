---
layout: post
title: "Create an email template for your SharePoint List using SharePoint Designer"
excerpt: "This post teaches you how to use SharePoint Designer to create an Email Template for use in your SharePoint List"
summary: "This post teaches you how to use SharePoint Designer to create an Email Template for use in your SharePoint List. You could use this for purposes as simple as just a email notification or some actions(e.g approval, feedback) on your SharePoint List Item"
categories: [Tutorials]
tags: [SharePoint]
comments: true
---

In my [previous post]({{ site.baseurl }}{% post_url 2017-08-18-document-expiry-reminder-system-in-sharepoint %}) I have taught about how to create a reminder system for your SharePoint Document Library and that reminder is via email notification. Now I am going to teach how to create an email template for this. The email template could something as simple as just mentioning that the document is nearing expiration or more complex to the point of adding links to action triggers in the email itself


### Orientation of the Email Message Builder<a name = "orientation"></a>
When you try to add an email action to your SharePoint Workflow and click on "these users", you will prompt with the below Email Message Builder. <br>
<span style="color:red">The ones pointed out in Red are for use in the Subject field.</span><br>
1. This is a string builder where you can key in your text(free text) you want to have in your email subject.
2. This is the lookup string builder where you can select the appropriate lookup value you want to include in your subject. But by using this will mean you cannot add other free text in the Subject. I suggest to use the string builder because it also has a lookup string builder where you can incorporate both free text and lookup value in the subject.

<span style="color:blue">The ones pointed out in Blue are for use in the email content field.</span>
3. This is to add hyperlink to your email content<br>
4. This is the lookup string builder as like in 2 but in this case, it does not stop the adding of free text to the email content. The email content field is still a free text field where you can use to add text you want and also add hyperlinks and lookup values.
<img src="{{ site.baseurl }}/images/orientationemail.jpg" alt="">

### Building the Subject of Email

Subject of the Email is probably the most important element of the email template because what is written there will probably determine whether this email is going to be read or goes straight to the deleted items. I suggest to keep the subject short and straight to the point because if the subject is too long, it might be truncated in view on the email application depending on the settings the user has.<br>
For example an email template to remind of 2 months expiration for my document expiry reminder system, I have the subject simply as "Document Expiry in Two Months Time:" Followed by a Lookup String to point to the Document Name. Then for One Month i will put "Document Expiry in One Months Time:" and so forth. I kept it as generic as possible so that people can gather it easily as an email rule rather than just kept it in the almighty Inbox.<br>
Select the string builder of the email subject and key in "Document Expiry in Two Months Time:". Then click on the Add or Change Lookup. Select Data Source as Current Item(to indicate the current item that trigger the workflow which in this case is the Document Entry in this Document Expiry Reminder System) and Field for Source to select Name.(so that it will show the name of the Document)
<img src="{{ site.baseurl }}/images/emailsubjectbuilder.jpg" alt="">


### Building the Email Content

Next part will be to create Email Content in the template. This is a free text field where you can also add in hyperlinks that could link to action triggers(build this with string builder which is also available in hyperlinks). An example that I have (red indicates created from Lookup string builder):
Link to edit the actual item in the list
<span style="color:red">[%Current Item:Encoded Absolute URL %]</span>/../EditForm.aspx?ID=<span style="color:red">[%Current Item:ID%]</span>

### Add the email addresses

The last part will be to add the email addresses to send to. You can add people to the "To" field and "CC" field. To add an email, click on the address book symbol next to the To or CC field. In my case, the people I am going to email to are already indicated in my list so I will select "Workflow Lookup for a User..." and select Current Item: Document Owner as that is specified as a user name field in my list and SharePoint will be able to resolve it to email address(es). Alternatively you can key in the email address manually.
<img src="{{ site.baseurl }}/images/emailuser.jpg" alt="">

Below is one of my email template for 2 months expiration. You can enhance it further exploring what about information you can put in the email by playing around with the lookup string builder.
<img src="{{ site.baseurl }}/images/finalemail.jpg" alt="">

### Conclusion

This brings the end to the tutorial. This is more of a recap of what I have done at my previous workplace which I failed to document it down earlier. Now I have tried to recreate it via a VM but could not get the email sending to work, will update this with corrected and more examples once i get that to work. Let me know if you noticed some discrepancies, I will get it amended as well.