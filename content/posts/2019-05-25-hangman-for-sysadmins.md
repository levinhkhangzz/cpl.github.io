---
title: "Hangman, but for sysadmins"
description: "The best way to lose your friends by playing hangman"
keywords: [fun, bash, hangman, game, sysadmin]
date: 2019-05-25
---

If you know what *hangman* is, skip to the second section.

## What is hangman?

---

We all know (or I expect people to know) what is and how to play *hangman*. It’s a two player (don’t know how you’d play this with more people, but I am sure somebody found a way) game in which one player thinks of a word then draws on a board or piece of paper a series of lines for each letter. After that it’s common to give the first letter and all other instances of it in the words. Let’s take an example. Player one thinks of the words `barber` and must now draw `_ _ _ _ _ _` and give away the first letter, `B` and the next occurrence. `B _ _ B _ _`.

![Hangman 1](/assets/images/hangman.svg)

Now the goal of the second player is to guess the word. This can be either by guessing the entire word if you’re confident enough, or by asking if a certain letter is part of the word. A good guess when playing in english is `E`. In this case it would work and player one must complete all occurrences of `E` (`B _ _ B E _`). If the second player guesses wrong (the letter is not part of the word or guesses the wrong word), then player one *removes a life*. This is done by drawing parts of a stickman on the *hanging space*. It’s up to the players to decide how many lives there are and how slow/fast the stickman is being drawn.

Imagine player two guesses wrong by saying `O`. Player one writes `O` to the side, to mark it as a used letter and then starts the stickman drawing.

![Hangman 2](/assets/images/hangman2.svg)

If player two manages to guess all the letters or the word, then player two wins. If player two runs out of lives guessing, then player one wins when the drawing of the stick man is complete.

![Hangman 3](/assets/images/hangman3.svg)


## How to pick a good, valid word

---

Now, if you’re a software engineer/developer/architect/<insert geeky term>, you might want to use this:

```shell
$ shuf -n1 /usr/share/dict/words
nonsyllogistic
```

```shell
$ shuf -n5 /usr/share/dict/words
netlike
ablepharous
nontrade
tulipiferous
hippodamous
```

The `shuf` command randomises whatever input you give it. In this case it’s the entire local dictionary that’s usually shipped with UNIX systems. If you don’t find `/usr/share/dict/words` you might look into other places. If you don’t have `shuf` as a command, install `coreutils`.

But wait, there’s more. We can take this to the next level, how about a random word of 10 chars? Easy.

```shell
$ egrep -x '.{10}' /usr/share/dict/words
abalienate
abaptiston
abasedness
abbeystede
...
```
 
Or you can select all words in a certain range.

```shell
$ egrep -x '.{2,4}' /usr/share/dict/words
aa
aal
aam
Aani
Aaru
Ab
aba
...
```

In other words, if you want to really defeat your friends (and lose them in the process) use the following command:

```shell
$ egrep -x '.{20}' /usr/share/dict/words | shuf -n1
thoracogastroschisis
```

## How to guess any word

---

The simplest solution would be to use `egrep` with all the known letters and `.` for the unknown. Let’s take the `b _ _ b e _ ` case.

```shell
$ egrep -x 'b..be.' /usr/share/dict/words
barbed
barbel
barber
barbet
bawbee
benben
bibber
bobbed
bobber
bombed
bomber
bribee
briber
bulbed
bumbee
```

You could use your  instinct and pick what you think the other person is thinking of. But we could also try to employ some statistics, and get the most common letter for a certain position.

A more *fancy* solution is to take the entire word dictionary, create a hashmap using the first letter of the word as the key and then store the rest of the word as a [trie](https://en.wikipedia.org/wiki/Trie). This could help you solve a hangman word in no time, with the most probable letters. The other solution, which would go outside the purpose of this post, would be **RegEx**. Expect a post about **RegEx** in the near future.
