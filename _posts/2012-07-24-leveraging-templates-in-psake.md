---
id: 510
title: Leveraging Templates in psake
date: 2012-07-24T01:15:30+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=510
permalink: /2012/leveraging-templates-in-psake/
categories:
  - Coding
  - Lab49
tags:
  - powershell
  - psake
  - templating
---
For the past few months I&#8217;ve been a technical editor for a book my good friend and colleague, [Doug Finke](http://dougfinke.com/blog), is writing entitled _[PowerShell for Developers](http://oreilly.com/catalog/0636920024491)_ which has just recently become available on [Amazon](http://www.amazon.com/Windows-PowerShell-Developers-Douglas-Finke/dp/1449322700). The purpose of the book is to show how easy it is to accomplish normally mundane, repetitive, or clunky tasks with PowerShell, a simple, concise scripting language that you already have installed on your box.

On my current project, I was recently tasked with setting up the build for our project. Of course, build scripts aren&#8217;t exactly glorious or interesting in any way and the prospect of dealing with MSBuild&#8217;s XML files gives me a fleeting sense of vertigo.

But fortunately, there&#8217;s a much, much less painful way to make build scripts by using psake.<!--more-->

## A Brief Introduction to psake

_(If you already know psake, skip to the next section!)_

[psake](https://github.com/psake/psake) (pronounced _sah-kay_) is a build scripting DSL of the [make](http://en.wikipedia.org/wiki/Make_(software)), [rake](http://rake.rubyforge.org/), [jake](https://github.com/jcoglan/jake/), and [cake](http://sourceforge.net/apps/trac/cake-build/) variety built up in PowerShell. Basically, you define tasks as small script blocks and can establish dependencies between them.

My projects tend to be organized like this:

<pre>+-MyApp\
  +-docs\
  +-src\
  | +-MyApp.sln
  +-lib\
  +-tools\
  | +-build\
  | | +-psake.psm1
  | | +-psake.ps1
  | | +-default.ps1
  +-psake.cmd</pre>

Here, `psake.psm1`, `psake.ps1`, and `psake.cmd` are from psake&#8217;s github repo, except that I modify `psake.cmd` to point to `tools\build`:

<pre>@echo off

if '%1'=='/?' goto help
if '%1'=='-help' goto help
if '%1'=='-h' goto help

powershell -NoProfile -ExecutionPolicy Bypass -Command "& '%~dp0\tools\build\psake.ps1' %*; if ($psake.build_success -eq $false) { exit 1 } else { exit 0 }"
goto :eof

:help
powershell -NoProfile -ExecutionPolicy Bypass -Command "& '%~dp0\tools\build\psake.ps1' -help"</pre>

My basic `default.ps1` psake script looks like this:

<pre class="brush:powershell">properties {
    # Since we're in tools\build, root is up two dirs.
    $rootDir = (Resolve-Path (Join-Path $psake.build_script_dir '..\..')).Path
    $srcDir = Join-Path $rootDir src
    $slnPath = Join-Path $srcDir PsakeDemo.sln

    $buildProperties = @{}

    if(-not $buildType) {
        $buildType = 'Debug'
    }

    $buildProperties.Configuration = "$buildType|AnyCPU"
}

task default -depends Clean, Compile, Test

task Clean {
    Invoke-MSBuild $slnPath 'Clean' $buildProperties
}

task Compile -depends Clean {
    Invoke-MSBuild $slnPath 'Build' $buildProperties
}

task Test -depends Compile {
    # TODO: call test framework CLI inside an `exec { ... }`
}

function Invoke-MSBuild([string]$Path, [string]$Target, [Hashtable]$Properties) {
    Write-Host "Invoking MSBuild" -Foreground Blue
    Write-Host @"
    Path       : $Path
    Target     : $Target
    Properties : $($Properties | Out-String)
"@

    $props = ($Properties.GetEnumerator() | % { $_.Key + '=' + $_.Value }) -join ';'

    exec { msbuild $Path "/t:$Target" "/p:$props" }
}</pre>

What&#8217;s great about psake is that there are really only three things you need to learn.

The first is the properties block up at the top. This guy is responsible for setting up your global state and is a good place to declare a bunch of variables that make working with your script easy to understand. As you can see, I like to declare variables for every directory path that might be needed as well as the `.sln` path. My convention here is that paths to directories are suffixed with &#8220;Dir&#8221;, paths to files are suffixed with &#8220;Path&#8221;, and all paths are always absolute. In the rare case of a need for a relative path, I would suffix the variable with &#8220;RelDir&#8221; or &#8220;RelPath&#8221; so the next guy who looks at my script knows what&#8217;s going on.

The second is `task`. This guy is the workhorse of our script; each stage of the build process should be encapsulated in a task so that we can mix and match them later on. Each task exposes out a list of other tasks that need to be done first. For instance, it&#8217;d be really problematic to run your tests before you&#8217;ve compiled! By adding the `-depends` comma-separated list of other task names, you can be sure that those tasks will be run before yours is executed, even if they&#8217;re not specified explicitly. You might also notice that I&#8217;ve defined a `default` task. This is essentially the `Main` method of our build script. While your script doesn&#8217;t _have_ to have one, you should make one so your script can be run without extra parameters. As you can see, I don&#8217;t define any work to be done in the default task, I just define what tasks should be done by default. Simple. (By the way, I could technically have just said `task default -depends Test` and that would have accomplished the same thing since Test depnds on Compile and Compile depends on Clean. I specify all three just to state my intentions.)

Finally, you need to know about `exec { ... }`. This guy runs some external process (like msbuild or your test runner) inside the braces, checks the exit code and throws an error if it doesn&#8217;t return 0. Handy!

New, at the bottom of my script I&#8217;ve added a function called `Invoke-MSBuild`. What I like about this is that now I have a nice PowerShell API for kicking off msbuild. If I need more switches, I can just add more parameters to the function. Plus, working with a `Hashtable` to set msbuild properties is so much easier than trying to construct `key1=value1;key2=value2`.

## Templating `CommonAssemblyInfo.cs`

In my `src` directory, I like to create two files and add them to my soltuion files: `CommonAssemblyInfo.cs` and `CommonAssemblyInfo.cs.template`, the idea being that the `.cs` file is the one that I reference in my projects (be sure to use &#8220;[add as link](http://msdn.microsoft.com/en-us/library/9f4t9t92.aspx)&#8220;) and the `.cs.template` file is the file that PowerShell reads teo create it. Let&#8217;s take a look at the template.

<pre class="brush:csharp">using System.Reflection;
using System.Runtime.InteropServices;

[assembly: AssemblyConfiguration("$buildType")]
[assembly: AssemblyCompany("North Horizon")]
[assembly: AssemblyProduct("PsakeDemo")]
[assembly: AssemblyCopyright("Copyright © Daniel Moore $((Get-Date).Year)")]

[assembly: ComVisible(false)]

[assembly: AssemblyVersion("$version")]
[assembly: AssemblyFileVersion("$version")]</pre>

First off, there&#8217;s no doubt how to read this file. It _is_ C#. Well, kinda. But the point is that anybody can read this file and make small changes as necessary, even if they didn&#8217;t know PowerShell or psake. Quite clearly we&#8217;re using PowerShell variables throughout (`$buildType` and `$version`) and, amazingly, we&#8217;ve got an honest-to-goodness PowerShell command in play: `$((Get-Date).Year)`. So this must require a couple KLoC framework, to do the parsing and such, yeah? Actually, it&#8217;s more like two:

<pre class="brush:powershell">properties {
    # ...
    $version = '1.0.0.0' # TODO: obtain version from build server or something.
    # ...
}

task Version {
    $templateName = 'CommonAssemblyInfo.cs.template'
    $templatePath = Join-Path $srcDir $templateName
    $outPath = Join-Path $srcDir ([IO.Path]::GetFileNameWithoutExtension($templateName))

    # &lt;1&gt;
    $content = [IO.File]::ReadAllText($templatePath, [Text.Encoding]::Default)

    # &lt;2&gt;
    Invoke-Expression "@`"`r`n$content`r`n`"@" | Out-File $outPath -Encoding utf8 -Force
}</pre>

I actually learned this technique while editing _PowerShell for Developers_ &#8211; this is just one small example from chapter 4, which goes into much greater depth about the awesome stuff you can do with templating.

So what does this do? Well, on mark `<1>`, we simply read in all the text from our template as a string using the tried-and-true `File` .NET API. I specify the encoding here so the copyright symbol gets read correctly.

On mark `<2>`, we wrap that string into a double-quote herdoc and invoke the whole thing as a script. This is how we leverage the entire PowerShell ecosystem with string interpolation and sub-expressions with almost no code.

This is, of course, just the begining &#8211; but that&#8217;s what makes PowerShell so awesome. With a few lines of code, you can leverage the whole of .NET in an intuitive and concise manner, and you&#8217;re not limited to just things the PowerShell team has thought of. PowerShell&#8217;s out-of-the-box cmdlet library makes common and often tough tasks easy, but the language provides all the building blocks you need to build anything you can imagine.

If you&#8217;re looking to get started in PowerShell or want to expand your toolbox of patterns and techniques, I can&#8217;t recommend enough that you take a look at _[PowerShell for Developers](http://oreilly.com/catalog/0636920024491)_ and see what else you can start automating.