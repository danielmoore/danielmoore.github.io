---
id: 299
title: Alpha-Blending Colors in PowerShell
date: 2010-08-22T13:40:07+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=299
permalink: /2010/alpha-blending-colors-in-powershell/
categories:
  - Coding
  - Lab49
tags:
  - colors
  - pipelining
  - powershell
---
The other day I was given the task of converting a particularly poorly designed VisualBrush into a LinearGradientBrush. One of the problems I came across very quickly was the use of semi-transparent colors layered on top of each other, and, of course, I needed a &#8220;flattened&#8221; color for my GradientStop. Now, I could have used Paint.NET or GIMP or Photoshop to put out a couple layers of colors, set the transparencies and used the color dropper to get the result. Of course, since I&#8217;m not a designer, I don&#8217;t have any of those things installed on my work computer, so I decided to just find the equation to blend the channels myself. It didn&#8217;t take long, and Wikipedia delivered the goods. According to [[Alpha\_compositing|the article]], the formula to merge two colors, [latex]C\_a[/latex] and [latex]C\_b[/latex], into some output color, [latex]C\_o[/latex], looks like this:

<p style="text-align: center;">
  [latex]C_o = C_a\alpha_a+C_b\alpha_b(1-\alpha_a)[/latex]
</p>

Since a color can be thought of as a three-tuple of its R, G, and B channels, the formula is easily distributed to each of these values.

At this point, I decided I could probably just pull out a calculator and crunch the numbers. But maybe, in about the same time, I could also whip something together, say in PowerShell, to do it for me. Since I&#8217;m still learning PowerShell, I figured the learning experience would be worth at least something.<!--more-->

So the first thing I needed was the ability to parse an ARGB hex code into a hashtable with each channel separated out. Here&#8217;s what I got.

<pre class="brush: powershell">function Blend-Colors ([string[]] $colors) {
    # $argbHexColorRegex should recognize all 4-byte hex color strings prefixed with a '#'
    # and assign them to groups named a, r, g, and b for each channel, respectively.
    $argbHexColorRegex = "(?i)^#(?'a'[0-9A-F]{2})(?'r'[0-9A-F]{2})(?'g'[0-9A-F]{2})(?'b'[0-9A-F]{2})$"

    foreach($colorHex in $colors) {
        if($colorHex -match $argbHexColorRegex) {
            $color = @{
                a = [int]::Parse($matches.a, 'AllowHexSpecifier')
                r = [int]::Parse($matches.r, 'AllowHexSpecifier');
                g = [int]::Parse($matches.g, 'AllowHexSpecifier');
                b = [int]::Parse($matches.b, 'AllowHexSpecifier');
            }

            $color | ft
        } else {
            throw "Invalid color: $colorHex"
        }
    }
}</pre>

Ok, not too shabby. If you&#8217;re wondering where `$matches` came from, as a side effect of the `-match` operation, the `$matches` hashtable is set with all the matched groups defined. I just prefer to use the object syntax in this case over the index syntax.

So, I&#8217;m not real happy with the amount of repetition going on in the assignment of my hashtable. My first attempt to clean it up looked like this:

<pre class="brush: powershell; gutter: false">$color = ('a','r','g','b') | % { @{ $_ = [int]::Parse($matches.$_, 'AllowHexSpecifier') } }</pre>

But that just gave me an array of hashtables with one attribute in each. I asked [Doug Finke](http://www.dougfinke.com/) what he thought and he recommended this modification:

<pre class="brush: powershell; gutter: false">('a','r','g','b') | % { $color = @{} } { $color.$_ = [int]::Parse($matches.$_, 'AllowHexSpecifier') }</pre>

Cool! It seems a little obvious now, though.

Next on the agenda was to translate the blending equation into PowerShell. Since the equation is a little more involved, I decided to abstract it out into a subfunction like so:

<pre class="brush: powershell">function Merge-Channel ($c0, $c1, $c) {
	$a0 = $c0.a / 255
	$a1 = $c1.a / 255
	return $a0 * $c0.$c + $a1 * $c1.$c * (1 - $a0)
}</pre>

`$c0` and `$c1` are color hashtables and `$c` is the name of the channel to blend. I had to divide the alpha channels by 255 to produce a value compatible with the equation, namely, between 0 and 1.

The reason I chose to accept the entire color and desired channel, rather than a more terse definition accepting the specific channel values and related alpha values was to make calling the code a little more elegant:

<pre class="brush: powershell; gutter: false">('r','g','b') | % `
	{ $mergeColor = @{ a = [Math]::Min(255, $outColor.a + $addColor.a) } } `
	{ $mergeColor.$_ = Merge-Channel $addColor $outColor $_ }</pre>

When blending, the alpha channels simply sum, so I put that in my hashtable initializer and just iterated over the color channels.

Now the final step is to return our value back in hex form. Fortunately, the formatting styles for `int` make this really easy:

<pre class="brush: powershell; gutter: false">return '#{0:x2}{1:x2}{2:x2}{3:x2}' -f (('a','r','g','b') | % { [int][Math]::Round($baseColor.$_, 0) })</pre>

I&#8217;m rounding to ensure the highest accuracy to the blended color, as opposed to simply truncating.

Putting it all together, I decided to create two array constants, `$argb` and `$rgb` to alias arrays of the channels. While I was at it, I also promoted my `$argbHexColorRegex` to a constant just for good measure. Finally, I made the base color white, so there would be something to blend against. The result looks like this:

<pre class="brush: powershell">function Blend-Colors ([Parameter(Mandatory=$true)] [string[]] $colors) {
    # $argbHexColorRegex should recognize all 4-byte hex color strings prefixed with a '#'
    # and assign them to groups named a, r, g, and b for each channel, respectively.
    Set-Variable argbHexColorRegex -Option Constant `
        -Value "(?i)^#(?'a'[0-9A-F]{2})(?'r'[0-9A-F]{2})(?'g'[0-9A-F]{2})(?'b'[0-9A-F]{2})$"
    Set-Variable argb -Option Constant -Value 'a','r','g','b'
    Set-Variable rgb -Option Constant -Value 'r','g','b'

    function Merge-Channel ($c0, $c1, $c) {
        $a0 = $c0.a / 255
        $a1 = $c1.a / 255
        return $a0 * $c0.$c + $a1 * $c1.$c * (1 - $a0)
    }

    $argb | % { $outColor = @{} } { $outColor.$_ = 255 } # set $outColor to white (#FFFFFFFF)

    foreach($color in $colors) {
        if(-not ($color -match $argbHexColorRegex)) {
            throw "Invalid color: $color"
        }

        $argb | % { $addColor = @{} } { $addColor.$_ =  [int]::Parse($matches.$_, 'AllowHexSpecifier') }

        $rgb | % `
            { $mergeColor = @{ a = [Math]::Min(255, $outColor.a + $addColor.a) } } `
            { $mergeColor.$_ = Merge-Channel $addColor $outColor $_ }

        $outColor = $mergeColor
    }

    return '#{0:x2}{1:x2}{2:x2}{3:x2}' -f ($argb | % { [int][Math]::Round($outColor.$_, 0) })
}</pre>

This is looking really good, and, really, I might have just stopped here. The only things I was missing at this point were pipelining and documentation, and since my solution had become completely over-engineered as it was, I decided I might as well go for broke.

The first thing I wanted to do was abstract out my initialization of `$outColor` to a parameter. Since I&#8217;d have to parse the string, I&#8217;d also need to abstract my color hex parser.

<pre class="brush: powershell">function Parse-Color ([string] $hex) {
	if($hex -match $argbHexColorRegex) {
		$argb | % { $color = @{} } { $color.$_ =  [int]::Parse($matches.$_, 'AllowHexSpecifier') }
		return $color;
	} else {
		return $null;
	}
}</pre>

The reason I decided to return null instead of throwing an error immediately is because I wanted to treat errors differently in both places. Specifically, if an invalid string is passed in to the `-background`, I want to throw an argument exception, and if something invalid comes in over the pipeline, I just want to write the error to the error output and keep on trucking.

I tried a few [different approaches](http://huddledmasses.org/writing-better-script-functions-for-the-powershell-pipeline/) to being able to both accept input over the parameter list and I finally found out about `[Parameter(ValueFromPipeline=$true)]`. Here is my test setup:

<pre class="brush: powershell">function Get-Range([int]$max) {
    for($i=0; $i -lt $max; $i++) {
        Write-Host "pushing $i to pipeline"
        Write-Output $i
    }
}

function Test-Pipeline([Parameter(ValueFromPipeline=$true)][int[]]$vals = $null) {
    process {
        foreach($item in @($vals)){
            Write-Host "processing $item from pipeline"
        }
    }
}</pre>

And my test output:

<pre class="brush: plain; gutter: false">&gt; Get-Range 3 | Test-Pipeline
pushing 0 to pipeline
processing 0 from pipeline
pushing 1 to pipeline
processing 1 from pipeline
pushing 2 to pipeline

&gt; Test-Pipeline ('1','2','3')
processing 1 from pipeline
processing 2 from pipeline
processing 3 from pipeline</pre>

Notice the `@($vals)` in my `foreach`? That&#8217;s to protect against null inputs by ensuring `$vals` is a list.

Now that I&#8217;ve got all my pieces together, I just need to put everything in place with a splash of [documentation](http://technet.microsoft.com/en-us/magazine/ff458353.aspx).

<pre class="brush: powershell">&lt;#
    .SYNOPSIS
    Takes a list of ARGB hex values and blends them in order against a specified background.

    .PARAMETER background
    The background color to blend against, defaults to white.

    .PARAMETER colors
    A list of ARGB hex color strings, can be pushed from the pipeline.

    .EXAMPLE
    Blend-Colors '#ff121212', '#705F6A87'

    .LINK
    http://en.wikipedia.org/wiki/Alpha_compositing
#&gt;
function Blend-Colors (
    [string] $background = '#FFFFFFFF',
    [Parameter(ValueFromPipeline = $true)] [string[]] $colors = $null) {
    begin {
        # $argbHexColorRegex should recognize all 4-byte hex color strings prefixed with a '#'
        # and assign them to groups named a, r, g, and b for each channel, respectively.
        Set-Variable argbHexColorRegex -Option Constant `
            -Value "(?i)^#(?'a'[0-9A-F]{2})(?'r'[0-9A-F]{2})(?'g'[0-9A-F]{2})(?'b'[0-9A-F]{2})$"

        Set-Variable argb -Option Constant -Value 'a','r','g','b'
        Set-Variable rgb -Option Constant -Value 'r','g','b'

        function Parse-Color ([string] $hex) {
            if($hex -match $argbHexColorRegex) {
                $argb | % { $color = @{} } { $color.$_ =  [int]::Parse($matches.$_, 'AllowHexSpecifier') }
                return $color;
            } else {
                return $null;
            }
        }

        function Merge-Channel ($c0, $c1, $c) {
            $a0 = $c0.a / 255
            $a1 = $c1.a / 255
            return $a0 * $c0.$c + $a1 * $c1.$c * (1 - $a0)
        }

        $outColor = Parse-Color $background
        if(-not $outColor) {
            throw (New-Object ArgumentException -ArgumentList "Invalid color: '$background'", 'background')
        }
    }
    process {
        foreach($color in @($colors)){
            $addColor = Parse-Color $color
            if(-not $addColor) {
                Write-Error "Invalid input color: $_"
                break
            }

            $rgb | % `
                { $mergeColor = @{ a = [Math]::Min(255, $outColor.a + $addColor.a) } } `
                { $mergeColor.$_ = Merge-Channel $addColor $outColor $_ }

            $outColor = $mergeColor
        }
    }
    end {
        return '#{0:x2}{1:x2}{2:x2}{3:x2}' -f ($argb | % { [int][Math]::Round($outColor.$_, 0) })
    }
}</pre>

Now all you have to do is save it in %userprofile%\My Documents\WindowsPowerShell\Modules\UITools as UITools.psm1 and call `Import-Module UITools` to bring in this function.