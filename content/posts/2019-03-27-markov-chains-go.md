---
title: "Markov Chains in Go"
description: "A fun little project that generates new text based on markov chains"
keywords: [markov chain, golang, project, fun]
date: 2019-03-27
---


## Introduction
---

While browsing the golang documentation and packages I stumbled upon something entirely new, a “codewalk” for a markov chain algorithm that generates text based on a “trained” markov chain from user input [[^1]]. This reminded me of the time I studied Markov Chains for a University course but never got to see a software implementation of what it might look. To my surprise it was pretty simple and straightforward.

Brushing up my knowledge of markov chains and checking out a visual explanations [[^3]] and another more complex implementation of the same algorithm [[^2]], I decided to try and make my own and then connect it to my API.

Now if you’re not interested in how I did it and just **want to dump some obscene amounts of text** for my poor little free Heroku hosted API to handle, then skip to **[here](#tryit)**. If you wish to see the **GitHub repo** go [here](https://git.cpl.li/markov).

## What are Markov Chains
---

I wouldn’t call myself a mathematician so if you want a more mathematical explanation check out Wikipedia, or any the referenced posts.

I will do my best and explain it with an example. If we provide the following block of text to a markov chain:

```text
I am an engineer
I am not a mathematician
```

We can split this text into sequences of different sizes

```go
// size 1
["I" "am" "an" "engineer" "I" "am" "not" "a" "mathematician" ...]

// size 2
["I am" "am an" "an engineer" "engineer I" "I am" "am not" ...]

// size 3
"I am an" "am an engineer" "an engineer I" "engineer I am" ...]
```

Now for each sequence we must also store the following token, thus creating a pair. Not that in our example the size 2 sequence `"I am"` can be followed by multiple words: `an` `not` so our transition mapping has to keep both relations and their occurrence count (as a pair may occur multiple times this increasing its frequency)

```go
Pair {
    Sequence: "I am"
    Next: {
        "an":  1
        "not": 1
    }
}
```

After the transition map is created on a sequence of input data, it’s actually up to you how you decide to generate the next token, the way I did it is by giving the Builder a *seed* which is actually just a sequence of word, if non is provided the Builder will pick at random a starting sequence and go from there.

## Implementation
---

It was easy to create a simple markov chain using the following structs:

```go
// represents a grouping of individual words
// eg: []string{“I”, “am”, “Alex”}, this can be extracted
// from an original string of any form or shape:
// “I am Alex”, “I   am  Alex”, “I:am:Alex”
// and it’s all up to the caller to split their strings into sequences
type Sequence []string
```

```go
// a pairs represents a possible transition between a sequence of n words
// and the next (single) word
// the Current sequence must be of an equal lenght to the chain pair size
// meaning you can’t have some transitions for 2-grouped words and 1-grouped words
type Pair struct {
	Current Sequence
	Next    string
}
```

```go
// by having a
type transitionMap map[string]int
// and then nested inside
frequencyMatrix map[string]transitionMap
// we generate our mapping of all encountered
// sequences to their respective next word
// and the number of times this occurs
```

And then by creating the `Builder` on top of a chain, we can continually generate new words, append them to a sequence and then use that sequence to further poll the `Chain` for new words.

## Improvements
---
There are plenty of improvements that can be made but this was intended as a fun little project to kill off some of the time I don’t have. Maybe another day I’ll pick this up and make it generate even more accurate and funny text, until then please feel free to enjoy an “interactive” version below.

<a name="tryit"></a>
## Try it
---
You can submit any sort of text here (it’s not stored anywhere) with a character
limit of 50.000 (and yes it’s both frontend and backend validated). You will
then be redirected to my api, where you’ll get a generated text of 200 words max.


<!-- MARKOV CHAIN INPUT FORM -->
<form action="https://api.cpl.li/markov" method="post" id="markov-form">
    <textarea
        name="markov-input"
        form="markov-form"
        rows="20" maxlength="50000"
        placeholder="Write your training text here">
    </textarea>
    <input type="submit" value="Submit">
</form>


## References
---

[^1]: [Codewalk: Markov chain algorithm](https://golang.org/doc/codewalk/markov/)
[^2]: [Building markov chains in golang](https://mb-14.github.io/tech/2018/10/24/gomarkov.html)
[^3]: [Markov Chains explained visually](http://setosa.io/ev/markov-chains/)
