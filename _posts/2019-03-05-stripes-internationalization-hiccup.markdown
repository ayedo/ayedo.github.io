---
layout: post
title:  "Stripe's internationalization hiccup"
date:   2019-03-05 19:22:00
categories: internationalization
---

This is just a reminder to all programmers who are developing applications that are supposed to be working internationally: the language the user would like to interact with your site has nothing to do with their country of residence.

There are counries which have mutliple official languages. There are expats who live in other countries too. Don't try to be too fancy here, just let people choose.

Even if you decide to have an educated guess (say by using browser provided information), always, always allow the user to change their language.

I stumpled upon this problem when I tried to integrate with Stripe's standard account subsription form for Switzerland. <a href="https://connect.stripe.com/oauth/authorize?response_type=code&client_id=ca_32D88BD1qLklliziD7gYQvctJIhWBSQ7&scope=read_write">Click here to interact with it</a>.

Try changeing the country to Germany. Now everything is in German. Now change it to Switzerland, and it's English. Why isn't it Swiss? Well, we have four official  languages here, and none of them is called Swiss. There is no way I could find to change, or configure the language.

I consulted Stripe on multiple channels about the issue, and the response I got was: there is no way to configure the language. The country defines the language. In the case of Switzerland that means: English.

So please, my mono-lingual friends, don't make it hard for us multi-lingual countries. Be nice and let us pick our favourite language!
