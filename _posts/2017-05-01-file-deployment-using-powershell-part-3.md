---
layout: post
title: "Files Deployment Using Powershell (Part 3): Link Them All Up"
summary:  This is the last of three posts to show how to create a file deployment utility with validation using powershell 
excerpt: "After explaining all the code for this, now is to link them all up!"
categories: Tutorials
tags: [Powershell]
comments: true
---


<img src="{{ site.baseurl }}/images/Endresultafterenhancement.jpg" alt="">

In the previous post, I have explained the individual code segments that makes this program tick. Now is the time to link them all up.
<br><br>
### Name your program

In order to run your program, you will need to name it first. I am calling mine Appimpl (short for application implementation). The code would be something like the below:
{% highlight powershell %}
function Appimpl {}
{% endhighlight %}
We will need to declare this program as a function so that we can trigger the program by keying in the name in powetshell. Between the opening and closing curly bracket are where all the code will be. 
<br><br>
### Start with the ingredients and user input

Immediately after naming your program. Add the ingredients and user input

{% highlight powershell %}
$filename = @()
$objects=@()
do {
$input = Read-host “Input the filename [Hit Enter for last value]”
if ([string]::IsNullOrEmpty($input)){}
else{$filename += $input}
} until ([string]::IsNullOrEmpty($input))
$inputsource = Read-host “Input the source path “
$inputDestination = Read-host “Input the destination path “
{% endhighlight %}
<br>
### Time Stamp the start of the deployment

One of the optional requirements is to get the start and end date time of the deployment. So when do we request for it? It will be when the actual copying of the files started, which will be immediately after the user inputs are completed. Thus next we will add the showing of the current date/time next. 
{% highlight powershell %}
Start of Deployment: '' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName
{% endhighlight %}
get-date is the powershell command to retrieve the current date time while  specifying the format(-format F) of the Date/Time to be of type long date long time. The ToString is to convert the current date/time to a string so that I can merge it with the timezone([string][TimeZoneInfo]::Local.DisplayName) as a single sentence
<br><br>
### Time to start the deployment

When requesting for user input of the file name, we write each of the filename to an array called…..filename. Now we will need to call out each individual item in the filename array to perform the deployment.

{% highlight powershell %}
ForEach ($item in $filename)
Copy-Item -Path $inputsource\$item -force -Destination $inputDestination
{% endhighlight %}
<br>
### Differentiating the source and destination file

After completing the file copy, we will need to verify if it is successfull by getting the hashfile, filesize and datemodified of each file. One at the source and one at the destination. So in order to cater to that, we will create a 2D array (An array sitting inside another array). This is to house the text ‘Source’ or ‘Destination’ (I use that to differentiate if the information is for the file at the source or destination) and the Source Location (which is in $inputsource) and Destination Location($inputDestination)

{% highlight powershell %}
$SourceOrDestinations = @((“Source”, $inputsource),(“Destination”,$inputDestination))
{% endhighlight %}
<br>
### Get all the file information(hashcode, timestamp, datemodified)

Using the 2D array, we will get the file information of the file at the source and destination. Foreach will allow us to read the source and followed by the destination.



{% highlight powershell linenos %}
ForEach($choice in $SourceOrDestinations)
{                     
	$object = New-Object PSObject -Property @{ 
		Path = $item
                    }
	$fullfilepath=$choice[1]
	$itemsize = (Get-Item "$fullfilepath\$item").length
	$itemdate = (Get-Item "$fullfilepath\$item").LastWriteTime
 
	$stream = ([IO.StreamReader]"$fullfilepath\$item").BaseStream
	[string]$hash = -join ([Security.Cryptography.HashAlgorithm]::Create( "MD5" ).ComputeHash( $stream ) | ForEach { "{0:x2}" -f $_ })
	$stream.Close()
 
	$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Location" -Value $choice[0] -PassThru
	$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Size" -Value $itemsize -PassThru
	$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Date Modified" -Value $itemDate -PassThru
	$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Hash" -Value $Hash -PassThru
 
	$objects += $object
}
{% endhighlight %}
<details>
<summary>Click here for explanation of code above</summary>
<span style="color:blue">Line 1</span> is to loop the array to read the source path followed by the destination.<br>
<span style="color:blue">Line 2 and 20(the last closing curly bracket)</span> wraps the code that is required to get the file information of each file(One for source and one for destination).<br>
<span style="color:blue">Line 3-5</span> is create the object required to display the information in tabular format. Line 6 is to get the file path which is residing in the SourceOrDestination array.<br>
<span style="color:blue">Line 7-8</span> is to get the file information required (filesize .length and datemodified .LastWriteTime).<br>
<span style="color:blue">Line 10-12</span> is gets the hash code.<br>
<span style="color:blue">Line 6-12</span> writes the information of location, filesize, date modified and hashcode of the file in object array.<br>
<span style="color:blue">Line 14-19</span> writes all the information collected from Line 6-12 into the objects array so that it can be displayed in tabular format later.
</details>
<br>
### Adding the finishing touches

Well at the end we can finally display all the information in the tabular format. In addition I added the Source Path and Destination as part of the output so we do not need to bloat the information that is in tabular format as we can exclude the full path just mentioning whether it is from the Source Path or Destination Path.

{% highlight powershell %}
Write-Host ‘Source Path: ‘ + $inputsource
Write-Host ‘Destination Path: ‘ + $inputDestination
write-output $objects | format-table -autosize
‘End of Deployment: ‘ +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName
{% endhighlight %}

So that’s all to it. To see the code altogether, get to [last section of the introduction post]({% post_url 2017-05-01-file-deployment-using-powershell-part-1 %}#allcode).