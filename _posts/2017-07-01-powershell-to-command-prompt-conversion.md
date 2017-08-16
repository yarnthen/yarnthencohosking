---
layout: post
title: "Powershell To Command Prompt Conversion"
excerpt: "This post teaches you how to convert your powershell code into a batch file"
summary: "This post teaches you how to convert your powershell code into a batch file"
categories: Tutorials
tags: [Powershell, "Command Prompt", Batch]
comments: true
---

This is something which came out from the Files Deployment Using Powershell series which I wrote. I have created the program as a ps1 file so that i can integrate it to my powershell sessions, if not I will need to copy the entire function (40+ lines of code) into my powershell session before i can start using the program. But all thanks to default security settings that are in place which prevent execution of scripts in powershell, you might get the below error.

<img src="{{ site.baseurl }}/images/restricted-750x160.jpg" alt="">

Simplest way to overcome this is to [get the rights to execute the ps1 scripts in powershell](https://blogs.msdn.microsoft.com/vivekkum/2009/01/29/powershell-first-step/). But hey, I am about doing things differently so I went about figuring another way to get things done.

Introducing my script to others makes me realize most of them are uncomfortable with the thing called powershell preferring the good old command prompt. Doing some research over the web for a solution brought me to [a post from Dmitry Sotnikov](https://dmitrysotnikov.wordpress.com/2008/06/27/powershell-script-in-a-bat-file/) who wrote about running powershell script in a batch file.

So running my program in a single command(by virtue of a batch file) – checked, getting others to run my powershell program while taking into consideration their unfamiliarity with powershell(we will be running powershell script in command prompt via a batch file) – checked. There are actually two ways of getting this done.

The first way is to encode the powershell commands which  I am not going to talk about it as it is quite straightforward and [Dmitry’s post](https://dmitrysotnikov.wordpress.com/2008/06/27/powershell-script-in-a-bat-file/) should more or less covered it.

The second way is to “flatten” all the commands into a single line of powershell code which in itself is challenging as command prompt does not  work well with some codes written in powershell so it will involve substituting quotes(‘) and double quotes(“) and backslashes(\) (because of the way command prompt interprets double quotes).

I will be using the code of Files Deployment Using Powershell for example. The techniques could be reused on other powershell programs you wrote.  So here we go using the 2nd method.

<span style="color:#993366">**No comments required for this**</span>

If you wrote comments(those with a hashtag # in front) on your powershell program, it is time now to strip it. Having comments on it might mean misinterpretation by the command prompt, what more they are not necessary to execute the program. The key here is to keep the minimum required code in order for the program to run in command prompt, having more just means more chance of issues.

<span font-family="courier new">
======BEFORE====================================<br>
<br>
<span style="color:red">function Appimpl</span><br>
<span style="color:red">{</span><br>
    <span style="color:red">$filename = @()</span>   <span style="color:blue"> #array for filename</span><br>
    <span style="color:red">$fullfilename= @()</span> <span style="color:blue">#array for the fullfilename including the path</span><br>
    <span style="color:red">$objects=@()</span> <span style="color:blue">#array for the objects</span><br>
<br>
======AFTER====================================<br>
<br>
<span style="color:red">function Appimpl</span><br>
<span style="color:red">{</span><br>
    <span style="color:red">$filename = @()</span><br>
    <span style="color:red">$fullfilename= @()</span><br>
    <span style="color:red">$objects=@()</span><br>
</span>

<span style="color:#993366">**Not much space left**</span>

The task is to keep the minimum amount of code. So we do not need those fancy space to keep the code looking tidy. At the end, everything will be squeezed together. So remove any unnecessary space and trailing space at the end of line.

<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
function Appimpl<br>
{<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$filename = @()<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$fullfilename= @()<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$objects=@()<br>
</span>
======AFTER====================================<br>
<span style="color:red">
function Appimpl<br>
{<br>
$filename = @()<br>
$fullfilename= @()<br>
$objects=@()<br>
</span>
</span>

<span style="color:#993366">**Squeeze Codes! Squeeze!**</span>

Try to squeeze as much code as a one liner as possible. This will keep the code more compact. Code like the curly bracket can usually be squeezed to the preceding line

<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
function Appimpl<br>
{<br>
$filename = @()<br>
$fullfilename= @()<br>
$objects=@()<br>
</span>
======AFTER====================================<br>
<span style="color:red">
function Appimpl{<br>
$filename = @()<br>
$fullfilename= @()<br>
$objects=@()<br>
</span>
</span>

<span style="color:#993366">**Lost in Translation**</span>

Command prompt has issues interpreting double quotes. So in order to translate double quotes to be understandable by command prompt, there are 3 types of translation required

***1st Type: Convert all double quotes for enclosing actual text to single quote***
<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
inputsource = Read-host <span style="color:black">"</span>Input the source path <span style="color:black">"</span><br>
$inputDestination = Read-host <span style="color:black">"</span>Input the destination path <span style="color:black">"</span><br>
</span>
======AFTER====================================<br>
<span style="color:red">
inputsource = Read-host <span style="color:black">'</span>Input the source path <span style="color:black">'</span><br>
$inputDestination = Read-host <span style="color:black">'</span>Input the destination path <span style="color:black">'</span><br>
</span>
</span>

***2nd Type: Convert all double quotes for enclosing variables to backslash double quote***

<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
$itemsize = (Get-Item <span style="color:black">"</span>$fullfilepath\\$item<span style="color:black">"</span>).length<br>
$itemdate = (Get-Item <span style="color:black">"</span>$fullfilepath\\$item<span style="color:black">"</span>).LastWriteTime<br>
$stream = ([IO.StreamReader]<span style="color:black">"</span>$fullfilepath\\$item<span style="color:black">"</span>).BaseStream<br>
</span><br>
======AFTER====================================<br>
<span style="color:red">
$itemsize = (Get-Item <span style="color:black">\\"</span>$fullfilepath\\$item<span style="color:black">\\"</span>).length<br>
$itemdate = (Get-Item <span style="color:black">\\"</span>$fullfilepath\\$item<span style="color:black">\\"</span>).LastWriteTime<br>
$stream = ([IO.StreamReader]<span style="color:black">\\"</span>$fullfilepath\\$item<span style="color:black">\\"</span>).BaseStream<br>
</span>
</span>

***3rd Type: Convert all single backslash to double backslash <u>except for those backslash created by 2nd Type</u>***

<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
$itemsize = (Get-Item \\"$fullfilepath<span style="color:black">\\</span>$item\\").length<br>
$itemdate = (Get-Item \\"$fullfilepath<span style="color:black">\\</span>$item\\").LastWriteTime<br>
$stream = ([IO.StreamReader]\\"$fullfilepath<span style="color:black">\\</span>$item\\").BaseStream<br>
</span>
======AFTER====================================<br>
<span style="color:red">
$itemsize = (Get-Item \\"$fullfilepath<span style="color:black">\\\\</span>$item\\").length<br>
$itemdate = (Get-Item \\"$fullfilepath<span style="color:black">\\\\</span>$item\\").LastWriteTime<br>
$stream = ([IO.StreamReader]\\"$fullfilepath<span style="color:black">\\\\</span>$item\\").BaseStream<br>
</span>
</span>


<span style="color:#993366">**End it with a semicolon**</span>

At the end of each line, add a semicolon. This is to interpret a different line by command prompt.

<span font-family="courier new">
======BEFORE====================================<br>
<span style="color:red">
$itemsize = (Get-Item \\"$fullfilepath\\\\$item\\").length<br>
$itemdate = (Get-Item \\"$fullfilepath\\\\$item\\").LastWriteTime<br>
$stream = ([IO.StreamReader]\\"$fullfilepath\\\\$item\\").BaseStream<br>
</span>
======AFTER====================================<br>
<span style="color:red">
$itemsize = (Get-Item \\"$fullfilepath\\\\$item\\").length<span style="color:black">;</span><br>
$itemdate = (Get-Item \\"$fullfilepath\\\\$item\\").LastWriteTime<span style="color:black">;</span><br>
$stream = ([IO.StreamReader]\\"$fullfilepath\\\\$item\\").BaseStream<span style="color:black">;</span><br>
</span>
</span>

<span style="color:#993366">**Squeeze everything into 1 liner by removing the Carriage Return/Line Feed**</span>

Squeeze everything into one line which is to form a single line with all the code you have left.

<span style="color:#993366">**Add some prefix and suffix to make it work**</span>

Add the text <span style="color:red">powershell.exe -command "</span> to the front and <span style="color:red">;appimpl"</span> to the end of the code. This will be all that is required to make it work. You can save the code as a .bat file and run it from command prompt and you will have a powershell program running from the command prompt. My ending code will look like the below:

```shell_session
powershell.exe -command “function Appimpl {;$filename = @();$fullfilename= @();$objects=@();do {;$input = Read-host ‘Input the filename [Hit Enter for last value]’;if ([string]::IsNullOrEmpty($input)){}else{$filename += $input};} until ([string]::IsNullOrEmpty($input));$inputsource = Read-host ‘Input the source path ‘;$inputDestination = Read-host ‘Input the destination path ‘;’Start of Deployment: ‘ +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName;ForEach ($item in $filename) {;Copy-Item -Path $inputsource\\$item -force -Destination $inputDestination;$SourceOrDestinations = @((‘Source’, $inputsource),(‘Destination’,$inputDestination));ForEach($choice in $SourceOrDestinations){;$object = New-Object PSObject -Property @{Path = $item};$fullfilepath=$choice[1];$itemsize = (Get-Item \\"$fullfilepath\\\\$item\\").length;$itemdate = (Get-Item \\"$fullfilepath\\\\$item\\").LastWriteTime;$stream = ([IO.StreamReader]\\"$fullfilepath\\\\$item\\").BaseStream;[string]$hash = -join ([Security.Cryptography.HashAlgorithm]::Create( ‘MD5’ ).ComputeHash( $stream ) | ForEach { ‘{0:x2}’ -f $_ });$stream.Close();$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Location' -Value $choice[0] -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Size' -Value $itemsize -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Date Modified' -Value $itemDate -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Hash' -Value $Hash -PassThru;$objects += $object}};Write-Host 'Source Path:'' $inputsource;Write-Host 'Destination Path:' $inputDestination;write-output $objects | format-table -autosize;'End of Deployment: '' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName};appimpl”
```

<span style="color:#993366">**Some more enhancement**</span>

The ending code above will actually be enough for you to run the program. But running it will show the below screenshot(which is very untidy)

<img src="{{ site.baseurl }}/images/powershellugly-666x350.jpg" alt="">


To make the program more presentable, add a <span style="color:red">@echo off</span> as the 1st line of the batch file and now the program will look more presentable as shown below

<img src="{{ site.baseurl }}/images/powershelltidy.jpg" alt="">

So below will be my final code saved in batch file:<br>

```shell_session
@echo off
powershell.exe -command “function Appimpl {;$filename = @();$fullfilename= @();$objects=@();do {;$input = Read-host ‘Input the filename [Hit Enter for last value]’;if ([string]::IsNullOrEmpty($input)){}else{$filename += $input};} until ([string]::IsNullOrEmpty($input));$inputsource = Read-host ‘Input the source path ‘;$inputDestination = Read-host ‘Input the destination path ‘;’Start of Deployment: ‘ +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName;ForEach ($item in $filename) {;Copy-Item -Path $inputsource\\$item -force -Destination $inputDestination;$SourceOrDestinations = @((‘Source’, $inputsource),(‘Destination’,$inputDestination));ForEach($choice in $SourceOrDestinations){;$object = New-Object PSObject -Property @{Path = $item};$fullfilepath=$choice[1];$itemsize = (Get-Item \\"$fullfilepath\\\\$item\\").length;$itemdate = (Get-Item \\"$fullfilepath\\\\$item\\").LastWriteTime;$stream = ([IO.StreamReader]\\"$fullfilepath\\\\$item\\").BaseStream;[string]$hash = -join ([Security.Cryptography.HashAlgorithm]::Create( ‘MD5’ ).ComputeHash( $stream ) | ForEach { ‘{0:x2}’ -f $_ });$stream.Close();$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Location' -Value $choice[0] -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Size' -Value $itemsize -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Date Modified' -Value $itemDate -PassThru;$object = Add-Member -InputObject $Object -MemberType NoteProperty -Name 'Hash' -Value $Hash -PassThru;$objects += $object}};Write-Host 'Source Path:'' $inputsource;Write-Host 'Destination Path:' $inputDestination;write-output $objects | format-table -autosize;'End of Deployment: '' +(get-date -format F).ToString() + [string][TimeZoneInfo]::Local.DisplayName};appimpl”
```

<span style="color:#993366">**Do User Acceptance Test**</span>

<img src="{{ site.baseurl }}/images/finaloutput.jpg" alt="">


