---
layout: post
title: "Files Deployment Using Powershell (Part 1): Introduction"
excerpt: Introduction to my file deployments using Powershell
summary: This is the first of three posts to show how to create a file deployment utility with validation using powershell 
categories: Tutorials
tags: [Powershell]
comments: true
---

### Some Background Story
I need a tool to do audit friendly deployment tasks. Bulk of my deployment activities involved simple copy and paste, but auditors have been challenging the method that I am using(select the file and copy and paste) as there is no accountability of this process. This issue can be easily resolved by having a automated deployment tool but that is a WIP with a project created for that. In the meantime, I need a short term solution to satisfy the auditors’ ‘hunger’ on the accountability. Basically, this tool will need to prove that what I deployed is the correct thing.

I have gathered a short list of requirements of what my tool is supposed to achieve.

 1. This needs to work on Windows 7 Desktop
 2. Something a normal user can start using immediately without requirements to install anything
 3. Can transfer files to UNC(Universal Naming Convention) path
 
The challenge about accountability is how can I prove that what I have deployed are actually the correct files. I believed i can overcome this by showing the hash code of individual files from the source and destination. This should be enough to prove the accountability.

**End Product**
The end result of what I have created is something like the below.
<img src="{{ site.baseurl }}/images/Endresult.jpg" alt="">


You can just get the end product code at the bottom. I am presuming here that you would have some basic knowledge of powershell here in order for you to use the end product.
### Explanation without techie jargons

The layman explanation of the code is as below:

 1. Copy file from A to B
 2. Read the hash code/Size in byte/Date Modified of the file in A and B
 3. Publish the results in a tabular format
<a name = "allcode"></a>
If you want to go through the whole long explanation of how its done, just go to [next post]({{ site.baseurl }}{% post_url 2017-05-01-file-deployment-using-powershell-part-2 %}).

{% highlight powershell %}
$filename = @()
$objects=@()
do {
 $input = Read-host "Input the filename [Hit Enter for last value]"
 if ([string]::IsNullOrEmpty($input)){}
 else{$filename += $input}
} until ([string]::IsNullOrEmpty($input))

$inputsource = Read-host "Input the source path "
$inputDestination = Read-host "Input the destination path "
'Start of Deployment: ' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName

ForEach ($item in $filename) {
 Copy-Item -Path $inputsource\$item -force -Destination $inputDestination
 $SourceOrDestinations = @(("Source", $inputsource),("Destination",$inputDestination))
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
}

Write-Host 'Source Path: ' $inputsource
Write-Host 'Destination Path: ' $inputDestination
write-output $objects | format-table -autosize
'End of Deployment: ' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName

}
{% endhighlight %}

