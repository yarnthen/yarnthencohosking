---
layout: post
title: "How to automate booking of meeting room(resource mailbox in Outlook): Introduction"
excerpt: This post teaches you how to automate a booking of resource (e.g. Meeting Room) which is via a resource MailBox in Outlook (Exchange)
summary: This post teaches you how to automate a booking of resource (e.g. Meeting Room) which is via a resource MailBox in Outlook (Exchange)
categories: Tutorials
tags: [vbscript, Outlook, Windows, Task Scheduler]
comments: true
---

One of the cool features of an email services is to leverage on mailboxes to manage reservation of resources(e.g. Meeting Rooms, Projectors, equipments). Gone are the days where someone will need to call up an operator to make a reservation of meeting rooms and the operator will need to check if someone has already booked for the timeslot before making a booking on your behalf. Now the resource(e.g. meeting room) will be signify by a Mailbox Calendar where you will need to send a meeting invite via your email application, if that timeslot is free, it will automatically accept your invite. If it isn't, it means the resource has been booked by someone else for that timeslot and will reject the invitation. This results in an automated way of managing reservation of resources.<br> 
But in most organizations, administrators will setup a limit to the advance booking period(mine is 4 weeks) of the resource to prevent resource hoarding (imagine if someone could book a meeting room on a particular day of the week for the next 20 years!) using recurring invites.<br> 
Same thing happened in my workplace where I am tasked to be making the reservation of meeting room for my team meeting. It starts to become a hassle to remember all these as you need to rebook these again and again every week(one month in advance) and in cases where you forget someone else might just book the resource ahead of you. So I decided to automate the booking of meeting room so that there is no way it can be a miss and also kept it away from bothering me. I kept this as a secret, my colleagues were all so impressed by my ability to ensure meeting room booking every week.<br>
Here's how i did it


# Vbscript for execution of booking
I used vbscript to write the code for the execution of the booking. Below is the code i used, note that I adapted the code from [outlookcode.com](http://www.outlookcode.com/codedetail.aspx?id=88) and make minor tweaks to get it to work the way i want. Look at my [next post]({{ site.baseurl }}{% post_url 2017-08-24-how-to-automate-booking-of-meeting-room-resource-mailbox-in-outlook-vbs-code %}) for the explanation of the code. Save the below code to a notepad and save as a vbs file in a location where you will not touch accidently(to prevent accidental triggering). 

{% highlight vbscript %}
Sub SendMeetingRequest()
	Dim objOL   'As Outlook.Application
	Dim objAppt 'As Outlook.AppointmentItem
	Dim MeetingDate

	MeetingDate = DateAdd("d", 28, Date)
	MeetingDate = CDate(MeetingDate & " 11:00:00 AM")

	Set objOL = CreateObject("Outlook.Application")
	Set objAppt = objOL.CreateItem(1)
	With objAppt
		.Subject = "My Team Meeting"
		.Start = MeetingDate
		.End = DateAdd("h", 1, .Start)
         
		' make it a meeting request
		.MeetingStatus = 1
		.RequiredAttendees = "someone@somewhere.dom" 'indicate the email address of the resource mailbox for this
		.Send
	End With
     
	Set objAppt = Nothing
	Set objOL = Nothing
End Sub
SendMeetingRequest
{% endhighlight %}

# Windows Task Scheduler for the automation
I used Windows Task Scheduler for the automation i required to send the booking invite. 
1. Go to your Windows Task Scheduler from your Start Menu. If it is not there, type Scheduler in the Search program and files(highlighted in screenshot below)
<img src="{{ site.baseurl }}/images/taskscheduler1.jpg" alt="">
2. Click on Create Basic Task... in the right panel of Task Scheduler
<img src="{{ site.baseurl }}/images/taskscheduler2.jpg" alt="">
3. Type a Name for the task and click Next>.
4. Set the Trigger as Weekly (in my case i set it as weekly because the automation is to book the meeting room every week). Then click Next>.
5. Set the Start Date as the day of your coming Meeting and Time when your Meeting ends. (In my case my meeting is every Wednesday 11am and ends at 12pm. So I set the start date 30 Aug 2017 and time 1201PM as the coming meeting will be on 30 Aug ). Recur to set to 1 week on Wednesday. Click Next>.
<img src="{{ site.baseurl }}/images/taskscheduler3.jpg" alt="">
6. Action to set to "Start a program". Then click Next>.
7. Browse for the vbs file you saved earlier. Leave the rest of the fields blank because it is unnecessary. Then Click on Next>.
<img src="{{ site.baseurl }}/images/taskscheduler4.jpg" alt="">
8. Click Finish to complete the setup.
9. Wait for the next time this schedule will run. In my case it will be on 30 Aug 2017 1201pm. The testing will be successful if you received an accepted Meeting Invite in your outlook.


# Conclusion
This marks the end of the tutorial. The Windows Task Scheduler is a very good tool which can be used for automation, this is just one example of how can leverage on something which is default and available easily.