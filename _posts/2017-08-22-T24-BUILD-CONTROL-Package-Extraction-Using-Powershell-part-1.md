---
layout: post
title: "Extract a T24 BUILD.CONTROL Package using Powershell(Part 1): Introduction"
excerpt: "This is post teaches you how to use PowerShell to extract T24 (Temenos Core Banking Product) Build Control Package"
summary: "This is first of two posts to teach you how to use PowerShell to extract T24 (Temenos Core Banking Product) Build Control Package"
categories: Tutorials
tags: [Powershell, T24, BUILD.CONTROL, BCON, Temenos]
comments: true
---

### Some Background Story
Those of you who are familiar with the Core Banking Software T24 would know that deployment of its programs or data is via their manual method deployment tool DL.DEFINE or much more automated tool BUILD.CONTROL. As most of its deployment will need to leverage on these methods, deployment will need to be done via a package which for BUILD.CONTROL is BCON packages. The BCON pack is designed such a way where outside of T24 application, it cannot be interpreted correctly because of naming convention used in the packages(see below). This makes it a nightmare for version control because record names for BCON package does not match the record names they are deploying in T24.
<img src="{{ site.baseurl }}/images/BCONExample.jpg" alt="">
<img src="{{ site.baseurl }}/images/BCONExampleExtracted.jpg" alt="">

Fortunately the files in the BCON package are actually only normal plain text files with an INDEX file(prefixed with DL.D_) to relate each of REC* records to its corresponding true record name. So it is possible to generate the package into their real record names for version control purposes. Below is the program I have written using Powershell to do just that.

You can just get the end product code at the bottom. I am presuming here that you would have some basic knowledge of powershell here in order for you to use the end product.

### Explanation without techie jargons
1. Read the index file(prefixed with DL.D_)
2. Go to Line 9 of the index file where it will indicate the list of folder name(TABLE in T24). The individual items are separated by a special character ý. Each item sequence represent the sequence as per the filename in BCON Package (e.g. first item in the list correspond to the folder of the REC00001, 2nd will be REC00002)
3. Go Line 10 of the index file where it will indicate the file name(RECORD in T24). The individual items are separated by a special character ý. Each item sequence represent the sequence as per the filename in BCON Package (e.g. first item in the list correspond to the record name of the REC00001, 2nd will be REC00002)
4. Create the folder based on 2 and copy the corresponding files and rename them to the record name based on what is derived from 3.

<a name = "allcode"></a>
If you want to go through the whole long explanation of how its done, I will discuss it in my [next post]({% post_url 2017-08-23-T24-BUILD-CONTROL-Package-Extraction-Using-Powershell-part-2 %}).


{% highlight powershell %}
function ExtractBCON{
    $input = Read-host "Input the package name"
    $filetoread="$input\DL.D_$input"
    $foldernames=(Get-Content $filetoread)[8]
    $filenames=(Get-Content $filetoread)[9]
    $foldername = $foldernames -split "ý"
    $filename = $filenames -split "ý"
    rd "Restore\$input" -recurse -erroraction "silentlycontinue"
    mkdir "Restore\$input" | out-null
    foreach ($folder in $foldername){
        if ( -Not (Test-Path -Path "Restore\$input\$folder")){
            New-Item -ItemType directory -Path "Restore\$input\$folder" | out-null
        }
    }
    for ($i=0; $i -lt $foldername.length; $i++){
        $recordname = "REC" + ($i + 1).ToString("00000")
        $folderpath=$foldername[$i]
        $filepath=$filename[$i]
        Copy-Item "$input\$recordname" -force "Restore\$input\$folderpath\$filepath"
    }
}ExtractBCON
{% endhighlight %}

You will need to input everything above in a PowerShell Prompt in order for it to work. If you want to run this as a batch file for easy reusability. Please refer to [this post]({% post_url 2017-07-01-powershell-to-command-prompt-conversion %}) for how to convert powershell to a batch file.