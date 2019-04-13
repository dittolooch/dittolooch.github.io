---
layout: post
title: Vocabulary Highlighter: a Chrome Extension
excerpt_separator: <!--more-->
tags: [javascript, chrome, extension]
---

![Chrome](https://user-images.githubusercontent.com/44837996/55928618-fbe67d80-5c4b-11e9-938a-4b21e064155f.jpg)

> Vocabulary Highlighter is a Google Chrome Extension that automatically highlights high-frequency GRE words as you browse the web.

## Why build Vocabulary Highlighter?
When I was preparing for the [GRE](https://www.ets.org/gre), I knew that memorising a decent number of GRE level vocabulary is required in order to do well in the verbal section, especially for someone whose first language is not English. Among so many exam preparation service providers, [Magoosh](https://gre.magoosh.com/) is undoubtedly one of the most popular, because of its practical tips, easy-to-use UI, and most importantly, its affordability, and they also have [Magoosh's GRE FlashCard](https://gre.magoosh.com/flashcards/vocabulary) freely available to anyone who wants to expand their vocabulary base and improve their GRE verbal score.

After memorising all 1000 words in the app, I tried to read as much The Economist as possible to not only review the new vocabulary I have learned, but also to familiarise myself with long and difficult passages common in GRE's verbal section, and therefore I found [Magoosh GRE Vocabulary Highlighter](https://chrome.google.com/webstore/detail/magoosh-gre-vocab-highlig/hlkndiknofmlmajfocifccmplknafjeo?hl=en), a Chrome extension that highlights the 1000 words included in the aforementioned app.

The extension works but <u>you have to click the extension icon to highlight</u> the words whenever you load a new page. Also, for websites that are designed to allow continuous scrolling to dynamically load new contents (like Facebook's newsfeed), you'll have to click the icon for <strong>as many times as new contents show up</strong>.

When I am on a 2-hour reading spree, I would like to have the extension automatically highlight the words as I browse the web, and thus, I figured I could build this simple extension from scratch and make it work in the way I want.

## Getting Started

Google usually has very detailed and useful explanations for all of its services and APIs. Although some of the references might be outdated (e.g. [this Gmail API reference](https://developers.google.com/gmail/api/guides/sending) that still uses Python 2), most are pretty reliable and easy to understand and implement.
I followed the instructions from [Google's Tutorial](https://developer.chrome.com/extensions/getstarted) and had the boilerplate ready to start with.


## Overview of the Extension

<strong>Highlighting:</strong>

The extension should execute the highlighting codes in every Chrome process (tab)

<strong>Switch on/off:</strong>

The extension should allow users to turn on/off the highlighting by clicking a radio button in the window that pops up when users click the extension icon next to the URL bar.

<strong>Select Vocabulary List:</strong>

The extension should allow users to select which vocabulary list to use for highlighting. For starters, the extension only supports Magoosh's 1000-word list and the vocabulary list popular among Chinese GRE takers (Hong Bao Shu, which literally means The Ruby Book, includes about 6500 words).

## Implementation: Finding Words to be Highlighted

A naive implementation is of course to scan every word in every text node in the <body> tag and for each word you scan the selected word list to check whether the word exists in the list. If there are <strong>n</strong> words in the content to be scanned and <strong>m</strong> words in the selected word list.This method would be running at <strong>O(n*m)</strong>, which is far from ideal.

Since the word list is static, it can be sorted first and then be broken down into 26 arrays to be stored in an object, of which keys are the leading alphabets of the words in the array.

Example:

{% highlight js %}

let wordlist = {
  'a':['abandon',...],
  'b':['bag',...],
  ...
}

{% endhighlight %}

With this wordlist object, the search can be performed more efficiently by taking the leading alphabet of the word to retrieve the array from the object, and then use the binary search algorithm to find the word. This implementation can check the existence of a word in the word list at <strong>O(n*log(m))</strong>

## Implementation: Updating DOM with Highlights

Initially I thought I could just take the ```body```'s innerHTML replace the <strong>keyword</strong> with ```<mark>keyword</mark>``` and boy I was wrong. It would kill all the event listeners attached to the descendants of ```body```. Neither does replacing textContent of the ```body``` work; it will mess up with the whole html structure.

Eventually I chose to walk the descendants of ```body``` and execute the highlighting one DOM at a time:


{% highlight js %}

function textNodesUnder(el){
  var n, a=[], walk=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null,false);
  while(n=walk.nextNode()){
      if (n.tagName!="SCRIPT"){
        a.push(n);
      }
  }
  return a;
}

{% endhighlight %}

Then we can scan every word in the text node without worrying about HTML tags because the value of each text node is its pure text content.

{% highlight js %}

function highlight_text_node(node, found_words){
    node_text_value = node.nodeValue
    found_words.forEach(function(word){
      node_text_value = node_text_value.replace(
      word,`<span class="highlighter highlight-on">${word}</span>`)
    })
    return node_text_value

}

{% endhighlight %}

Finally we can replace the text node content of the innerHTML with the highlighted text node content produced by the ```function highlight_text_node```.

{% highlight js %}

updated_text_node_value = highlight_text_node(node,found_words)
node.parentElement.innerHTML =
node.parentElement.innerHTML.replace(
    node.nodeValue,
    updated_text_node_value
)

{% endhighlight %}

This implementation still might destroy the web page's structure if there are DOMs with only one word that happen to be in the selected word list.

For example:

```<a class="grandiloquent">grandiloquent</a>```



The text node of this <strong>a</strong> tag is <strong>grandiloquent</strong>, so <strong>function highlight_text_node</strong> will produce:

```<span class="highlighter highlight-on">grandiloquent</span>```

and then the replacement implementation will find 'grandiloquent' in the innerHTML of this ```a``` tag and replace it with the ```<span>``` tag above, which means the class name, which happens to include this particular GRE word, will be replaced as well.


Nevertheless I do not worry too much about this potential bug because of the obscure nature of GRE words. Who would use "grandiloquent" as an attribute of an html tag anyway?


## Final Words

I never really learned much about JavaScript so the code definitely looks messy. Having decided to build this extension at the spur of the moment, I just wanted to hack it up and get it to run with minimal external packages and minimal time.

Quizlet has a free API that allows developers to get any Quizlet User's vocabulary set. It would be a nice improvement of functionality if you can build up your Quizlet sets over time and then the Vocabulary Highlighter is able to sync with the sets and help you identify the words you recently added to your Quizlet sets.

That's it. Go ahead and read [Google's Tutorial](https://developer.chrome.com/extensions/getstarted), and build your own small extension for fun.
