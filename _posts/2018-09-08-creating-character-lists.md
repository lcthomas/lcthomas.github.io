---
title: A messy and time-consuming way to create character lists using Stanford's NER tagger
author: lindsaythomas
layout: post
permalink: /2018/09/creating-character-lists/
---
## Lengthy preamble
Recently I've been trying to teach myself something about network analysis, and I ran across [this article](https://www.degruyter.com/view/j/itit.2018.60.issue-1/itit-2017-0023/itit-2017-0023.xml?intcmp=trendmd#j_itit-2017-0023_fn_003) by Markus Luczak-Roesch, Adam Grener, and Emma Fenton, called "Not-so-distant reading: A dynamic network approach to literature" ([project site](https://vuw-fair.github.io/dickens-and-data-science/)) It describes a tool for generating dynamic and static networks of character occurrences in a text using R, which I'm sort of familiar with. The authors direct readers to their [Github repo](https://github.com/vuw-sim-stia/lit-cascades), so I decided to check it out.

If you want to use the tool to model connections between characters in a novel, the software requires two inputs: a plain-text file of the novel you want to examine, and a list of the characters in that novel. More on that in a second. From those inputs, the program returns a variety of outputs, including two different network models of character co-occurrences and various analyses of these models. I encourage you to read Luczak-Roesch et al's piece for more about the tool, including descriptions of the network models. It's a pretty fun tool to experiment with, and, if you are familiar with RStudio, it's not that hard to get up and running on your own machine. (A warning, however: As with all R scripts, you might have some problems trying to install the packages you will need. One that gives lots of people trouble in particular is rJava, which the rWeka package uses. rJava issues always take me forever to figure out.)

This post, however, focuses on a method for creating character lists to use with the tool. One of the things that's always stopped me from doing anything with network analysis in the past is the sheer amount of labor involved in creating something like a social network of character co-occurrences in a novel, especially a contemporary text. I wasn't sure how best to approach the problem. Luczak-Roesch et al's tool gets you part of the way there: it defines what constitutes a "connection" between characters, and creates the networks. But in order to create networks of character appearances, you still have to provide a list of the characters in the novel. And for novels without readily available character lists, this means putting in lots of work to create such lists for each novel you want to analyze.<br/>
**Note:** The software is also designed to run in an unsupervised way -- you can create networks of verb co-occurrence in a novel, for instance -- but capturing networks of characters specifically would still require supervision. For more on using the tool in an unsupervised fashion, see Luczak-Roesch et al, pg 32.

Will the method I am about to describe save you that labor and time? Not really! But it will provide you with a starting point for that work, particularly for novels for which you cannot find a reliable or full list of characters.

The character list the tool requires is formatted simply: `A: B1, B2, B3, etc,` where `A` is the name the character will be listed as in your networks, and `B1` etc are any name that character is called in the text. The names in the `B` position, in other words, should be exact transcriptions of any name that specific character is called at any point in the text. These can have spaces, and should be separated by a comma. The name in the `A` position can also have spaces, but I ended up not using them because it made analysis I wanted to do later using `igraph` easier. Here's an example from a list for Neal Stephenson's _Cryptonomicon_:<br/>
>Amy: Amy Shaftoe, Amy<br/>
>Andrew: Andrew, Andrew Loeb, Andrews, Andy, Andy Loeb, Loeb<br/>

Luczak-Roesch et al have made the character lists they used with the tool (for 19 nineteenth-century British novels) available with the software on Github; additional character lists are being collected here: [https://osf.io/ewf4j/](https://osf.io/ewf4j/).

## The steps
Standard caveat: I'm still learning how to do things with Python/programming in general, so this is all extremely messy. I'm positive there are better and more efficient ways to do everything I describe. However, I think there's some value in showing my mess to those who are trying to learn themselves. As someone who often tries to teach myself technical stuff by reading online tutorials and blog posts (not always the best strategy), I appreciate being talked through a mess. But if you know what you're doing, you most likely won't find the following useful.

### 1. Clean up plain text file
To create one of these lists from scratch, I started with a plain-text version of the text I wanted to model. For contemporary texts, there are various ways to get this, but this process constitutes a post of its own. I'll just say here that if you have an ebook version, for example, you can use free programs like Calibre to wrangle it into plain text format. I then cleaned up the text a bit in a text editor to help with tagging by removing common punctuation marks (I use the old-school TextWrangler for this kind of stuff because I'm proficient with its regex find/replace functions):
- quotation marks (may need to straighten first)
- apostrophes
- single quotes
- slashes
- hyphens and dashes

### 2. Download Stanford NER package
Here's where I followed some of the instructions from [this tutorial by Erick Peirson](https://erickpeirson.github.io/python/2015/05/01/named-entity-recognition.html). I downloaded [Stanford's NER package](https://nlp.stanford.edu/software/CRF-NER.shtml#Download) and unpacked it in my home directory.

### 3. Tag your text
I used my terminal to `cd` into this directory, where I had also placed the plain-text file I had just cleaned up. I ran the following shell command (just type this into your terminal):
```./ner.sh ./your-plain-text-file.txt > ./your-plain-text-file_tagged.txt```
The first part of this command tells the NER package to run, and to use the plain-text file you moved into that folder. The `>` operator directs the output of this operation (the tagging) to a new file, which you call whatever you like. See Peirson's tutorial for more details on this.<br/>
**Note:** Ideally, we would want to do the tagging using Python too, perhaps with the NER Python package. However, I couldn't figure out how to get that package to work correctly.

### 4. Clean up your tagged text
Now you have the tagged text, but it's not in a useful form for our purposes just yet. Here's an example of what it looks like:
>Bobby/PERSON Shaftoe/PERSON ,/O and/O the/O other/O halfdozen/O Marines/O on/O his/O truck/O ,/O are/O staring/O down/O the/O length/O of/O Kiukiang/LOCATION Road/LOCATION ,/O onto/O which/O theyve/O just/O made/O this/O careening/O highspeed/O turn/O ./O

The NER tagger will tag 4 classes by default: PERSON, LOCATION, ORGANIZATION, and misc (/O). But we just want a list of all of the entities tagged as PERSON, with first and last names combined. I put together a very hacky and bad Jupyter notebook that will do just this. You can find it on [Github](https://github.com/lcthomas/network-analysis-novels). I'm still in the very early stages of learning Python, so I want to stress again that it's bad and very stupid -- but it worked for my purposes. The notebook contains more instructions about how to use it.<br/>
**Note:** We could use the NER shell package to print out each entity and its class to a 2-column csv (more details on that here: [https://nlp.stanford.edu/software/CRF-NER.shtml#Starting](https://nlp.stanford.edu/software/CRF-NER.shtml#Starting)), but we would still need to do some post-processing to get a list of all of the PERSON entities.

### 5. Clean up your entity list
The output of the Jupyter notebook is a json file with all of the PERSON entities listed on their own lines. As the notebook states, however, this list will still contain duplicate names. I turned once again to TextWrangler to help me with this because it's just faster for me right now, and I used it to delete duplicates, remove json quotes and brackets, alphabetize the list, and remove the tabs at the beginning of each line. Then I saved this file as a plain-text file.

### 6. Now the real work begins
Yay, a character list! you might think. Unfortunately, no. What you have now is a list of everything in the text that the Stanford NER tagger has identified as a PERSON. This is not necessarily the same thing as a list of the characters in your novel. They might be the names of historical or current public figures, for example, or mistakes that the tagger has made. The list will also include character nicknames and alternate names that you will want to associate with one another for your character list. Unfortunately, I don't know of a better method for actually creating the character list once you get to this point than doing it by hand (But again, what do I know? If you know of one, please don't be shy). The PERSON entity list gives you a starting point, but that's it.

When I used this method to create some character lists, I found it worked best to alphabetize the entity list, because it allowed me to quickly see what names might be associated with one another. Then, I exhaustively checked each name on the entity list with the full text of the novel by searching for all of the instances of that name in the text. If it was clear the name was a character name, it stayed on the list. If the name was a nickname, I associated it with a character name. If the name was a reference to a historical or public figure (who was not a character in the text), or if the tagger had made a mistake, I deleted it. How long this takes depends on the number of characters in the novel and how well you know the novel -- but it too, me, at minimum, several hours per text.

Creating such lists is time-intensive, but it also leads to some interesting questions about who exactly counts as a "character" in a text. Do people who show up for only one scene count as characters? What about people who exist only in the memories of other characters, or in flashback scenes? What about unnamed people to whom the text devotes no more than, say, one scene? I tried to focus on these more interesting questions to help get me through the slog.

And there you have it: a messy and time-consuming way to create lists of characters in novels!
