---
title: "Searching For Phrases in Base64-encoded Strings"
date: 2017-07-27T22:55:14-05:00
draft: false
aliases:
    - /2017/07/27/searching-for-phrases-in-base64-encoded-strings/
categories:
    - Guides
---

If you’re well-versed in concepts like base64-encoding, code obfuscation, and malware detection, you’re free to skip down to the good part (literally the section title “The Good Part”). The short version is: You can perform searches for plaintext strings encoded in base64. I’m personally very excited about this.
<!--more-->

For the less-experienced, or the more avid readers of the previous category, please read on.

With my focus on web-based security, I consider myself fortunate that traditional endpoint malware, binaries built from compiled languages like C, are outside of my typical scope. I have an incredible respect for researchers in the trenches reverse-engineering local Windows infections, and it’s a skillset I hope to breach into in the future.

That being said, my world of interpreted web-focused languages like PHP and JavaScript isn’t without its challenges. Since the behavior of these scripts can be identified by directly reading the files themselves, basic malware scanning can locate an impressive amount of activity just by searching within file contents. For example, if visitors to your website are getting redirected to evildomain.com, then a search of your site’s files and database for “evildomain.com” is fairly likely to turn up the culprit.

One reason (among several) that I qualify the previous statement with “fairly likely” is because the individuals developing web-based infections are turning, more and more commonly, to obfuscating their code. There are a plethora of methods to accomplishing this, the end result simply being code that a human couldn’t be expected to read.

The most common technique used in PHP-based malware is to base64-encode the intended payload, then direct the parser to decode and evaluate the string. Let’s look at a super-basic example of what I’m talking about before we move on to the meat and potatoes of this report.

## The (Really, Really Basic) Scenario

So I put my hacker hat on, find a site that doesn’t appear to be updated frequently and is unlikely to notice a defacement for a while. I find and exploit a vulnerability in the site, and can now write to their site’s files. If I wanted to insert a really simple defacement into the website, let’s say a basic tag reading “HACKED BY HEYITSMIKEYV”,  the code I’d want to execute would look like this:

{{<highlight php>}}echo "HACKED BY HEYITSMIKEYV";{{</highlight>}}

However, following an incredibly busy week of defacing site after site, malware scanners catch on to me. Now they’ve got a signature for my defacement, and can undo all of my hard work automatically (it wasn’t a very hard thing to find, after all). I’m not a very creative hacker, and I really don’t want to change the behavior of my script. So I turn to some basic obfuscation. My previous code, base64-encoded, becomes `ZWNobyAiSEFDS0VEIEJZIEhFWUlUU01JS0VZViI7`.  Now it’s just a matter of telling PHP to execute whatever gets spat out by decoding that string:

{{<highlight php>}}eval(base64_decode('ZWNobyAiSEFDS0VEIEJZIEhFWUlUU01JS0VZViI7'));{{</highlight>}}

*(Note: Yes, scanners do scan for `eval(base64_decode(`. There are countless ways to further obfuscate the decoding and execution of base64 strings, so I’m using the most basic form of this for illustrative purposes).*

With my code updated, scanners that are simply searching for the string `HEYITSMIKEYV` are going to turn up empty-handed, because that string doesn’t exist in my code anymore, despite having no trouble displaying it.

Taken a few steps further, what happens when the “good guys” inevitably find my new code? They’ll just add a signature for `ZWNobyAiSEFDS0VEIEJZIEhFWUlUU01JS0VZViI7` and I’m in the exact same boat.

Unless, of course, I’m able to add some arbitrary, randomized padding to the payload every time it’s deployed. This is easily accomplished using comment padding:

{{<highlight php>}}
// The following line is randomized on every victim's site:
// g987agghpaog9-8yb;lbh
echo "HACKED BY HEYITSMIKEYV";
{{</highlight>}}
This code, obfuscated in the same manner as above, becomes:

{{<highlight php>}}
eval(base64_decode('Ly8gVGhlIGZvbGxvd2luZyBsaW5lIGlzIHJhbmRvbWl6ZWQgb24gZXZlcnkgdmljdGltJ3Mgc2l0ZToKLy8gZzk4N2FnZ2hwYW9nOS04eWI7bGJoCmVjaG8gIkhBQ0tFRCBCWSBIRVlJVFNNSUtFWVYiOw=='));
{{</highlight>}}
Now, with a constantly-changing pattern of base64-encoding, it becomes a much different task to find my particular defacement.

## A Fairly-Quick Primer On Base64

Base64, if you didn’t pop over to the Wikipedia link from earlier in this article, is simply another numbering system comparable to decimal (base 10), binary (base 2), hexadecimal (base 16), etc. Where base 10 has a total of ten possible “values” of a digit (0-9), base64 has (you guessed it) sixty-four. These are comprised of:

- Numerals (`0-9`)
- Lowercase letters (`a-z`)
- Uppercase letters (`A-Z`)
- Forward slash (`/`)
- Plus symbol (`+`)

Following these characters may be up to two equals signs (`=`), used for padding, depending on how many characters were input.

When encoding a standard string of characters into base64, the translation effectively becomes “Three characters in, four characters out.” Because of this, it helps to think of your input string being broken into three-character chunks. If there are any characters left over at the end that don’t fit into a triad, equals-sign padding keeps things consistent.

With strings that are evenly divisible by three characters in length, it’s a pretty clean translation. For instance, in the diagram below, we encode the string `foo` in base64, and the result is `Zm9v`.

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 100px;">
<tbody>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
</tr>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">v</td>
</tr>
</tbody>
</table>
{{</raw>}}

And what happens to “bar”?

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 100px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">y</td>
</tr>
</tbody>
</table>
{{</raw>}}

When we combine them into the string `foobar`, the behavior is predictable. Because this string is also divisible by three characters, the output is literally the result of combining both base64-encoded strings directly.

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 200px;">
<tbody>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
</tr>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">y</td>
</tr>
</tbody>
</table>
{{</raw>}}

When the position of the input triads are swapped, so too are the output blocks, as we can see with `barfoo`:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 200px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">v</td>
</tr>
</tbody>
</table>
{{</raw>}}

## The Challenge Of Searching Within Base64 Blocks

So if I’ve got a huge block of base64-encoded gibberish, and I wanted to search for the phrase `foobar` in it without decoding it all first, I could just search for `Zm9vYmFy`, right? Well, sort of. That particular string will only occur if the first character in my search term also happens to be the first character in a triad. If the base64 input was `@foobar` instead, we end up with a much different output, `QGZvb2Jhcg==`.

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Q</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">G</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">2</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">h</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">c</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">g</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
</tr>
</tbody>
</table>
{{</raw>}}

Now that our search string `foobar` is broken into triads differently (`@fo`, `oba`, and `r`), we no longer see the same patterns as before. Similarly, with another character prepended we see a third pattern:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Q</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">B</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">2</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">i</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">X</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
</tr>
</tbody>
</table>
{{</raw>}}

Here comes the useful part: If we add one more character before our target string, we’ve pushed the first character of our search into the first position of a triad once more. This means the same patterns as before start to come back:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">@</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Q</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">B</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">A</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
</tr>
</tbody>
</table>
{{</raw>}}

`Zm9vYmFy` is back! Awesome! Now we can see that there are three key “versions” a string can be encoded into, based on if the string begins in the first, second, or third position of a triad.

### Unfortunately...

In our previous examples, we were dealing only with characters preceding our target term `foobar`, with no characters following it. When we’re dealing with larger bodies of code though, it’s unlikely that our term appears at the absolute end of the file in every case. When dealing with base64, even whitespace characters like spaces, tabs, and newlines will all be encoded differently. In other words, we can’t rely on the final triad of our string to remain predictable unless it’s completely filled with our target term. Let’s look at some new examples. In these, we’ll add a single character before `foobar`, and pad out the end of the final triad.

Using capital “Z”s, we get:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Z</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">W</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">2</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">h</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">c</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">p</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">a</td>
</tr>
</tbody>
</table>
{{</raw>}}

With ampersands (&)…

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">&amp;</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">&amp;</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">&amp;</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">2</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">h</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">c</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">i</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">m</td>
</tr>
</tbody>
</table>
{{</raw>}}

And finally with spaces:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">f</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">o</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">a</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">r</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">G</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">v</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">b</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">2</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">h</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">c</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">i</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">A</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">g</td>
</tr>
</tbody>
</table>
{{</raw>}}

So while the first and last triads get changed around based on the characters preceding and following the search string, the middle triad stays static. You may have also noticed that the pattern `Zv` in the output of the first triad doesn’t change between iterations. While true, including this pattern in searches may turn up false positives, as there are many different input strings that can result in an output block ending with `Zv`. It’s best for our key patterns (which we’ll get to shortly) to stick to complete four-character output blocks.

## The Good Part

To recap, we’ve confirmed that a string, encoded into base64, has three possible output patterns depending on where it was located in the input. We also know that internal triads of characters will always return predictable groups of four output characters regardless of what comes before and after them. With this in mind, let’s apply this to our earlier scenario.

My super-cool hacker handle, `HEYITSMIKEYV` will be encoded into base64 differently depending on the number of characters leading up to it in the code. But there are really only three ways it can appear, and we can identify those by doing our own padding, and ignoring incomplete triads. To keep things clear, I’ll mark which triads we’re keeping, and which we’re dropping.

First, let’s encode it with no padding, so the first triad starts with the first character in the string:


{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 400px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">H</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">V</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">L</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">W</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
</tr>
</tbody>
</table>
{{</raw>}}

Next we add a single character before `HEYITSMIKEYV`. We’ll use underscores. Note that adding a single character to a set of four complete triads means we now have a fifth, incomplete one.

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 500px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">_</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">H</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">X</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">0</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">h</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">W</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">0</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">1</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">0</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">g</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">DROP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">DROP</td>
</tr>
</tbody>
</table>
{{</raw>}}

And finally, with two characters of padding:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 500px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">_</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">_</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">H</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4"></td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">X</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">1</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">9</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">N</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">N</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">t</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">W</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Y</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">=</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">DROP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="12">KEEP</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="12">DROP</td>
</tr>
</tbody>
</table>
{{</raw>}}

Once we drop our incomplete groups of characters, we end up with three complete sets:

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 400px;">
<tbody>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">H</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">V</td>
</tr>
<tr>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">E</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">Z</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">L</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">W</td>
</tr>
</tbody>
</table>
{{</raw>}}

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
</tr>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">W</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">0</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">1</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">0</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">Z</td>
</tr>
</tbody>
</table>
{{</raw>}}

{{<raw>}}
<table style="font-family: monospace; margin: auto; text-align: center; height: 60px; width: 300px;">
<tbody>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">Y</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">T</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">S</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="4">M</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">I</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">K</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="4">E</td>
</tr>
<tr>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">R</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">l</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">J</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">V</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">F</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">N</td>
<td style="background-color: #4e4e4e; color: #a9a9b3;" colspan="3">N</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">S</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">U</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">t</td>
<td style="background-color: #383838; color: #a9a9b3;" colspan="3">F</td>
</tr>
</tbody>
</table>
{{</raw>}}

Cleaned up for legibility, we’re left with the following key strings:

- `SEVZSVRTTUlLRVlW`
- `WUlUU01JS0VZ`
- `RVlJVFNNSUtF`

### Why Is This Important?

With those three key strings, we now have the ability to find `HEYITSMIKEYV` anywhere in a base64-encoded block of code, regardless of size or position. Reviewing our first example of obfuscated code, we produced:

{{<highlight php>}}
eval(base64_decode('ZWNobyAiSEFDS0VEIEJZIEhFWUlUU01JS0VZViI7'));
            // Note matching key string --- WUlUU01JS0VZ                                        
{{</highlight>}}

Our second example, with a little bit of extra “entropy” added, was:

{{<highlight php>}}
eval(base64_decode('Ly8gVGhlIGZvbGxvd2luZyBsaW5lIGlzIHJhbmRvbWl6ZWQgb24gZXZlcnkgdmljdGltJ3Mgc2l0ZToKLy8gZzk4N2FnZ2hwYW9nOS04eWI7bGJoCmVjaG8gIkhBQ0tFRCBCWSBIRVlJVFNNSUtFWVYiOw=='));
            // Note matching key string ------------------------------------------------------------------------------------------------------------------- RVlJVFNNSUtF 
{{</highlight>}}

## Generating Key Strings

As a huge proponent of “automate the boring stuff”, and finding the process of manually counting and dropping character blocks firmly in the “boring” category, I wrote a Python script to handle things for us.

[You can find the script on my GitHub.](https://github.com/heyitsmikeyv/base64-keystrings)

We can use it to tackle an example problem: Finding a string in an obfuscated PHP file.

Our company’s imaginary website is serving malicious javascript to our users! The script is being sourced from `evildomain.com/malware.js`. We’re running a pretty big framework so it’s not trivial to identify the file it’s coming from, and searching our files for that address turned up empty. Let’s see if it’s been base64-encoded.

{{<highlight bash>}}
$ base64-keystrings.py "evildomain.com/malware.js"
ZXZpbGRvbWFpbi5jb20vbWFsd2FyZS5q
aWxkb21haW4uY29tL21hbHdhcmUu
dmlsZG9tYWluLmNvbS9tYWx3YXJlLmpz

$ grep -ro -e "ZXZpbGRvbWFpbi5jb20vbWFsd2FyZS5q" -e "aWxkb21haW4uY29tL21hbHdhcmUu" -e "dmlsZG9tYWluLmNvbS9tYWx3YXJlLmpz" public_html/
./public_html/includes/media/badfile.php:dmlsZG9tYWluLmNvbS9tYWx3YXJlLmpz
{{</highlight>}}

Searching for our three key strings in `public_html/` turned up the culprit: `./public_html/includes/media/badfile.php`!

{{<highlight php>}}
<?php
/* 
 * badfile.php
 * Totally Legit PHP File
 * Version 1.1
 * 
 * Definitely necessary for your website to run. Don't touch it.
 */

 // This file is encoded for copyright protection.
 // Don't decode it or I'll sue you.
 eval(base64_decode("Ly8gSGFoYSB0aGV5J2xsIG5ldmVyIGZpbmQgbWUhCmhhY2tpbmdNYWluZnJhbWUoKTsKaW5qZWN0aW5nQ29kZSgpOwpicm93c2VyVGFrZW92ZXIoJ2h0dHA6Ly9ldmlsZG9tYWluLmNvbS9tYWx3YXJlLmpzJyk7"));
{{</highlight>}}

### Caveats

This obviously isn’t a silver bullet. There are a great deal of ways to circumvent this, like running the base64 through a rot13 cipher before execution, or gzcompressing the payload before saving it to a file, et cetera. However, a great deal of attackers fail to employ these evasions, meaning this technique still holds merit.

## TL;DR

When a string is encoded as part of a larger base64 block, there are three unique key strings that can be used to identify it when encoded. These can be used to search a website’s filesystem for specific base64-encoded strings in the event of malware infection. These strings are also useful when creating detection signatures for malware scanners, since one of the three strings will always match regardless of the content surrounding the search term in its file.

In case you missed the link, you can download base64-keystrings.py from my [GitHub](https://github.com/heyitsmikeyv/base64-keystrings).

Thank you so much for reading! 