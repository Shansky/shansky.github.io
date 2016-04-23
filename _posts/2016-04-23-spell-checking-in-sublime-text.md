---
layout: post
title: Spell Checking in Sublime Text
---

## I need it so TIL

I thought that it'd be nice to have spell checking in sublime for writing this posts in English.
So Sublime Text uses [Hunspell](http://hunspell.sourceforge.net/) for spell checking and you need to install it in your os:
```bash
sudo apt-get install hunspell
```

#### Let's say that i want two languages: Polish and English (us & gb):
Polish dictionary can be found in [repository by titoBouzout](https://github.com/titoBouzout/Dictionaries) or you can download some from openoffice.

Create dir in ```~/.config/sublime-text-3/Packages/Language - Polish``` - for my example it was ubuntu, on mac osx it could be: ```~/Library/Application\ Support/Sublime\ Text\ 3/Packages/Language - Polish```. Put your dictionary (i want extra ```Polish.dic```) there.

After that just add those lines to your sublime text user settings:

```json
{
    "dictionary": "Packages/Language - English/en_US.dic",
    "dictionary": "Packages/Language - Polish/Polish.dic",
    "spell_check": true
}
```

To change language in sublime-text: ```View > Dictionary > Language - English > en_US```

### Source: 
 [spell_checking 4 sublime-text](https://www.sublimetext.com/docs/2/spell_checking.html)