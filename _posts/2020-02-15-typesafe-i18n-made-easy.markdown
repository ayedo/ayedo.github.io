---
layout: post
title:  "Typesafe i18n made easy (with Kotlin)"
date:   2020-02-15 19:22:00
categories: internationalization
---

A missing translation, while technically a benign problem, is also a usually visible, and embarrassing one. You do not want your customers to believe that the sloppiness of forgetting a translation is indicative of your product's general quality.

I live and work in Switzerland, a country with no less than four official languages. Here a large portion of public projects will involve internationalization in some form. Even in this context, in my personal experience, it regularly happens that translations are missing on live production systems. I was wondering if maybe we are not using the right tool for the job?

I have been programming in Kotlin a lot lately, and I've discovered that its conciseness may allow for a safer internationalization experience. One where the compiler supports the effort.

We first start out by defining the following interface:

{% highlight kotlin %}
interface Translatable {
    fun getEn(): String
    fun getDe(): String
}
{% endhighlight %}

Nothing fancy yet: a Translatable has a function to return a String for each supported language.

Now two features of Kotlin comes into to play: 

1. built-in language support for singletons
2. the possibility of declaring multiple classes or objects in a single file. 

In a single file we start defining instances of Translatable:

{% highlight kotlin %}
object Greeting : Translatable {
    override fun getEn() = "Hello"
    override fun getDe() = "Hallo"
}

object ClickHereToRestorePassword : Translatable {
    override fun getEn() = "Click here to restore your password."
    override fun getDe() = "Klicke hier, um Dein Passwort neu zu setzen."
}
{% endhighlight %}

What I like about this is:
- if we add a language the code won't compile until we define the corresponding function for all instances of Translatable
- the translations are next to each other in a single file, not spread across multiple files which require context switches to compare. Mistakes are more obvious that way.

Using these Translatable objects is not yet ergonomic: when we want to translate we have to invoke the correct function depending on the desired language. 

Further, whatever language it might be, one usually wants to have all translations for the same one. It's helpful to create an object to store a language and invoke the corresponding language function for you:

{% highlight kotlin %}
object Translator {

    private fun translate(translatable: Translatable, language: Language): String =
        when (language) {
            EN -> translatable.getEn()
            DE -> translatable.getDe()
        }
    
    fun forLanguage(language: Language) = { translatable: Translatable ->
        translate(translatable, language)
    }

}
{% endhighlight %}

This is how it's used:

{% highlight kotlin %}
val translator = Translator.forLanguage(Language.EN)

val message = 
"""
${translator(Greeting)},
${translator(ClickHereToRestorePassword)}
"""
{% endhighlight %}

One more thing that has popped up: how do you support Strings which require a variable to be placed in the middle of it?

One way to do it would be to define a class instead of a singleton object:

{% highlight kotlin %}
class YouHaveBookedTheCourse(private val courseName: String) : Translatable {
    override fun getEn() =
        "You have booked the course \'$courseName\'"

    override fun getDe() =
        "Du hast dich für den Kurs \'$courseName\' angemeldet."
}
{% endhighlight %}

What are the disadvantages of the approach? The ones I found:
- possibly difficult for non-programmers (e.g. translators)
- it requires one object per Translatable

There is one more feature of Kotlin that plays nicely into this: Kotlin also compiles to Javascript. This means that the translations defined in this way can be shared across the front-, and backend code.

What do you think? I'm looking forward to your input. Discussion on <a href="https://www.reddit.com/r/programming/comments/f6hewi/typesafe_i18n_made_easy_with_kotlin/">reddit</a> and <a  href="https://news.ycombinator.com/item?id=22370096">hackernews</a>.

Working example code can be found on [GitHub](https://github.com/ayedo/typesafei18n)