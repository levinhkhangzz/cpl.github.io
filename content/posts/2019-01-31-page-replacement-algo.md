---
title: "Page Replacement Design"
description: "Common algorithms used for page replacement in a MMU"
keywords: [technical, tutorial, low-level, algorithm, design]
date: 2019-01-31
---

## NRU

> Very crude approximation of LRU

This algorithm takes advantage of the `R` (referenced/read/written) bit and `M` (modified) bit. On every update cycle (could be every clock interrupt) all pages get their `R` bit set to 0, to help detect unreferenced pages.

On a page fault, pages are inspected and separated in 4 classes:

Class | R | M
------|---|---
0     | 0 | 0
1     | 0 | 1
2     | 1 | 0
3     | 1 | 1

The page swap algorithm will remove one random page from the lowest available class.

## FIFO

> Might throw out important pages

As the name suggest, **F**irst-**I**n-**F**irst-**O**ut keeps a list in memory of all page frames. When a fault occurs, the tail page is thrown out and the new page is inserted at the "top" of the list. While this might work it may also throw out some important pages, causing unnecessary page faults.

```text
[1] -> [4] -> [2] -> [3] -> [5]

-- page fault, 7 --

       [1] -> [4] -> [2] -> [3]
[7] -> [1] -> [4] -> [2] -> [3]

-- page fault, 5 --

       [7] -> [1] -> [4] -> [2]
[5] -> [7] -> [1] -> [4] -> [2]
```

## Second Chance

> Big improvement over FIFO

Similar to *FIFO*, the pages are stored in a list. Instead of removing the oldest page every single time a page fault occurs, we first check the `R` bit of the page. If it's `0`, we throw out the page and insert the new one at the start. If the bit is `1` then we move the page to the start of the list (just as we would do with a new page, and set it's `R` bit to `0`), this way we give the page another chance. The search continues up the list, moving pages which have their `R` bit `1` to the start, until an unused page is found.

```text
[1] -> [4] -> [2] -> [3] -> [5]

-- page fault, 7 --
-- assume 5 has bit R, 1 --
-- assume 3 has bit R, 0 --

[5] -> [1] -> [4] -> [2] -> [3]
       [5] -> [1] -> [4] -> [2]
[7] -> [5] -> [1] -> [4] -> [2]
```

You might notice, the algorithm can get expensive in terms of resources as it has to move items around in the list.

## Clock

> Realistic

Instead of having to move pages in a list, this algorithm uses a circular list and a pointer to the oldest page.

```text

-> [1] -> [4] -> [2] -> [3] -> *points back to [1]
    ,
   /|\
    |
  oldest
   page
 pointer
```

If a page fault occurs, a similar approach to second chance is taken, check bit `R`, if it's `0` evict the page and insert the new one in it's place. If the bit is `1`, set it to `0` and move the pointer to the next entry.

## LRU

> Excellent, but difficult to implement exactly

LRU is a cyclic cache that tracks **when** was each page used last and updates the order of the list to keep track of the **L**east-**R**ecently-**U**sed page.

This is very expensive to do in both software and hardware, there are special hardware implementations which work.

There are a couple of software solutions which try to approximate/recreate the behaviour of the LRU algorithm.

## NFU

> Fairly crude approximation to LRU

The **N**ot-**F**requently-**U**sed algorithm requires a counter for each page. The counter starts off at 0, and on every clock interrupt the counter is updated if the `R` bit is `1`. When a page fault occurs the page with the lowest counter is replaced with the new one.

There are multiple ways the counter can be incremented. The simplest one is to keep adding 1 to it every time `R` is `1` (or simply `counter += R`).

The issue with this is, some pages may be used heavily in one cycle of the program, thus increasing their counter to high numbers. Then these pages may go unused for the rest of the program, but because of their high counters, they may never be overtaken by new pages which will be swapped in and out.

## Ageing

> Efficient algorithm that approximates LRU well

A simple modification to the counter updating policy used by *NFU* makes it much better. Instead of simply adding `R` on every update, we first shift the counter 1 bit to the right and then add `R` to the leftmost bit. This way the counter binary representation will contain the history/timeline when the page was used.

```text
Imagine the R bit sequence: 0, 0, 1, 1, 1, 0, 1, 1, 0, 1, 0, 0

time    Page Counter
 |        00000000
 |        00000000
 |        10000000
 |        11000000
 |        11100000
 |        01110000
 |        10111000
 |        11011100
 |        01101110
 |        10110111
 |        01011011
 |        00101101
\|/
 `
```

## Working Set

> Somewhat expensive to implement

The *working set* algorithm takes advantage of the fact that normal programs will have a certain order/logic/pattern in the way they call pages. Page references tend to cluster around a small number of pages, this algorithm is used to order together pages `k` in a set that may be used at some point `t` in time, giving us the group `w(k, t)`.

In the context of page replacement, the algorithm will use an approximate time `t` of last use and the `R` bit. The system will have a `T`, current time. When an update occurs, pages are scanned and:

* If `R` is `1`, update the timestamp to the current time
* If `R` is `0`
  * If the age (`T-t`) is greater than `t`, remove it
  * If the age (`T-t`) is less than or equal to `t`, save the smallest one

If all pages have `R` equal to `1` then a page at random will be removed.

## WSClock

> Good efficient algorithm

This is a combination of the *Clock* and *Working Set* algorithms.

A circular list structure will store the `t` and `R` for each page. When the page pointed at has the `R` bit `1`, it has `R` set to `0` and the pointer move to the next entry. If the current page has `R` equal to `0` then `t` is checked and if `T-t` is greater than it, the page gets replaced. Otherwise the algorithm just moves to the next page to avoid having to update the page.
