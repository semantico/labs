---
layout: article
category: articles
title: Yes is no and no is yes
author: georgeb
abstract: During a regular Perl vs Ruby debate, I experimented with redefining truth in Ruby.
---

Due to Ruby's "open classes", it's fairly easy to redefine anything in the standard library,
including base classes like `true` and `false`. Unfortunately, as
[this StackOverflow answer](http://stackoverflow.com/a/2773819/243949) touches on,
some of the logic around `true` and `false` is baked into the language and so unmodifiable
(at least in MRI - I'm yet to try with Rubinius et al). I had a play in [Pry](http://pryrepl.org/)
and managed to redefine `==` on true and false, which leads to some nice weirdness,
but probably wouldn't affect the behavior of control statements like `if` (which use the
built in truthiness).

Check out the asciicast below:

<script type="text/javascript" src="http://ascii.io/a/3699.js" id="asciicast-3699"> </script>


