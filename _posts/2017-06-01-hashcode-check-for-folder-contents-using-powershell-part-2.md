---
layout: post
title: "Hash Code Check For Files In A Folder Using Powershell (Part 2): Explanation Of Code"
excerpt: "Learn how to create a single hash code for an entire folder of files and folder"
summary: "This is the last of two posts to teach you how to do a hash code check on an entire folder's content using powershell"
categories: Tutorials
tags: [Powershell]
comments: true
---


I structure this post by breaking down the code according to the requirements. I hope this will allow everyone to better understand how its done.


<span style="color:#993366">**Reusable Function** </span>

The main part of this is to generate the hash code of individual files. As this portion requires to be executed a lot of times(depends on number of files in the folder), the most efficient way to code this is by writing a function for generating of hash code so that it can be reused again and again. Explanation of the code is available in [my earlier post.]({% post_url 2017-06-01-hashcode-check-for-folder-contents-using-powershell-part-1 %}#hashcode)



{% highlight powershell %}
function gethash ($hashcheck){
$stream = ([IO.StreamReader]"$hashcheck").BaseStream
[string]$hashcode = -join ([Security.Cryptography.HashAlgorithm]::Create( "SHA256" ).ComputeHash( $stream ) | ForEach { "{0:x2}" -f $_ })
$null = $stream.Seek(0,0)
$stream.Close()
return $hashcode
}
{% endhighlight %}

<span style="color:#993366">**Consolidation of all files**</span>

Next is to gather all the files which are present in the folder so that we can use it to find the individual hash code of each file. we will use the function get-ChildItem to loop the entire folder 

Command will be like the below:

<span font-family="courier new">$filenames=<span style="color:blue">gci -path $source -recurse</span>  | <span style="color:red">Where-Object {$_.mode -notmatch “d”}</span> | <mark>select -expandproperty fullname</mark>></span>

The <span style="color:blue">blue</span> statement is to use the get-childItem (short form gci) to search for all the items in $source recursely(including subfolders). Result will be piped to the <span style="color:red">red</span> statement which is to omit all the directories(we only need hash code of files). Finally this is piped to the <mark>highlighted</mark> statement which will get the full path and name. All these will be written to the filenames string array.

Getting the full filename is necessary as the gethash function we created will require the full filename in order to generate the hashcode. But the final output of this is to create all the filenames and its hashcodes in the xml file. So having the full filename is no good as the full filename will change depends on the location you placed the folder. What we need is the relative path and filename from the location the folder is residing

For example if you place the folder named happy in C:\Users\Yarnthen. A fullfilename will be something like C:\Users\Yarnthen\happy\fileA. We can’t write the full filename to the xml file because it will just keep changing depending on where you place the folder. (e.g. C:\Users\Cohos\happy\fileA if you put the folder in C:\Users\Cohos). Thus what we need here is the relative path and filename (in the example shown will be something like \fileA). This will be the same throughout no matter where we place the folder. To do this, we will need to strip the Source path from the full filename.

Command will be like the below:

<span style="color:red" font-family="courier new">$folder=Split-Path $source -Leaf</span><br>
<span style="color:red" font-family="courier new">$file = $filename.Substring($Source.Path.Length + 1)</span>


<span style="color:#993366">**Create an object**</span>


We will need an array to hold all the information. In order to achieve this, we will need to declare a PSObject for each record and store it to the objects array. PSObject is a custom object which allows you to add custom properties to the object itself. For example in this case, I can create a PSObject to house the name of the file and add custom property of Hashcode to PSObject.


<span style="color:red" font-family="courier new">$objects=@()</span><br>
<span style="color:red" font-family="courier new">$object = New-Object PSObject -Property @{Path=$folder+ “\” + $file}</span><br>
<span style="color:red" font-family="courier new">$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name “SHA256” -Value $Hash -PassThru</span>

<span style="color:#993366">**Generate the data**</span>

Once we know how to create the object, we can start to generate all the information required to form the XML. And the code will be just to loop all the files and store its relative filename and hashcode(using the reusable function) to the PSObject. As below:

<span style="color:red" font-family="courier new">$objects=@()</span><br>
<span style="color:red" font-family="courier new">foreach ($filename in $filenames){</span><br>
<span style="color:red" font-family="courier new">$file = $filename.Substring($Source.Path.Length + 1)</span><br>
<span style="color:red" font-family="courier new">$object = New-Object PSObject -Property @{Path=$folder+ “\” + $file}</span><br>
<span style="color:red" font-family="courier new">$hash = gethash $filename</span><br>
<span style="color:red" font-family="courier new">$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name “SHA256” -Value $Hash -PassThru</span><br>
<span style="color:red" font-family="courier new">$objects += $object</span>

<span style="color:#993366">**Create the xml file**</span>

Once we have all the information in the PSObject, we can use it to create the XML file (i am naming it hash.xml). Of course the xml file must have some headers in content as per what other xml files have. I have taken the cue from the xml file that can be generated by fciv for headers, after all this is actually still doing what fciv is doing, albeit in a more advanced hashing algorithm.


{% highlight powershell %}
"<?xml version=""1.0"" encoding=""utf-8""?>" >hash.xml
"<FILEHASH>" >>hash.xml
foreach ($object in $objects){
"`t<FILE_ENTRY>" >>hash.xml
"`t`t<name>" + $object.path + "</name>" >>hash.xml
"`t`t<SHA256>" + $object.SHA256 + "</SHA256>" >>hash.xml
"`t</FILE_ENTRY>" >>hash.xml
}
"</FILEHASH>" >>hash.xml
{% endhighlight %}

<span style="color:#993366" font-family="arial black">**Hashing the hash.xml**</span>

Last part will be to hash the hash.xml created so as to achieve our end state of one hash code.

<span style="color:red" font-family="courier new">$hash = gethash “hash.xml”</span>
<span style="color:red" font-family="courier new">$hash</span>

<span style="color:#993366" font-family="arial black">**Try it out!**</span>

Do some testing with the program. Move the folder around and try to generate the hash to see if it is still the same. Make some changes and check the hash again. That is all for this post. Remember you can get the full code in the [introduction post]({% post_url 2017-06-01-hashcode-check-for-folder-contents-using-powershell-part-1 %}#allcode)

