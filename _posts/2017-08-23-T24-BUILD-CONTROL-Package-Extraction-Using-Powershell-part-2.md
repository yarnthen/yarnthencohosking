---
layout: post
title: "Extract a T24 BUILD.CONTROL Package using Powershell(Part 2): Explanation of Code"
excerpt: "This is post teaches you how to use PowerShell to extract T24 (Temenos Core Banking Product) Build Control Package"
summary: "This is second of two posts to teach you how to use PowerShell to extract T24 (Temenos Core Banking Product) Build Control Package"
categories: Tutorials
tags: [Powershell, T24, BUILD.CONTROL, BCON, Temenos]
comments: true
---

I structure this post by breaking down the code according to the requirements. I hope this will allow everyone to better understand how its done.<br>

### Ingredients we need
There are multiple strings we need to gather what we want.
* String to store the package name. This is gathered by requesting user input.
{% highlight powershell %}
$input = Read-host "Input the package name"
{% endhighlight %}
* String to store the index file name. This is derived from the package name. Index file is inside the package and index file naming convention is prefixed by DL.D_ and ending with package name.
{% highlight powershell %}
$filetoread="$input\DL.D_$input"
{% endhighlight %}
* String to store all the folder names(TABLE in T24). This is on line 9 of the index file.
{% highlight powershell %}
$foldernames=(Get-Content $filetoread)[8]
{% endhighlight %}
* String to store all the file names(RECORD in T24). This is on line 10 of the index file.
{% highlight powershell %}
$filenames=(Get-Content $filetoread)[9]
{% endhighlight %}
* String array to store all the folder names(TABLE in T24). Difference with foldernames string is that it is an array that holds the individual folder names by splitting foldernames string based on separator "ý"
{% highlight powershell %}
$foldername = $foldernames -split "ý"
{% endhighlight %}
* String array to store all the file names(RECORD in T24). Difference with filenames string is that it is an array that holds the individual file names by splitting filenames string based on separator "ý"
{% highlight powershell %}
$filename = $filenames -split "ý"
{% endhighlight %}

### Storing the output package
This is to store the output of the extracted package. We will keep the package name intact but instead place it in a folder created called Restore. out-null is solely for hiding any output message like folder created which is unnecessary for this.
{% highlight powershell %}
mkdir "Restore\$input" | out-null
{% endhighlight %}

### Creating the folders(TABLE in T24)
Remember I mentioned about the T24 records are stored in individual folders (TABLE in T24)) which are derived from the index file line number 9. This is to create the folders itself so that the files(record name in T24) can be placed in there later. The if statement is to check if the folder was already created by one of the earlier entries as there is no need to recreate again if that is the case
{% highlight powershell %}
foreach ($folder in $foldername){
    if ( -Not (Test-Path -Path "Restore\$input\$folder")){
        New-Item -ItemType directory -Path "Restore\$input\$folder" | out-null
    }
}
{% endhighlight %}

### Copy and renaming the file names in deployment package to its RECORD names
Now is the task to copy and renaming the file names according to RECORD names. The sequence of the file name (REC0000x) correspond to the sequence of RECORD names indicated in the line 10 of the index file. The ToString part is to add preceding zeros so that RECORD number is 00001 and not 1 because that is the naming convention of the file names.
{% highlight powershell %}
for ($i=0; $i -lt $foldername.length; $i++){
    $recordname = "REC" + ($i + 1).ToString("00000")
    $folderpath=$foldername[$i]
    $filepath=$filename[$i]
    Copy-Item "$input\$recordname" -force "Restore\$input\$folderpath\$filepath"
    }
}
{% endhighlight %}

### Enclose the code to create a function
In order to run your program, you will need to name it first. I am calling mine ExtractBCON. Add at the end of the code ExtractBCON as well to trigger the program immediately. This will be useful if you converted the entire  code to run as batch file. Enclose all the rest of the code between the curly brackets. The code would be something like the below:
{% highlight powershell %}
function ExtractBCON{
<all the rest of code here!!!>
}ExtractBCON
{% endhighlight %}

### Conclusion
So that’s all to it. To see the code altogether, get to [last section of the introduction post]({{ site.baseurl }}{% post_url 2017-08-22-T24-BUILD-CONTROL-Package-Extraction-Using-Powershell-part-1 %}#allcode).