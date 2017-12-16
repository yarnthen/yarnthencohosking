---
layout: post
title: "How to automate booking of meeting room(resource mailbox in Outlook): VBScript code explanation"
excerpt: This is continuation of the post that teaches you how to automate a booking of resource (e.g. Meeting Room) which is via a resource MailBox in Outlook (Exchange) where I explain in depth the VBscript code.
summary: This is continuation of the post that teaches you how to automate a booking of resource (e.g. Meeting Room) which is via a resource MailBox in Outlook (Exchange) where I explain in depth the VBscript code.
categories: Tutorials
tags: [vbscript, Outlook, Windows, Task Scheduler, Scheduling, Meeting, Schedule]
comments: true
---

This is actually a very short vb script code which i used from [outlookcode.com](http://www.outlookcode.com/codedetail.aspx?id=88) and make minor tweaks to get it to work the way i want. So what I will do is just to go line by line for the code itself

{% highlight vbscript linenos %}
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

1. **Line 1 and 24:** Wrapper for the function SendMeetingRequest. Anything inside will be run as part of the function.
2. **Line 2-4:** Declaration of the objects required. We need to declare the Outlook object(objOL) in order to use its functions and AppointmentItem(objAppt) so that we can create a meeting request to send out. MeetingDate is used to gather the inputs required which in this case is to find the date and time for the Meeting Request.
3. **Line 6-7:** Used to find the next date time for the next meeting request. In this case, the limit is 4 weeks so I will add 28 days to date and merge the date with the time of the meeting.
4. **Line 9:** Create the Outlook Object instance so that we can use its function. In this case, will be in Line 10 which is to CreateItem
5. **Line 10:** Create the instance which holds the Meeting Request. objOL.CreateItem with arguement 1 is to indicate create an appointitem. Other options like 2 is to create a ContactItem and 0 is a MailItem
6. **Line 11-18:** Set the necessary information for the sending of the AppointmentItem to be successful. Subject of the Appointment, Start Date/Time, End Date/Time(derived by adding 1 hour from Start Date/Time), Status(set to 1 to schedule the meeting, other options like 2 is to cancel the request) and lastly Required Attendees(to set to the email address of the resource mailbox. You will need to find the email address of the meeting room and add here)
7. **Line 19:** Execute the sending of request
8. **Line 22-23:** Destroy instances of objects created earlier for memory housekeeping
9. **Line 25:** The earlier codes from Line 1 to 24 is to create the function while Line 25 is the actual execution of the routine.

