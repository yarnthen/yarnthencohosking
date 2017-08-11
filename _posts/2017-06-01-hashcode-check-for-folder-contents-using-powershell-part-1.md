---
layout: post
title: "Hash Code Check For Files In A Folder Using Powershell (Part 1): Introduction"
excerpt: "Learn how o create a single hash code for an entire folder of files and folder"
summary: "This is the first of two posts to teach you how to do a hash code check on an entire folder's content using powershell"
categories: Tutorials
tags: [Powershell]
comments: true
---

Microsoft has provided a means to generate hashcode using the File Checksum Integrity Verifier(fciv). It allows a hashcode to be generated for a single file and also for all files in a folder using the -xml option to write all the hashcode of files in a folder in an xml file.

But fciv has been created more than 10 years ago and being stated as unsupported means Microsoft will not enhance it further. fciv can only support up to cryptography standard of MD5 and SHA1 and currently higher standards(eg SHA256) are already available. So in order to overcome this, there are third party tools available in the market(including [one from Microsoft via Powershell script](https://gallery.technet.microsoft.com/PowerShell-File-Checksum-e57dcd67)). I have decided to take up a challenge to create one myself by writing it in Powershell.

<span style="color:#993366">**Explanation without techie jargons**</span>

The layman explanation of the code is as below:
<a name = "allcode"></a>
1. Get all the files in a particular folder(including subfolders)
2. Read the hash code of each individual file
3. Write hash code of each individual file to an xml
4. Final output will be the hash code of the xml file
You can just get the end product code at the bottom. If you want to go through the whole long explanation of how its done, just go to [next post]({% post_url 2017-06-01-hashcode-check-for-folder-contents-using-powershell-part-2 %}).


{% highlight powershell %}
function checkhash {
$source = Read-host "Input the path:"
$source = resolve-path -path $source
function gethash ($hashcheck){
$stream = ([IO.StreamReader]"$hashcheck").BaseStream
[string]$hashcode = -join ([Security.Cryptography.HashAlgorithm]::Create( "SHA256" ).ComputeHash( $stream ) | ForEach { "{0:x2}" -f $_ })
$null = $stream.Seek(0,0)
$stream.Close()
return $hashcode
}
$filenames=gci -path $source -recurse  | Where-Object {$_.mode -notmatch "d"} | select -expandproperty fullname
$folder=Split-Path $source -Leaf
$objects=@()
foreach ($filename in $filenames){
$file = $filename.Substring($Source.Path.Length + 1)
$object = New-Object PSObject -Property @{Path=$folder+ "\" + $file}
$hash = gethash $filename
$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name "SHA256" -Value $Hash -PassThru
 
$objects += $object
}
"<?xml version=""1.0"" encoding=""utf-8""?>" >hash.xml
"<FILEHASH>" >>hash.xml
foreach ($object in $objects){
"`t<FILE_ENTRY>" >>hash.xml
"`t`t<name>" + $object.path + "</name>" >>hash.xml
"`t`t<SHA256>" + $object.SHA256 + "</SHA256>" >>hash.xml
"`t</FILE_ENTRY>" >>hash.xml
}
"</FILEHASH>" >>hash.xml
$hash = gethash "hash.xml" 
$hash
}
{% endhighlight %}