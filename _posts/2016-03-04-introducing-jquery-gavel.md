---
layout: post
title: Introducing jQuery Gavel 
comments: true
---

*A simple jQuery validation plugin.*

I've always found it annoying how most javascript/jQuery validation plugins are either extremely hard to customize or extraordinarily bloated. Every time I needed to do something a  little more complex than the norm I was faced with having to add clunky additions, switch to another plugin or in the worst cases switch to fully custom validation. As a result I decided to make an attempt at something a little more practical.

![Vagrant Banner](/public/images/posts/gavel_banner.jpg)

##jQuery Gavel

[Gavel](https://github.com/OdinsPlasmaRifle/jquery.gavel) was born form annoyance. I can't say it is better than alternatives but it definitely was a worthwhile learning experience. It gets its job done with minimal hassle and when anything more complex is needed it has a simple set of easily extendable functions.

So, if at any point you in need a simple, very lightweight plugin give Gavel a try. It is quite easy:

```html
<script type="text/javascript" src="jquery.min.js"></script>
<script type="text/javascript" src="jquery.gavel.js"></script>
<form method="GET" name="example" id="example">
    <input name="example_input1" type="text" data-gavel data-gavel-rules="required|alphabetic"/>
    <input value="Submit" id="submit" type="submit"/>
</form>
```

Basically, you just need to indicate what fields are handled by Gavel by adding the 'data-gavel' attribute:

```html
<input name="example_input1" type="text" data-gavel/>
```

Then you can add all the validation rules you require:

```html
<input name="example_input1" type="text" data-gavel data-gavel-rules="required|alphabetic"/>
```

Finally, initiate the plugin on the form:

```javascript
$("#example").gavel();
```

The following is a list of rules available to Gavel by default:

Rule | Usage
---- | -----
alphanumeric | data-gavel-rules="alphanumeric"
numeric | data-gavel-rules="numeric"
alphabetic | data-gavel-rules="alphabetic"
email | data-gavel-rules="email"
telephone | data-gavel-rules="telephone"
date | data-gavel-rules="date"
required | data-gavel-rules="required"
match | data-gavel-rules="match[name_of_element_to_match]"
min | data-gavel-rules="min[10]"
max | data-gavel-rules="max[15]"


###Further reading:

[Gavel docs](https://github.com/OdinsPlasmaRifle/jquery.gavel/blob/master/README.md)
