---
layout: post
title: "Unique Ticket Number Generator for your SharePoint List"
excerpt: "This post teaches you how to use SharePoint Designer Workflow to create a ticket number for your individual SharePoint list record"
summary: "This post teaches you how to use SharePoint Designer Workflow to create a ticket number for your individual SharePoint list record"
categories: Tutorials
tags: [SharePoint]
comments: true
---

<style>
  .bordered {
    border: 1px solid black;
  }
</style>

I want a way where it is possible to identify individual entries to SharePoint Custom list easily. This will require a unique ID for each entry in the SharePoint List.

One easy way to do it is just to use the ID field that is a default field in a SharePoint List which is a basically the sequence number based on the order of creation in a list. But this unique ID is not be easily relatable, as it is difficult for someone to remember this unique ID especially if there are a lot of entries in the list. 

I decided to create a unique id based on the sequence of the entry created per day prefixed by the date. (e.g the fist entry created on 21st Jan 2017 will be 201701210001 and the first entry created on 22nd Jan 2017 will be 201701220001)

## Detailed steps

Create two SharePoint Custom Lists(shown below are the necessary columns)

<keyword>Sharepoint List 1: Ticket System</keyword> 
This is the main list where the unique ticket number is shown. You can customise it by add any additional fields you like.

1.  **Title (Multi Line Text):** This is the field where you key in the title of your request

2. **DateCreated(Calculated (calculation based on other columns)):** This is to format the default Created field which have date and time format to YYYYMMDD format
<img src="{{ site.baseurl }}/images/datecreated.jpg" alt="">

3. **Counter (Number):** This is to store the counter which is the sequence that the ticket is created of the day. Set this field as hidden as it is not to be availalble for input (Go to advanced settings->Allow Managment of Content Type Yes->Go to the Item Content type and set the Counter field as hidden)

4. **CalculatedCounter (Calculated (calculation based on other columns)):** This is to enhanced the counter with prefix 0s(zeros) in front so as to create a possibly of more numbers(eg. first ticket of the day will have counter 0001 so the last ticket of the day can run to 9999). You can also add some more prefix or suffix on your preference to differentiate the tickets (e.g. you can add INC in front to create a ticket number for incident ticket)
<img src="{{ site.baseurl }}/images/calculatedcounter.jpg" alt="">

5. **Ticket Number (Calculated (calculation based on other columns)):** This is to store the unique ticket ID by combining the DateCreated and CalculatedCounter
<img src="{{ site.baseurl }}/images/ticketnumber.jpg" alt="">

You can also add other fields that you want for the type of ticketing system you are created (e.g. helpdesk ticket, incident ticket)

<keyword>Sharepoint List 2: ITRS </keyword>
For reference purpose to create the unique ID, this list will only have a single entry at all times

Fields | Type | Explanation
------------- | ------------- | -------------
Title | Single Line of Text | This is a default column that we will use to store a date in YYYYMMDD format for reference so if a ticket is created on the next available day, it will help to reset the number counter back to 1. If a new ticket is created on the same day, it will increment the number counter by 1
Counter | Number | This is the counter which is the sequence number of a ticket created per day


Create the first and only entry in the ITRS SharePoint List. You just need to add a first entry by keying in something to the title field of the first record(Don't need to worry about what is keyed in as the SharePoint Workflow will work around it later). Do not key in anything in the counter field.  
<img src="{{ site.baseurl }}/images/ITRS.jpg" alt="">
<br>

### SharePoint Workflow
Now its time to create the workflow.
Go to Library->Workflow Settings dropdown arrow->Create a Workflow in SharePoint Designer<br>
<img src="{{ site.baseurl }}/images/createworkflow.jpg" alt="">

Name your Workflow and set Platform Type to 'SharePoint 2010 Workflow' .Note I used a SharePoint 2010 Workflow for this despite using SharePoint 2013 as the 2010 Workflow is good enough to get this done. So you can do this in both SharePoint 2010 and SharePoint 2013.


Check for the calculated field DateCreated against the title in ITRS list(note we look for the entry we want in ITRS by finding the entry with ID 1, as there will only be one entry in ITRS permanently, the ID will always be 1. Do take note to never delete the entry in ITRS)

If they don't tally, that means it is the first ticket created on that day and thus counter will be 1
<div class="bordered"><img src="{{ site.baseurl }}/images/SPWorkflow1.jpg" alt=""></div>


Update the ITRS entry by changing the title to the value in DateCreated and counter in ITRS to 1
<div class="bordered"><img src="{{ site.baseurl }}/images/SPWorkflow3.jpg" alt=""></div>


If they tally, that means the counter in Ticket System will need to increment by one. So create a variable(calc) to store the incremented number
<div class="bordered"><img src="{{ site.baseurl }}/images/SPWorkflow2.jpg" alt=""></div>

Write the variable(calc) value to the counter in the current item of Ticket System
<div class="bordered"><img src="{{ site.baseurl }}/images/SPWorkflow4.jpg" alt=""></div>

Set the workflow to run automatically when an item is created
<div class="bordered"><img src="{{ site.baseurl }}/images/workflowsetting.jpg" alt=""></div>

The unique ticket number will then be created as it is a calculated value based on the DateCreated field and CalculatedCounter field(which in turn is created from Counter field) 
<div class="bordered"><img src="{{ site.baseurl }}/images/autoticketid.jpg" alt=""></div>






