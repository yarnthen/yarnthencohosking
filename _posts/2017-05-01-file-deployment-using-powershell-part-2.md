---
layout: post
title: "Files Deployment Using Powershell (Part 2): Explanation of Code"
excerpt: "I structure this post by breaking down the code according to the requirements. I hope this will allow everyone to better understand how its done."
summary: This is the second of three posts to show how to create a file deployment utility with validation using powershell 
categories: Tutorials
tags: [Powershell]
comments: true
---


I structure this post by breaking down the code according to the requirements. I hope this will allow everyone to better understand how its done.<br><br>


### Ingredients we need

There are 2 arrays that we will require to house the below information:<br>
-filename: array to house all filenames to do multi file transfer<br>
-object: array to house all the info for output in a tabular format later

The code could be something like the below:
{% highlight powershell %}
$filename = @()
$objects=@()
{% endhighlight %}

### 1st Requirement: User Input

We will need some code to read for inputs to determine what needs to be copied.
1st thing will be the filename, which will be getting via the below code:

{% highlight powershell %}
$input = Read-host “Input the file name”
{% endhighlight %}
This will trigger a user input that will write into a string called input

Remember one of the ingredients we required is an array to house all the filenames to do multi file transfer. This is where it will come in handy. The way we are going to prompt for additional files to added is by looping the question  “Input the filename” until the user hit enter without any other inputs. Oh yeah, we also change the question in order to make it more intuitive “Input the filename [Hit Enter for last value]”.

{% highlight powershell linenos %}
$input = Read-host "Input the filename [Hit Enter for last value]"
  if ([string]::IsNullOrEmpty($input)){}
  else{$filename += $input}
} until ([string]::IsNullOrEmpty($input))
{% endhighlight %}

<details>
<summary>Click here for explanation of code above</summary>
<span style="color:blue">1st line and the last line</span> is to indicate to perform whatever that is in 2nd to 4th line until it detected a last entry (ENTER key without any input).<br>
<span style="color:blue">2nd line</span> is the question and the request for user input.<br>
<span style="color:blue">3rd line</span> is to detect for the last entry (ENTER key without any input).<br> 
<span style="color:blue">4th line</span> will write the input into the array filename until the last entry.
</details><br>
Next thing to do is to request for user input of the source/destination path and write them to string variables for usage later.
Command will be like the below:
{% highlight powershell %}
$inputsource = Read-host "Input the source path "
$inputDestination = Read-host "Input the destination path "
{% endhighlight %}<br>
### 2nd Requirement: File Copy (obviously)

File Copy is done using Powershell command <span style="color:blue">copy-item</span> with parameters Path, Force, Destination. 
-Path: Full filename (source path plus the filename).
-Force: to indicate that copying is mandatory with no questions asked (e.g. sometimes copying a read-only file will result in a prompt to acknowledge the copying of the read-only file).
-Destination: for the destination path to copy to.
Command will be like the below:
{% highlight powershell %}
Copy-Item -Path $filename -Force -Destination $inputDestination
{% endhighlight %}
Now is the time to copy all the files(their name) stored in the filename array. To do this we will need to read each individual entry in the filename array.


{% highlight powershell linenos %}
ForEach ($item in $filename) {
        Copy-Item -Path $inputsource\$item -force -Destination $inputDestination
}
{% endhighlight %}

<details>
<summary>Click here for explanation of code above</summary>
<span style="color:blue">1st and last line</span> of the code is to read each individual entry in the filename array and perform whatever is in line 2.<br>
<span style="color:blue">Line 2</span> is the command to do the filecopy.
</details>
<br>
### 3rd Requirement: Showing the hashcode of files

<a name = "hashcode"></a>
Showing of hashcode in Powershell 4+ is a simple one line cmdlet called Get-FileHash. But the powershell version that comes default with my Windows 7 machine is 2.x. Trying to be as vanilla as possible, so I decided against upgrading to Powershell 4+. Fortunately, Boe Prox has solved this in [his blog post](http://learn-powershell.net/2013/03/25/use-powershell-to-calculate-the-hash-of-a-file). Below is the extract I used from [his script](https://gallery.technet.microsoft.com/scriptcenter/Get-Hashes-of-Files-1d85de46).

{% highlight powershell linenos %}
$stream = ([IO.StreamReader]"fullfilename").BaseStream
[string]$hash = -join ([Security.Cryptography.HashAlgorithm]::Create("MD5").ComputeHash( $stream ) | ForEach { "{0:x2}" -f $_ })
$null = $stream.Seek(0,0)
$stream.Close()
{% endhighlight %}

<details>
<summary>Click here for explanation of code above</summary>
<span style="color:blue">1st line</span> is to read the file as a file stream. The "fullfilename" is the location of the file and its filename combined.<br>
<span style="color:blue">2nd line</span> is the actual command to generate the hash code.("MD5" can be changed to whatever algorithm you want like SHA256).<br>
<span style="color:blue">The last two line</span> is to clean up and close the file stream in order for it to be reused for the next hash code generation. 
</details><br>

### Optional Requirement: Showing timestamp and size of the files, start and end date time of the deployment

Trying to make this as a more professional looking deployment tool, I have decided to add in the timestamp and size of file as part of the output. It should also be able to show the start and end date time of the deployment
Code is straightforward for getting the file details using cmdlets Get-Item to seek for the length(size of file) and LastWriteTime(timestamp or last modified date)
{% highlight powershell %}
$itemsize = (Get-Item "fullfilename").length
$itemdate = (Get-Item "fullfilename").LastWriteTime
{% endhighlight %}<br>
As for the part of  deployment start and end date time, modify(hint: change the “Start of Deployment” to “End of Deployment”) as needed and place the below code at the beginning of the program and the end of the program.

{% highlight powershell %}
Start of Deployment: '' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName
{% endhighlight %}
<br>
### Output in table format

Remember the final output you see in the [introduction]({{ site.baseurl }}{% post_url 2017-05-01-file-deployment-using-powershell-part-1 %})? We will need an array to hold all the information and finally output all the results. One of the first ingredient at the start of this post called objects which is an array will be host to that.

Next we will need to add records to the objects array. Records will be the name of the file, one record for the one at the source and another record for the one at the destination. Each entry should be able hold some information of its own. For example  a record for file named abc.txt should also have information in the same record its hashcode, timestamp and filesize.

In order to achieve this, we will need to declare a PSObject for each record and store it to the objects array. PSObject is a custom object which allows you to add custom properties to the object itself. For example in this case, I can create a PSObject to house the name of the file and add custom properties to PSObject itself like for hashcode, timestamp.

Example code will be as below:

{% highlight powershell linenos %}
$object = New-Object PSObject -Property @{ 
   Name = $item
}
$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Location" -Value $SourceOrDestination -PassThru
$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Size" -Value $itemsize -PassThru
$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Date Modified" -Value $itemDate -PassThru
$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "Hash" -Value $Hash -PassThru
{% endhighlight %}
<details>
<summary>Click here for explanation of code above</summary>
<span style="color:blue">1st line of the code</span> is to create a new PSObject by the name of object (I used object as we are going to store it into the objects array which seems common sense to me, but if that confuses you, change it into something else)<br>
<span style="color:blue">2nd line</span> is to create the identifier of that object which is the name. ($item is the string variable which will house the filename).<br>
<span style="color:blue">3rd line</span> is to close the object creation.<br>
<span style="color:blue">4,5,6,7</span> are for adding of custom properties to the object itself which we have already identified as filesize(string variable $itemsize), date modified(string variable $itemDate) and hashcode (string variable $Hash). One more property will be SourceOrDestination(string variable $SourceOrDestination). This is used to differentiate whether we are referring to the file at the Source Location or the file at the Destination location.
</details>
<br>Once we have all the information required for the object, we will just add them to the array like below:
{% highlight powershell %}
$objects += $object
{% endhighlight %}
Once we have completed adding all the object to the objects array. We can print the output with the command below:
{% highlight powershell %}
write-output $objects | format-table -autosize
{% endhighlight %}
This command above will print out in a table and autosize to align it for each entry.

Now that we have gotten all the core understanding of everything. It is time to link them all up. I will explain that in the [next post]({{ site.baseurl }}{% post_url 2017-05-01-file-deployment-using-powershell-part-3 %})

