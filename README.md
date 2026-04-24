<h1 align="center">Fireburst OS</h1>

<div align="center">
  <img src="os/assets/images/dragon-login.png" height="425" />
</div>

## Table of Contents

- [Deploy](#Deployment)
- [Configure Backend](#Backend Configuration)


## Deployment
This is a static repository and therefore can be hosted on GitHub Pages by GitHub actions. Simply navigate to [.github/workflows/deploy.yml](.github/workflows/deploy.yml) and execute [the workflow](.github/workflows/deploy.yml)! Configuration for the workflow is ```debian_mini``` ```950MB``` and
 - [x] deploy to pages 
 - [ ] update pages deployment
#### Backend Configuration
Unfortunately, due to security restricions I cannot share the backend URL that I use. It is hosted on [Google Apps Scripts](https://script.google.com) and is therefore easy to deploy.

1. Create a new Apps Script at [script.google.com](https://script.google.com)
2. Delete everything in ```Code.gs``` and replace it with this:

```Code.gs```
```gas
# code coming soon!
```
3. Next, press Deploy then input ```deploy as me (your email here)``` and ```Anyone```
4. Then, copy the URL and you are good to go!


# My Guide :)

## Heading 2

~~1234~~ 

_Italics_

**Bold Text** 
 
```Kotlin
// code snippet

 var language = "Kotlin "
 val len = language.trim().length()
```
### Links

https://kotlinlang.org/

you can find a similar repo [here](https://github.com/Ericgacoki/Markdown_Files)

or

[Click here](https://kotlinlang.org/  "Normal link")

or

[Referal link](README.md "Refers to a file within the repo")


### Username @mentions
@Ericgacoki

### Emojis

basecampy    :basecampy:

goberserk    :goberserk:

shipit       :shipit:

### Flags

pirate :pirate_flag:

crossed :crossed_flags:

checkered :checkered_flag:

rainbow :rainbow_flag:

**countries** 

Kenya :kenya:

Malaysia :malaysia:

## see more emojis [from github emoji API](https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md#github-custom-emoji "Github Emojis")

### image

![Image](https://image.shutterstock.com/image-photo/image-260nw-1418646482.jpg "sample image")

### keywords

`for` (item `in` items){ `if` ...}

### tables

|                H1      |   H2  |    H3    |
|                  ---   |  ---  |   ---    |
| :technologist: Android |  Is   |    Fun   |
|                Go      |  Try  |It :wink: |

### Blockquotes

> this is a block quote

### colors

- ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) `#f03c15`
- ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+) `#c5f015`
- ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) `#1589F0`


```diff
 colored text
 
- text in red
+ text in green
! text in orange
# text in gray
```

### Lists

1. list one 
2. list two
3. list three

- bullet1
- bullet2

4. list four

hr line1 .
___


hr line2 .

***