.. The overview file describes the purpose of the framework
   Added: 2015-12-10
   Author: Gauthier Delamarre <gauthier@objective-php.org>

========
Introduction
========

What?
========

Objective PHP is a lightweight framework written in PHP7 with OOP in mind. This is why it's called that - there is other reason, except a pun on Objective C of course :)

Is Objective PHP a fullstack framework? Is it a micro-framework? Actually, neither. We use to call it a mini-framework, meaning that it's somewhere in between: it provides the developers with much more than a micro-framework does, but also less than a fullstack.

Objective PHP aims at handling applicative workflows, then let the developer do their work. No more, no less.

For higher level components, like Forms generators or ORMs for instance, we thought that it would be more efficient to let developers bring their usual tools in Objective rather than forcing them to use our own alternatives.


Why?
========

You may ask yourself: why did those guys bothered with another PHP framework? The answer is quite simple: we're bored with spending more times understanding and masterizing the framework itself than with working on the actual applications features.

**productivity** is the main target of Objective PHP

On top of that, we thought taht working on a new framework would also be an opportunity to consider performances in a different way. Most frameworks rely on cache to offer decent performances. Well, cache can help. A bit. But once you cached the poor performing components, what more can you do? Nothing. 

 **performance** is the second major concern of Objective PHP


How?
=========

The key idea to help developers getting efficient and comfortable with Objective PHP is to reduce as much as possible the number of different mechanism to achieve essential tasks the framework has to handle. 

After many tries, we came down to a combination of two central concepts:

- *Middlewares* to set up a workflow
- *Events* to ease the connexion between the application components



