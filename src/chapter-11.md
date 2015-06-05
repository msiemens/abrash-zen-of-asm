# Chapter 11: String Instruction Applications

Now that we've got a solid understanding of what the string instructions do, let's look at a few applications to get a sense of what they're particularly good for. The applications we'll look at include copying arrays, searching strings for characters, looking up entries in tables, comparing strings, and animation.

There's a lot of meat in this chapter, and a lot of useful code. The code isn't fully fleshed out, since I'm trying to illustrate basic principles rather than providing you with a library from A to Z, but that's actually all to the good. You can build on this code to meet your specific needs or write your own code from scratch once you understand the ins and outs of the string instructions. In either case, you'll be better off with code customized to suit your purposes than you would be using any one-size-fits-all code I could provide.

I'll frequently contrast the string instruction-based implementations with versions built around non-string instructions. This should give you a greater appreciation for the string instructions, and may shed new light on the non-string instructions as well. I'll tell you ahead of time how the comparisons will turn out: in almost every case the string instructions will prove to be vastly superior. The lesson we learned in the last chapter holds true: *use the string instructions to the hilt!* There's nothing like them under the (8088) sun.

Contrasting string and non-string implementations also reinforces an important point. There are many, many ways to accomplish any given task on the 8088. It's knowing which approach to choose that separates the journeyman programmer from the guru.

## String Handling With `lods` and `stos`

`lods`{.nasm} is an odd bird among string instructions, being the only string instruction that doesn't benefit in the least from `rep`{.nasm}. While `rep`{.nasm} does work with `lods`{.nasm}, in that it causes `lods`{.nasm} to repeat multiple times, the combination of the two is nonetheless totally impractical: what good could it possibly do to load AL twice (to say nothing of 64 K times)? Without `rep`{.nasm}, `lods`{.nasm} is still better than `mov`{.nasm}, but not *that* much better; `lods`{.nasm} certainly doesn't generate the quantum jump in performance that `rep stos`{.nasm} and `rep movs`{.nasm} do. So—when *does* `lods`{.nasm} really shine?

It turns out that `lods`{.nasm} is what might be called a "synergistic"instruction, at its best when used with `stos`{.nasm} (or sometimes `scas`{.nasm}, or even non-string instructions) in a loop. Together, `lods`{.nasm} and `stos`{.nasm} let you load an array or string element into AL, test and/or modify it, and then write the element back to either the original array or a new array, as shown in Figure 11.1.

![](../images/fig11.1aRT.png)

![](../images/fig11.1bRT.png)

You might think of the `lods`{.nasm}-process-`stos`{.nasm} combination as being a sort of "meta-`movs`{.nasm}," whereby you can whip up customized memory-to-memory moves as needed. Of course, `lods`{.nasm}/`stos`{.nasm} is slower than `movs`{.nasm} (especially `rep movs`{.nasm}), but by the same token `lods`{.nasm}/`stos`{.nasm} is far more flexible. Besides, `lods`{.nasm}/`stos`{.nasm} isn't *that* slow—*all* of the 8088's memory-accessing instructions suffer by comparison with `movs`{.nasm}. Placed inside a loop, the `lods`{.nasm}/`stos`{.nasm} combination makes for fairly speedy array and string processing.

For example, [Listing 11-1](#listing-11-1) copies a string to a new location, converting all characters to uppercase in the process, by using a loop containing `lods`{.nasm} and `stos`{.nasm}. [Listing 11-1](#listing-11-1) takes just 773 us to copy and convert. By contrast, [Listing 11-2](#listing-11-2), which uses non-string instructions to perform the same task, takes 921 us to perform the copy and conversion.

By the way, [Listing 11-1](#listing-11-1) could just as easily have converted `SourceString`{.nasm} to uppercase in place, rather than copying the converted text to `DestString`{.nasm}. This would be accomplished simply by loading both DS:SI and ES:DI to point to `SourceString`{.nasm}, as shown in [Listing 11-3](#listing-11-3), which changes nothing else from [Listing 11-1](#listing-11-1).

Why is this interesting? It's interesting because two pointers—DS:SI and ES:DI—are used to point to a single array. It's often faster to maintain two pointers and use `lods`{.nasm} and `stos`{.nasm} than it is to use a single pointer with non-string instructions, as in [Listing 11-4](#listing-11-4). [Listing 11-3](#listing-11-3) runs in 771 us, about the same as [Listing 11-1](#listing-11-1) (after all, they're virtually identical). However, [Listing 11-4](#listing-11-4) takes 838 us, even though it uses only one pointer to point to the array being converted to uppercase.

The `lods`{.nasm}/`stos`{.nasm} pair lies somewhere between the repeated string instructions and the non-string instructions in terms of performance and flexibility. `lods`{.nasm}/`stos`{.nasm} isn't as fast as any of the repeated string instructions, both because two instructions are involved and because it can't be used with a `rep`{.nasm} prefix but must instead be placed in a loop. However, `lods`{.nasm}/`stos`{.nasm} is a good deal more flexible than any repeated string instruction, since once a memory operand is loaded into AL or AX it can be tested and manipulated easily (and often quickly as well, thanks to the accumulator-specific instructions).

On the other hand, the `lods`{.nasm}/`stos`{.nasm} pair is certainly faster than non-string instructions, as [Listings 11-1](#listing-11-1) through [11-4](#listing-11-4) illustrate. However, `lods`{.nasm}/`stos`{.nasm} is not as flexible as the non-string instructions, since DS:SI and ES:DI must be used as pointer registers and only the accumulator can be loaded from and stored to memory.

On balance, the `lods`{.nasm}/`stos`{.nasm} pair overcomes some but not all of the limitations of repeated string instructions, and does so at a substantial performance cost *vis-a-vis* the repeated string instructions. One thing that `lods`{.nasm}/`stos`{.nasm} doesn't do particularly well is modify memory directly. For example, suppose that we want to set the high bit of every byte in a 1000-byte array. We could of course do this with `lodsb`{.nasm} and `stosb`{.nasm}, setting the high bit of each word while it's loaded into AL. [Listing 11-5](#listing-11-5), which does exactly that, takes 10.07 us per word.

However, we could also use a plain old `or`{.nasm} instruction working directly with a memory operand to do the same thing, as shown in [Listing 11-6](#listing-11-6). [Listing 11-6](#listing-11-6) is just as fast as [Listing 11-5](#listing-11-5) at 10.06 us per word, and it's also considerably shorter at 13 rather than 21 bytes, with 1 less byte inside the loop. `lods`{.nasm}/`stos`{.nasm} isn't *disastrously* worse in this case, but it certainly isn't the preferred solution—and there are plenty of other situations in which `lods`{.nasm}/`stos`{.nasm} is less than ideal.

For instance, when registers are tight, the extra pointer register `lods`{.nasm}/`stos`{.nasm} takes can be sorely missed. If the accumulator is reserved for some specific purpose and can't be modified, `lods`{.nasm}/`stos`{.nasm} can't very well be used. If a pointer to far data is needed by other instructions in the same routine, the limitation of `stos`{.nasm} to operating in the ES segment would become a burden. In other words, while the `lods`{.nasm}/`stos`{.nasm} pair is more flexible than the repeated string instructions, its limitations are significant nonetheless.

The point is not simply that the `lods`{.nasm}/`stos`{.nasm} pair is not as flexible as the non-string instructions. The real point is that you shouldn't assume you've come up with the best solution just because you've used string instructions. Yes, I know that I've been touting string instructions as the greatest thing since sliced bread, and by and large that's true. However, because the string instructions have a sharply limited repertoire and often require a good deal of preliminary set-up, you must consider your alternatives before concluding that a string instruction-based implementation is best.

## Block Handling With `movs`

Simply put, `movs`{.nasm} is the king of the block copy. There's no other 8088 instruction that can hold a candle to `movs`{.nasm}when it comes to copying blocks of data from one area of memory to another. It does take several instructions to set up for `movs`{.nasm}, so if you're only moving a few bytes and DS:SI and ES:DI don't happen to be pointing to your source and destination, you might want to use a regular `mov`{.nasm}. Whenever you want to move more than a few bytes, though, `movs`{.nasm}—or better yet `rep movs`{.nasm}—is the ticket.

Let's look at the archetypal application for `movs`{.nasm}, a subroutine which copies a block of memory from one memory area to another. What's special about the subroutine we'll look at is that it handles copying a block when the destination of the copy overlaps the source. This is a bit tricky because the direction in which the copy must proceed—from the start of the block toward the end, or vice-versa—depends on the direction of overlap.

If the destination block overlaps the source block and starts at a lower memory address than the source block, then the copy can proceed in the normal direction, from lower to higher addresses, as shown in Figure 11.2.

![](../images/fig11.2RT.png)

If the destination block overlaps the source block and starts at a *higher* address, however, the block must be copied starting at its highest address and proceeding toward the low end, as shown in Figure 11.3.

![](../images/fig11.3RT.png)

Otherwise, the first data copied to the destination block would wipe out source data that had yet to be copied, resulting in a corrupted copy, as shown in Figure 11.4.

![](../images/fig11.4aRT.png)

![](../images/fig11.4bRT.png)

Finally, if the blocks don't overlap, the copy can proceed in either direction, since the two blocks can't conflict.

The block-copy subroutine `BlockCopyWithOverlap`{.nasm} shown in [Listing 11-7](#listing-11-7) handles potential overlap problems exactly as described above. In cases where the destination block starts at a higher address than the source block, `BlockCopyWithOverlap`{.nasm} performs an `std`{.nasm} and uses `movs`{.nasm} to copy the source block starting at the high end and proceeding to the low end. Otherwise, the source block is copied from the low end to the high end with `cld`{.nasm}/`movs`{.nasm}. `BlockCopyWithOverlap`{.nasm} is both remarkably compact and very fast, clocking in at 5.57 ms for the cases tested in [Listing 11-7](#listing-11-7). The subroutine could actually be more compact still, but I've chosen to improve performance at the expense of a few bytes by copying as much of the block as possible a word rather than a byte at a time.

There are two points of particular interest in [Listing 11-7](#listing-11-7). First, `BlockCopyWithOverlap`{.nasm} only handles blocks that reside in the same segment, and then only if neither block wraps around the end of the segment. While it would certainly be possible to write a version of the subroutine that properly handled both potentially overlapping copies between different segments and segment wrapping, neither of those features is usually necessary, and the additional code would reduce overall performance. If you need such a routine, write it, but as a general practice don't write extra, slower code just to handle cases that you can readily avoid.

Second, `BlockCopyWithOverlap`{.nasm} nicely illustrates a nasty aspect of the use of word-sized string instructions when the Direction flag is set to 1. The basic problem is this: if you point to the last byte of a block of memory and perform a word-sized operation, the byte *after* the end of the memory block will be accessed along with the last byte of the block, rather than the last *two* bytes of the block, as shown in Figure 11.5.

![](../images/fig11.5RT.png)

This problem of accessing the byte after the end of a memory block can occur with all word-sized instructions, not just string instructions. However, it's especially liable to happen with a word-sized string instruction that's moving its pointer or pointers backward (with the Direction flag equal to 1) because the temptation is to point to the end of the block, set the Direction flag, and let the string instruction do its stuff in repeated word-sized chunks for maximum performance. To avoid this problem, you must always be sure to point to the last *word* rather than byte when you point to the last element in a memory block and then access memory with a word-sized instruction.

Matters get even more dicey when byte-and word-sized string instructions are mixed when the Direction flag is set to 1. This is done in [Listing 11-7](#listing-11-7) in order to use `rep movsw`{.nasm} to move the largest possible portion of odd-length memory blocks. The problem here is that when a string instruction moves its pointer or pointers from high addresses to low, the address of the next byte that we want to access (with `lodsb`{.nasm}, for example) and the address of the next word that we want to access (with `lodsw`{.nasm}, for example) differ, as shown in Figure 11.6.

![](../images/fig11.6RT.png)

For a byte-sized string instruction such as `lodsb`{.nasm}, we *do* want to point to the end of the array. After that `lodsb`{.nasm} has executed with the Direction flag equal to 1, though, where do the pointers point? To the address 1 byte—not 1 word—lower in memory. Then what happens when `lodsw`{.nasm} is executed as the next instruction, with the intent of accessing the word just above the last byte of the array? Why, the last byte of the array is incorrectly accessed again, as shown in Figure 11.7.

![](../images/fig11.7aRT.png)

![](../images/fig11.7bRT.png)

The solution, as shown in [Listing 11-7](#listing-11-7), is fairly simple. We must perform the initial `movsb`{.nasm} and then adjust the pointers to point 1 byte lower in memory—to the start of the next *word*. Only then can we go ahead with a `movsw`{.nasm}, as shown in Figure 11.8.

![](../images/fig11.8aRT.png)

![](../images/fig11.8bRT.png)

Mind you, all this *only* applies when the Direction Flag is 1. When the Direction flag is 0, `movsb`{.nasm} and `movsw`{.nasm} can be mixed freely, since the address of the next byte is the same as the address of the next word when we're counting from low addresses to high, as shown in Figure 11.9.

![](../images/fig11.9RT.png)

[Listing 11-7](#listing-11-7) reflects this, since the pointer adjustments are only made when the Direction flag is 1.

[Listing 11-8](#listing-11-8) contains a version of `BlockCopyWithOverlap`{.nasm} that does exactly what the version in [Listing 11-7](#listing-11-7) does, but does so without string instructions. While [Listing 11-8](#listing-11-8) doesn't *look* all that much different from [Listing 11-7](#listing-11-7), it takes a full 15.16 ms to run -quite change from the time of 5.57 ms we measured for [Listing 11-7](#listing-11-7). Think about it: [Listing 11-7](#listing-11-7) is nearly *three times* as fast as [Listing 11-8](#listing-11-8), thanks to `movs`{.nasm}—and it's shorter too.

Enough said.

## Searching With `scas`

`scas`{.nasm} is often (but not always, as we shall see) the preferred way to search for either a given value or the absence of a given value in any array. When `scas`{.nasm} is well-matched to the task at hand, it is the best choice by a wide margin. For example, suppose that we want to count the number of times the letter 'A' appears in a text array. [Listing 11-9](#listing-11-9), which uses non-string instructions, counts the number of occurrences of 'A' in the sample array in 475 us. [Listing 11-10](#listing-11-10), which does exactly the same thing with `repnz scasb`{.nasm}, finishes in just 203 us. That, my friends, is an improvement of 134%. What's more, [Listing 11-10](#listing-11-10) is shorter than [Listing 11-9](#listing-11-9).

Incidentally, [Listing 11-10](#listing-11-10) illustrates the subtlety of the pitfalls associated with forgetting that `scas`{.nasm} repeated zero times (with CX equal to zero) doesn't alter the flags. If the `jcxz`{.nasm} instruction in [Listing 11-10](#listing-11-10) were to be removed, the code would still work perfectly—except when the array being scanned was exactly 64 K bytes long and *every* byte in the array matched the byte being searched for. In that one case, CX would be zero when `repnz scasb`{.nasm} was restarted after the last match, causing `repnz scasb`{.nasm} to drop through without altering the flags. The Zero flag would be 0 as a result of DX previously incrementing from 0FFFFh to 0, and so the `jnz`{.nasm} branch would not be taken. Instead, DX would be incremented again, causing a non-existent match to be counted. The result would be that 1 rather than 64 K matches would be returned as the match count, an error of considerable magnitude.

If you could be sure that no array longer than 64 K-1 bytes would ever be passed to `ByteCount`{.nasm}, you *could* eliminate the `jcxz`{.nasm} and speed the code considerably. Trimming the fat from your code until it's matched exactly to an application's needs is one key to performance.

### `scas` and Zero-Terminated Strings

Clearly, then, when you want to find a given byte or word value in a buffer, table, or array of a known fixed length, it's often best to load up the registers and let a repeated `scas`{.nasm} do its stuff. However, the same is not always true of searching tasks that require multiple comparisons for each byte or word, such as a loop that ends when either the letter 'A' *or* a zero byte is found. Alas, `scas`{.nasm} can perform just one comparison per memory location, and `repz`{.nasm} or `repnz`{.nasm} can only terminate on the basis of the Zero flag setting after that one comparison. This is unfortunate because multiple comparisons are exactly what we need to handle C-style strings, which are of no fixed length and are terminated with zeros. `rep scas`{.nasm} can still be used in such situations, but its sheer power is diluted by the workarounds needed to allow it to function more flexibly than it is normally capable of doing. The choice between repeated `scas`{.nasm} instructions and other approaches then must be made on a case-by-by case basis, according to the balance between the extra overhead needed to coax `scas`{.nasm} into doing what is needed and the inherent speed of the instruction.

For example, suppose we need a subroutine that returns either the offset in a string of the first instance of a selected byte value or the value zero if a zero byte (marking the end of the string) is encountered before the desired byte is found. There's no simple way to do this with `scasb`{.nasm}, for in this application we have to compare each memory location first to the desired byte value and then to zero. `scasb`{.nasm} can perform one comparison or the other, but not both.

Now, we *could* use `rep scasb`{.nasm} to find the zero byte at the end of the string, so we'd know how long the string was, and then use `rep scasb`{.nasm} again with CX set to the length of the string to search for the selected byte value. Unfortunately, that involves processing *every* byte in the string once before even beginning the search. On average, this double-search approach would read every element of the string being searched once and would then read one-half of the elements again, as shown in Figure 11.10. By contrast, an approach that reads each byte and immediately compares it to both the desired value *and* zero would read only one-half of the elements in the string, as shown in Figure 11.11. Powerful as repeated `scasb`{.nasm} is, could it

![](../images/fig11.10aRT.png)

![](../images/fig11.10bRT.png)

possibly run fast enough to allow the double-search approach to outperform an approach that accesses memory only one-third as many times?

The answer is yes... conditionally. The double-search approach actually *is* slightly faster than a `lodsb`{.nasm}-based single-search string-searching approach for the average case. The double-search approach performs relatively more poorly if matches tend to occur most frequently in the first half of the strings being searched, and relatively better if matches tend to occur in the second half of the strings. Also, the more flexible `lodsb`{.nasm}-based approach rapidly becomes the solution of choice as the termination condition becomes more complex, as when a case-insensitive search is desired. The same is true when modification as well as searching of the string is desired, as when the string is converted to uppercase.

[Listing 11-11](#listing-11-11) shows `lodsb`{.nasm}-based code that searches a zero-terminated string for the character 'z'. For the sample string, which has the first match right in the middle of the string, [Listing 11-11](#listing-11-11) takes 375 us to find the match. [Listing 11-12](#listing-11-12) shows `repnz scasb`{.nasm}-based code that uses the double-search approach. For the same sample string as [Listing 11-11](#listing-11-11), [Listing 11-12](#listing-11-12) takes just 340 us to find the match, despite having to perform about three times as many memory accesses as [Listing 11-11](#listing-11-11)—a tribute to the raw

![](../images/fig11.11RT.png)

power of repeated `scas`{.nasm}. Finally, [Listing 11-13](#listing-11-13), which performs the same search using non-string instructions, takes 419 us to find the match.

It is apparent from [Listings 11-11](#listing-11-11) and [11-12](#listing-11-12) that the performance margin between `scas`{.nasm}-based string searching and other approaches is considerably narrower than it was for array searching, due to the more complex termination conditions. Given a still more complex termination condition, `lods`{.nasm} would likely become the preferred solution due to its greater flexibility. In fact, if we're willing to expend a few bytes, the greater flexibility of `lods`{.nasm} can be translated into higher performance for [Listings 11-11](#listing-11-11), as follows.

[Listing 11-14](#listing-11-14) shows an interesting variation on [Listings 11-11](#listing-11-11). Here `lodsw`{.nasm} rather than `lodsb`{.nasm} is used, and AL and AH, respectively, are checked for the termination conditions. This technique uses a bit more code, but the replacement of two `lodsb`{.nasm} instructions with a single `lodsw`{.nasm} and the elimination of every other branch pays off handsomely, as [Listing 11-14](#listing-11-14) runs in just 325 us, 15% faster than [Listings 11-11](#listing-11-11) and 5% faster than [Listing 11-12](#listing-11-12). The key here is that `lods`{.nasm} allows us leeway in designing code to work around the slow memory access and slow branching of the 8088, while `scas`{.nasm} does not. In truth, the flexibility of `lods`{.nasm} can make for better performance still through in-line code... but that's a story for the next few chapters.

### More on `scas` and Zero-Terminated Strings

While repeated `scas`{.nasm} instructions aren't ideally suited to string searches involving complex conditions, they *do* work nicely with strings whenever brute force scanning comes into play. One such application is finding the offset of the *last* element of some sort in a string. For example, [Listing 11-15](#listing-11-15), which finds the last non-blank element of a string by using `lodsw`{.nasm} and remembering the offset of the most recent non-blank character encountered, takes 907 us to find the last non-blank character of the sample string, which has the last non-blank character in the middle of the string. [Listing 11-16](#listing-11-16), which does the same thing by using `repnz scasb`{.nasm} to find the end of the string and then `repz scasw`{.nasm} with the Direction flag set to 1 to find the first non-blank character scanning backward from the end of the string, runs in just 386 us.

That's an *amazing* improvement given our earlier results involving the relative speeds of `lodsw`{.nasm} and repeated `scas`{.nasm} in string applications. The reason that repeated `scas`{.nasm} outperforms `lodsw`{.nasm} by a tremendous amount in this case but underperformed it earlier is simple. The `lodsw`{.nasm}-based code always has to check every character in the string—right up to the terminating zero—when searching for the last non-blank character, as shown in Figure 11.12.

While the `scasb`{.nasm}-base code also has to access every character in the string, and then some, as shown in Figure 11.13, the worst case is that

![](../images/fig11.12RT.png)

[Listing 11-16](#listing-11-16) accesses string elements no more than twice as many times as [Listing 11-15](#listing-11-15). In our earlier example, the *best* case was a two-to-one ratio. The timing results for [Listings 11-15](#listing-11-15) and [11-16](#listing-11-16) show that the superior speed, lack of prefetching, and lack of branching associated with repeated `scas`{.nasm} far outweigh any performance loss resulting from a memory-access ratio of less than two-to-one.

By the way, [Listing 11-16](#listing-11-16) is an excellent example of the need to correct for pointer overrun when using the string instructions. No matter which direction we scan in, it's necessary to undo the last advance of DI performed by `scas`{.nasm} in order to point to the byte on which the comparison ended.

![](../images/fig11.13aRT.png)

![](../images/fig11.13bRT.png)

[Listing 11-16](#listing-11-16) also shows the use of `jcxz`{.nasm} to guard against the case where CX is zero. As you'll recall from the last chapter, repeated `scas`{.nasm} doesn't alter the flags when started with CX equal to zero. Consequently, we must test for the case of CX equal to zero before performing `repz scasw`{.nasm}, and we must treat that case if we had never found the terminating condition (a non-blank character). Otherwise, the leftover flags from an earlier instruction might give us a false result following a `repz scasw`{.nasm} which doesn't change the flags because it is repeated zero times. In [Listing 11-21](#listing-11-21) we'll see that we need to do the same with repeated `cmps`{.nasm} as well.

Bear in mind, however, that there are several ways to solve any problem in assembler. For example, in [Listing 11-16](#listing-11-16) I've chosen to use `jcxz`{.nasm} to guard against the case where CX is zero, thereby compensating for the fact that `scas`{.nasm} repeated zero times doesn't change the flags. Rather than thinking defensively, however, we could actually take advantage of that particular property of repeated `scas`{.nasm}. How? We could set the Zero flag to 1 (the "match" state) by placing `sub dx,dx`{.nasm} before `repz scasw`{.nasm}. Then if `repz scasw`{.nasm} is repeated zero times because CX is zero the following conditional jump will reach the proper conclusion, that the desired non-match (a non-blank character) wasn't found.

As it happens, `sub dx,dx`{.nasm} isn't particularly faster than `jcxz`{.nasm}, and so there's not much to choose from between the two solutions. With `sub dx,dx`{.nasm} the code is 3 cycles faster when CX isn't zero but is the same number of bytes in length, and is considerably slower when CX is zero. (There's really no reason to worry about performance here when CX is zero, however, since that's a rare case that's always handled relatively quickly. Rather, our focus should be on losing as little performance as possible to the test for CX being zero in the more common case—when CX *isn't* zero.) In another application, though, the desired Zero flag setting might fall out of the code preceding the repeated `cmps`{.nasm}, and no extra code at all would be required for the test for CX equal to zero. [Listing 11-24](#listing-11-24), which we'll come to shortly, is such a case.

What's interesting here is that it's instinctive to use `jcxz`{.nasm}, which is after all a specialized and fast instruction that is clearly present in the 8088's instruction set for just such a purpose as protecting against repeating a string comparison zero times. The idea of presetting a flag and letting the comparison drop through without changing the flag, on the other hand, is anything but intuitive—but is just about as effective as `jcxz`{.nasm}, more so under certain circumstances.

Don't let your mind be constrained by intentions of the designers of the 8088. Think in terms of what instructions *do* rather than what they were *intended* to do.

### Using Repeated `scasw` on Byte-Sized Data

[Listing 11-16](#listing-11-16) is also a fine example of how to use repeated `scasw`{.nasm} on byte-sized data. You'll recall that one of the rules of repeated string instruction usage is that word-sized string instructions should be used wherever possible, due to their faster overall speed. It turns out, however, that it's rather tricky to apply this rule to `scas`{.nasm}.

For starters, there's hardly ever any use for `repnz scasw`{.nasm} when searching for a specific byte value in memory. Why? Well, while we could load up both AH and AL with the byte we're looking for and then use `repnz scasw`{.nasm}, we'd only find cases where the desired byte occurs at least twice in a row, and then we'd only find such 2-byte cases that didn't span word boundaries. Unfortunately, there's no way to use `repnz scasw`{.nasm} to check whether either AH or AL—but not necessarily both—matched their respective bytes. With `repnz scasw`{.nasm}, if AX doesn't match all 16 bits of memory, the search will continue, and individual byte matches will be missed.

On the other hand, we *can* use `repz scasw`{.nasm} to search for the first *non-match*, as in [Listing 11-16](#listing-11-16). Why is it all right to search a word at a time for non-matches but not matches? Because if *either* byte of each word compared with `repz scasw`{.nasm} doesn't match the byte of interest (which is stored in both AH and AL), then `repz scasw`{.nasm} will stop, which is what we want. Of course, there's a bit of cleaning up to do in order to figure out which of the 2 bytes was the first non-match, as illustrated by [Listing 11-16](#listing-11-16). Yes, it is a bit complex and does add a few bytes, but it also speeds things up, and that's what we're after.

In short, `repz scasw`{.nasm} can be used to boost performance when scanning for non-matching byte-sized data. However, `repnz scasw`{.nasm} is generally useless when scanning for matching byte-sized data.

### `scas` and Look-Up Tables

One common application for table searching is to get an element number or an offset into a table that can be used to look up related data or a jump address in another table. We saw look-up tables in Chapter 7, and we'll see them again, for they're a potent performance tool.

`scas`{.nasm} is often excellent for look-up code, but the pointer and counter overrun characteristic of all string instructions make it a bit of a nuisance to calculate offsets and/or element numbers after repeated `scas`{.nasm} instructions. [Listing 11-17](#listing-11-17) shows a subroutine that calculates the offset of a match in a word-sized table in the process of jumping to the associated routine from a jump table. Notice that it's necessary to subtract the 2-byte overrun from the difference between the final value of DI and the start of the table. The calculation would be the same for a byte-sized table scanned with `scasb`{.nasm}, save that `scasb`{.nasm} has only a 1-byte overrun and so only 1 would be subtracted from the difference between DI and the start of the table.

Finding the element number is a slightly different matter. After a repeated `scas`{.nasm}, CX contains the number of elements that weren't scanned. Since CX counts down just once each time `scas`{.nasm} is repeated, there's no difference between `scasw`{.nasm} and `scasb`{.nasm} in this respect.

Well, if CX contains the number of elements that weren't scanned, then subtracting CX from the table length in elements must yield the number of elements that *were* scanned. Subtracting 1 from that value gives us the number of the last element scanned. (The first element is element number 0, the second element is element number 1, and so on.) [Listing 11-18](#listing-11-18) illustrates the calculation of the element number found in a look-up table as a step in the process of jumping to the associated routine from a jump table, much as in [Listing 11-17](#listing-11-17).

### Consider Your Options

Don't assume that `scas`{.nasm} is the ideal choice even for all memory-searching tasks in which the search length is known. Suppose that we simply want to know if a given character is any of, say, four characters: 'A', 'Z', '3', or '!'. We could do this with `repnz scasb`{.nasm}, as shown in [Listing 11-19](#listing-11-19). Alternatively, however, we could simply do it with four comparisons and conditional jumps, as shown in [Listing 11-20](#listing-11-20). Even with the prefetch queue cycle-eater doing its worst, each compare and conditional jump pair takes no more than 16 cycles when the jump isn't taken (the jump is taken at most once, on a match), which stacks up pretty well against the 15 cycle per comparison and 9 cycle set-up time of `repnz scasb`{.nasm}. What's more, the compare-and-jump approach requires no set-up instructions. In other words, the less sophisticated approach might well be better in this case.

The Zen timer bears this out. [Listing 11-19](#listing-11-19), which uses `repnz scasb`{.nasm}, takes 183 us to perform five checks, while [Listing 11-20](#listing-11-20), which uses the compare-and-jump approach, takes just 119 us to perform the same five checks. [Listing 11-20](#listing-11-20) is not only 54% faster than [Listing 11-19](#listing-11-19) but is also 1 byte shorter. (Don't forget to count the look-up table bytes in [Listing 11-19](#listing-11-19).)

Of course, the compare-and-jump approach is less flexible than the look-up approach, since the table length and contents can't be passed as parameters or changed as the program runs. The compare-and-jump approach also becomes unwieldy when more entries need to be checked, since 4 bytes are needed for each additional compare-and-jump entry where the `repnz scasb`{.nasm} approach needs just 1. The compare-and-jump approach finally falls apart when it's no longer possible to short-jump out of the comparison/jump code and so jumps around jumps must be used, as in:

```nasm
cmp   al,'Z'
jnz   $+5
jmp   CharacterFound
cmp   al,'3'
```

When jumps around jumps are used, the comparison time per character goes from 16 to 24 cycles, and `rep scasb`{.nasm} emerges as the clear favorite.

Nonetheless, [Listings 11-19](#listing-11-19) and [11-20](#listing-11-20) illustrate two important points. Point number 1: the repeated string instructions tend to have a greater advantage when they're repeated many times, allowing their speed and compact size to offset the overhead in set-up time and code they require. Point number 2: specialized as the string instructions are, there are ways to program the 8088 that are more specialized still. In certain cases, those specialized approaches can even outperform the string instructions. Sure, the specialized approaches, such as the compare-and-jump approach we just saw, are limited and inflexible—but when you don't need the flexibility, why pay for it in lost performance?

## Comparing Memory to Memory With `cmps`

When `cmps`{.nasm} does exactly what you need done it can't be beat, although to an even greater extent than with `scas`{.nasm} the cases in which that is true are relatively few. `cmps`{.nasm} is used for applications in which byte-for-byte or word-for-word comparisons between two memory blocks of a known length are performed, most notably array comparisons and substring searching. Like `scas`{.nasm}, `cmps`{.nasm} is not flexible enough to work at full power on other comparison tasks, such as case-insensitive substring searching or the comparison of zero-terminated strings, although with a bit of thought `cmps`{.nasm} can be made to serve adequately in some such applications.

`cmps`{.nasm} does just one thing, but it does far better than any other 8088 instruction or combination of instructions. The one transcendent ability of `cmps`{.nasm} is the direct comparison of two fixed-length blocks of memory. The obvious use of `cmps`{.nasm} is in determining whether two memory arrays or blocks of memory are the same, and if not, where they differ. [Listing 11-21](#listing-11-21), which runs in 685 us, illustrates `repz cmpsw`{.nasm} in action. [Listing 11-22](#listing-11-22), which performs exactly the same task as [Listing 11-21](#listing-11-21) but uses `lodsw`{.nasm} and `scasw`{.nasm} instead of `cmpsw`{.nasm}, runs in 1298 us. Finally, [Listing 11-23](#listing-11-23), which uses non-string instructions, takes a leisurely 1798 us to complete the task. As you can see, `cmps`{.nasm} blows away not only non-string instructions but also other string instructions under the right circumstances. (As I've said before, there are many, many different sequences of assembler code that will work for any given task. It's the choice of implementation that makes the difference between adequate code and great code.)

By the way, in [Listings 11-21](#listing-11-21) though [11-23](#listing-11-23) I've used `jcxz`{.nasm} to make sure the correct result is returned if zero-length arrays are compared. If you use this routine in your code and you can be sure that zero-length arrays will never be passed as parameters, however, you can save a few bytes and cycles by eliminating the `jcxz`{.nasm} check. After all, what sense does it make to compare zero-length arrays... and what sense does it make to waste precious bytes and cycles guarding against a contingency that can never arise?

Make the comparison a bit more complex, however, and `cmps`{.nasm} comes back to the pack. Consider the comparison of two zero-terminated strings, rather than two fixed-length arrays. As with `scas`{.nasm} in the last section, `cmps`{.nasm} can be made to work in this application by first performing a `scasb`{.nasm} pass to determine one string length and then comparing the strings with `cmpsw`{.nasm}, but the double pass negates much of the superior performance of `cmps`{.nasm}. [Listing 11-24](#listing-11-24) shows an implementation of this approach, which runs in 364 us for the test strings.

We found earlier that `lods`{.nasm} works well for string searching when multiple termination conditions must be dealt with. That is true of string comparison as well, particularly since there we can benefit from the combination of `scas`{.nasm} and `lods`{.nasm}. The `lodsw`{.nasm}/`scasw`{.nasm} approach, shown in [Listing 11-25](#listing-11-25), runs in just 306 us—19% faster than the `rep scasb`{.nasm}/`repz cmpsw`{.nasm}-based [Listing 11-24](#listing-11-24). For once, I won't bother with a non-string instruction-based implementation, since it's perfectly obvious that replacing `lodsw`{.nasm} and `scasw`{.nasm} with non-string sequences such as:

```nasm
mov   ax,[si]
inc   si
inc   si
```

and:

```nasm
cmp   [di],ax
      :
inc   di
inc   di
```

can only reduce performance.

`cmps`{.nasm} and even `scas`{.nasm} become still less suitable if a highly complex operation such as case-insensitive string comparison is required. Since both source and destination must be converted to the same case before being compared, both must be loaded into the registers for manipulation, and only `lods`{.nasm} among the string instructions will do us any good at all. [Listing 11-26](#listing-11-26) shows code that performs case-insensitive string comparison. [Listing 11-26](#listing-11-26) takes 869 us to run, which is not very fast by comparison with [Listings 11-21](#listing-11-21) through [11-25](#listing-11-25). That's to be expected, though, given the flexibility required for this comparison. The more flexibility required for a given task, the less likely we are to be able to bring the full power of the highly-specialized string instructions to bear on that task. That doesn't mean that we shouldn't try to do so, just that we won't always succeed.

If we're willing to expend 200 extra bytes or so, we can speed [Listing 11-26](#listing-11-26) up considerably with a clever trick. Making sure a character is uppercase takes a considerable amount of time even when all calculations are done in the registers, as is the case in [Listing 11-26](#listing-11-26). Fast as the instructions in the macro `TO_UPPER`{.nasm} in [Listing 11-26](#listing-11-26) are, two to five of them are executed every time a byte is made uppercase, and a time-consuming conditional jump may also be performed.

So what's better than two to five register-only instructions with at most one jump? A look-up table, that's what. [Listing 11-27](#listing-11-27) is a modification of [Listing 11-26](#listing-11-26) that looks up the uppercase version of each character in `ToUpperTable`{.nasm} with a single instruction—and the extremely fast and compact `xlat`{.nasm} instruction, at that. (It's possible that `mov`{.nasm} could be used instead of `xlat`{.nasm} to make an even faster version of [Listing 11-27](#listing-11-27), since `mov`{.nasm} can reference any general-purpose register while `xlat`{.nasm} can only load AL. As I've said, there are many ways to do anything in assembler.) For most characters there is no uppercase version, and the same character that we started with is looked up in `ToUpperTable`{.nasm}. For the 26 lowercase characters, however, the character looked up is the uppercase equivalent.

You may well be thinking that it doesn't make much sense to try to speed up code by *adding* a memory access, and normally you'd be right. However, `xlat`{.nasm} is very fast—it's a 1-byte instruction that executes in 10 cycles—and it saves us the trouble of fetching the many instruction bytes of `TO_UPPER`{.nasm}. (Remember, instruction fetches are memory accesses too.) What's more, `xlat`{.nasm} eliminates the need for conditional jumps in the uppercase-conversion process.

Sounds good in theory, doesn't it? It works just as well in the real world, too. [Listing 11-27](#listing-11-27) runs in just 638 us, a 36% improvement over [Listing 11-26](#listing-11-26). Of course, [Listing 11-27](#listing-11-27) is also a good deal larger than [Listing 11-26](#listing-11-26), owing to the look-up table, and that's a dilemma the assembler programmer faces frequently on the PC: the choice between speed and size. More memory, in the form of look-up tables and in-line code, often means better performance. It's actually relatively easy to speed up most code by throwing memory at it. The hard part is knowing where to strike the balance between performance and size.

Although both look-up tables and in-line code are discussed elsewhere in this volume, a broad discussion of the issue of memory versus performance will have to wait until Volume II of *The Zen of Assembly Language*. The mechanics of translating memory into performance—the knowledge aspect, if you will—is quite simple, but understanding when that tradeoff can and should be made is more complex and properly belongs in the discussion of the flexible mind.

### String Searching

Perhaps the single finest application of `cmps`{.nasm} is in searching for a sequence of bytes within a data buffer. In particular, `cmps`{.nasm} is excellent for finding a particular text sequence in a buffer full of text, as is the case when implementing a find-string capability in a text editor.

One way to implement such a searching capability is by simply starting `repz cmps`{.nasm} at each byte of the buffer until either a match is found or the end of the buffer is reached, as shown in Figure 11.14.

![](../images/fig11.14aRT.png)

![](../images/fig11.14bRT.png)

[Listing 11-28](#listing-11-28), which employs this approach, runs in 2995 us for the sample search sequence and buffer.

That's not bad, but there's a better way to go. Suppose we load the first byte of the search string into AL and use `repnz scasb`{.nasm} to find the next candidate for the full `repz cmps`{.nasm} comparison, as shown in Figure 11.15.

![](../images/fig11.15aRT.png)

![](../images/fig11.15bRT.png)

By so doing we could use a fast repeated string instruction to disqualify most of the potential strings, rather than having to loop and start up `repz cmps`{.nasm} at each and every byte in the buffer. Would that make a difference?

It would indeed! [Listing 11-29](#listing-11-29), which uses the hybrid `repnz scasb`{.nasm}/`repz cmps`{.nasm} technique, runs in just 719 us for the same search sequence and buffer as [Listing 11-28](#listing-11-28). Now, the margin between the two techniques could vary considerably, depending on the contents of the buffer and the search sequence. Nonetheless, we've just seen an improvement of more than 300% over already-fast string instruction-based code! That improvement is primarily due to the use of `repnz scasb`{.nasm} to eliminate most of the instruction fetches and branches of [Listing 11-28](#listing-11-28).

Even when you're using string instructions, stretch your mind to think of still-better approaches...

As for non-string implementations, [Listing 11-30](#listing-11-30), which performs the same task as do [Listings 11-28](#listing-11-28) and [11-29](#listing-11-29) but does so with non-string instructions, takes a full 3812 us to run. It should be very clear that non-string instructions should be used in searching applications only when their greater flexibility is absolutely required.

Make no mistake, there's more to searching performance than simply using the right combination of string instructions. The right choice of algorithm is critical. For a list of several thousand sorted items, a poorly-coded binary search might well beat the pants off a slick `repnz scasb`{.nasm}/`repz cmps`{.nasm} implementation. On the other hand, the `repnz scasb`{.nasm}/`repz cmps`{.nasm} approach is excellent for searching free-form data of the sort that's found in text buffers.

The key to searching performance lies in choosing a good algorithm for your application *and* implementing it with the best possible code. Either the searching algorithm or the implementation may be the factor that limits performance. Ideally, a searching algorithm would be chosen with an eye toward using the strengths of the 8088—and that usually means the string instructions.

### `cmps` Without `rep`

In the last chapter I pointed out that `scas`{.nasm} and `cmps`{.nasm} are slower but more flexible when they're not repeated. Although `repz`{.nasm} and `repnz`{.nasm} only allow termination according to the state of the Zero flag, `scas`{.nasm} and `cmps`{.nasm} actually set all the status flags, and we can take advantage of that when `scas`{.nasm} and `cmps`{.nasm} aren't repeated. Of course, we should use `repz`{.nasm} or `repnz`{.nasm} whenever we can, but non-repeated `scas`{.nasm} and `cmps`{.nasm} let us tap the power of string instructions when `repz`{.nasm} and `repnz`{.nasm} simply won't do.

For instance, suppose that we're comparing two arrays that contain signed 16-bit values representing signal measurements. Suppose further that we want to find the first point at which the waves represented by the arrays cross. That is, if wave A starts out above wave B, we want to know when wave A becomes less than or equal to wave B, as shown in Figure 11.16.

![](../images/fig11.17RT.png)

If wave B starts out above wave A, then we want to know when wave B becomes less than or equal to wave A.

There's no way to perform this comparison with repeated `cmps`{.nasm}, since greater-than/less-than comparisons aren't in the limited repertoire of the `rep`{.nasm} prefix. However, plain old non-repeated `cmpsw`{.nasm} is up to the task, as shown in [Listing 11-31](#listing-11-31), which runs in 1232 us. As shown in [Listing 11-31](#listing-11-31), we must initially determine which array starts out on top, in order to set SI to point to the initially-greater array and DI to point to the other array. Once that's done, all we need do is perform a `cmpsw`{.nasm} on each data point and check whether that point is still greater with `jg`{.nasm}. `loop`{.nasm} repeats the comparison for however many data points there are—and that's the whole routine in a very compact package! The 3-instruction, 5-byte loop of [Listing 11-31](#listing-11-31) is hard to beat for this fairly demanding task.

By contrast, [Listing 11-32](#listing-11-32), which performs the same crossing search but does so with non-string instructions, has 6 instructions and 13 bytes in the loop and takes considerably longer—1821 us—to complete the sample crossing search. Although we were unable to use repeated `cmps`{.nasm} for this particular task, we were nonetheless able to improve performance a great deal by using the string instruction in its non-repeated form.

## A Note About Returning Values

Throughout this chapter I've been returning "not found" statuses by passing zero pointers (pointers set to zero) back to the calling routine. This is a commonly used and very flexible means of returning such statuses, since the same registers that are used to return pointers when searches are successful can be used to return zero when searches are not successful. The success or failure of a subroutine can then be tested with code like:

```nasm
call  FindCharInString
and   si,si
jz    CharNotFound
```

Returning failure statuses as zero pointers is particularly popular in high-level languages such as C, although C returns pointers in either AX, DX:AX, or memory, rather than in SI or DI.

However, there are many other ways of returning statuses in assembler. One particularly effective approach is that of returning success or failure in either the Zero or Carry flag, so that the calling routine can immediately jump conditionally upon return from the subroutine, without the need for any anding, oring, or comparing of any sort. This works out especially well when the proper setting of a flag falls out of the normal functioning of a subroutine. For example, consider the following subroutine, which returns the Zero flag set to 1 if the character in AL is whitespace:

```nasm
Whitespace:
    cmp   al,' '          ;space
    jz    WhitespaceDone
    cmp   al,9            ;tab
    jz    WhitespaceDone
    and   al,al           ;zero byte
WhitespaceDone:
    ret
```

The key point here is that the Zero flag is automatically set by the comparisons preceding the `ret`{.nasm}. Any test for whitespace would have to perform the same comparisons, so practically speaking we didn't have to write a single extra line of code to return the subroutine's status in the Zero flag. Because the return status is in a flag rather than a register, `Whitespace`{.nasm} could be called and the outcome handled with a very short sequence of instructions, as follows:

```nasm
mov   al,[Char]
call  Whitespace
jnz   NotWhitespace
```

The particular example isn't important here. What is important is that you realize that in assembler (unlike high-level languages) there are many ways to return statuses, and that it's possible to save a great deal of code and/or time by taking advantage of that. Now is not the time to pursue the topic further, but we'll return to the issues of passing values and statuses both to and from assembler subroutines in Volume II of *The Zen of Assembly Language*.

## Putting String Instructions to Work in Unlikely Places

I've said several times that string instructions are so powerful that you should try to use them even when they don't seem especially well-matched to a particular application. Now I'm going to back that up with an unlikely application in which the string instructions have served me well over the years: animation.

This section is actually a glimpse into the future. Volume II of *The Zen of Assembly Language* will take up the topic of animation in much greater detail, since animation truly falls in the category of the flexible mind rather than knowledge. Still, animation is such a wonderful example of what the string instructions can do that we'll spend a bit of time on it here and now. It'll be a whirlwind look, with few details and nothing more than a quick glance at theory, for the focus isn't on animation *per se*. What's important is not that you understand how animation works, but rather that you get a feel for the miracles string instructions can perform in places where you wouldn't think they could serve at all.

### Animation Basics

Animation involves erasing and redrawing one or more images quickly enough to fool the eye into perceiving motion, as shown in Figure 11.17.

![](../images/fig11.17RT.png)

Animation is a marginal application for the PC, by which I mean that the 8088 barely has enough horsepower to support decent animation under the best of circumstances. What *that* means is that the Zen of assembler is an absolute must for PC animation.

Traditionally, microcomputer animation has been performed by exclusive-oring images into display memory; that is, by drawing images by inserting the bits that control their pixels into display memory with the `xor`{.nasm} instruction. When an image is first exclusive-ored into display memory at a given location, the image becomes visible. A second exclusive-oring of the image at the same location then erases the image. Why? That's simply the nature of the exclusive-or operation.

Consider this. When you exclusive-or a 1 bit with another bit once, the other bit is flipped. When you exclusive-or the same 1 bit with that other bit again, the other bit is again flipped—*right back to its original state*, as shown in Figure 11. 18.

![](../images/fig11.18RT.png)

After all, a bit only has two possible states, so a double flip must restore the bit back to the state in which it started. Since exclusive-oring a 0 bit with another bit never affects the other bit, exclusive-oring a target bit twice with either a 1 or a 0 bit always leaves the target bit in its original state.

Why is exclusive-oring so popular for animation? Simply because no matter how many images overlap, the second exclusive-or of an image always erases it without interfering with any other images. In other words, the perfect reversibility of the exclusive-or operation means that you could exclusive-or each of 10 images once at the same location, drawing the images right on top of each other, then exclusive-or them all again at the same place—and they would all be erased. With exclusive-oring, the drawing or erasing of one image never interferes with the drawing or erasing of other images it overlaps.

If you're catching all this, great. If not, don't worry. I'm not going to spend time explaining animation now—better we should wait until Volume II, when we have the time to do it right. The important point is that exclusive-oring is a popular animation technique, primarily because it eliminates the complications of drawing and erasing overlapping images.

[Listing 11-33](#listing-11-33), which bounces 10 images around the screen, illustrates animation based on exclusive-oring. When run on an Enhanced Graphics Adapter (EGA), [Listing 11-33](#listing-11-33) takes 30.29 seconds to move and redraw every image 500 times. (Note that the long-period Zen timer was used to time [Listing 11-33](#listing-11-33), since we can't perform much animation within the 54 ms maximum period of the precision Zen timer.)

[Listing 11-33](#listing-11-33) isn't a general-purpose animation program. I've kept complications to a minimum in order to show basic exclusive-or animation. [Listing 11-33](#listing-11-33) allows us to observe the fundamental strengths and weaknesses (primarily the latter) of the exclusive-or approach.

When you run [Listing 11-33](#listing-11-33), you'll see why exclusive-oring is less than ideal. While overlapping images don't interfere with each other so far as drawing and erasing go, they do produce some unattractive on-screen effects. In particular, unintended colors and patterns often result when multiple images are exclusive-ored into the same bytes of display memory. Another problem is that exclusive-ored images flicker because they're constantly being erased and redrawn. (Each image could instead be redrawn at its new location before being erased at the old location, but the overlap effects characteristic of exclusive-oring would still cause flicker.) That's not all, though. There's a still more serious problem with exclusive-or based animation...

Exclusive-oring is slow.

The problem isn't that the `xor`{.nasm} instruction itself is particular slow; rather, it's that the `xor`{.nasm} instruction isn't a string instruction. `xor`{.nasm} can't be repeated with `rep`{.nasm}, it doesn't advance its pointers automatically, and it just isn't as speedy as, say, `movs`{.nasm}. Still, neither `movs`{.nasm} nor any other string instruction can perform exclusive-or operations, so it would seem we're stuck.

We're hardly stuck, though. On the contrary, we're bound for glory!

### String Instruction-Based Animation

If string instructions can't perform exclusive-oring, then we'll just have to figure out a way to animate without exclusive-oring. As it turns out, there's a *very* nice way to do this. I learned this approach from Dan Illowsky, who developed it before string instructions even existed, way back in the early days of the Apple II.

First, we'll give each image a small blank fringe. Then we'll make it a rule never to move an image by more than the width of its fringe before redrawing it. Finally we'll draw images by simply copying them to display memory, destroying whatever they overwrite, as shown in Figure 11.19. Now, what does that do for us?

![](../images/fig11.19aRT.png)

![](../images/fig11.19bRT.png)

*Amazing* things. For starters, each image will, as it is redrawn, automatically erase its former incarnation. That means that there's no flicker, since images are never really erased, but only drawn over themselves. There are also no color effects when images overlap, since only the image that was drawn most recently at any given pixel is visible.

In short, this sort of animation (which I'll call "block-move animation") actually looks considerably better than animation based on exclusive-oring. That's just frosting on the cake, though—the big payoff is speed. With block-move animation we suddenly don't need to exclusive-or anymore—in fact, `rep movs`{.nasm} will work beautifully to draw a whole line of an image in a single instruction. We also don't need to draw each image twice per move—once to erase the image at its old location and once to draw it at its new location—as we did with exclusive-oring, since the act of drawing the image at a new location serves to erase the old image as well. But wait, there's more! `xor`{.nasm} accesses a given byte of memory twice per draw, once to read the original byte and once to write the modified byte back to memory. With block-move animation, on the other hand, we simply write each byte of an image to memory once and we're done with that byte. In other words, between the elimination of a separate erasing step and the replacement of read-`xor`{.nasm}-write with a single write, block-move animation accesses display memory only about one-third as many times as exclusive-or animation. (The ratio isn't quite 1 to 4 because the blank fringe makes block-move animation images somewhat larger.)

Are alarm bells going off in your head? They should be. Think back to our journey beneath the programming interface. Think of the cycle-eaters. Ah, you've got it! *Exclusive-or animation loses about three times as much performance to the display adapter cycle-eater as does block-move animation.* What's more, block-move animation uses the blindingly fast `movs`{.nasm} instruction. To top it off, block-move animation loses almost nothing to the prefetch queue cycle-eater or the 8088's slow branching speed, thanks to the `rep`{.nasm} prefix.

Sounds almost too good to be true, doesn't it? It is true, though: block-move animation relies almost exclusively on one of the two most powerful instructions of the 8088 (`cmps`{.nasm} being the other), and avoids the gaping maws of the prefetch queue and display adapter cycle-eaters in the process. Which leaves only one question:

How fast *is* block-move animation?

Remember, theory is fine, but we don't trust any code until we've timed it. [Listing 11-34](#listing-11-34) performs the same animation as [Listing 11-34](#listing-11-34), but with block-move rather than exclusive-or animation. Happily, [Listing 11-34](#listing-11-34) lives up to its advance billing, finishing in just 10.35 seconds when run on an EGA. *Block-move animation is close to three times as fast as exclusive-oring in this application*—and it looks better, too. (You can slow down the animation in order to observe the differences between the two sorts of animation more closely by setting `DELAY`{.nasm} to a higher value in each listing.)

Let's not underplay the appearance issue just because the performance advantage of block-move animation is so great. If you possibly can, enter and run [Listings 11-33](#listing-11-33) and [11-34](#listing-11-34). The visual impact of block-move animation's flicker-free, high-speed animation is startling. It's hard to imagine that any programmer would go back to exclusive-oring after seeing block-move animation in action.

That's not to say that block-move animation is perfect. Unlike exclusive-oring, block-move animation wipes out the background unless the background is explicitly redrawn after each image is moved. Block-move animation does produce flicker and fringe effects when images overlap. Block-move animation also limits the maximum distance by which an image can move before it's redrawn to the width of its fringe.

If block-move animation isn't perfect, however, it's *much* better than exclusive-oring. What's really noteworthy, however, is that we looked at an application—animation—without preconceived ideas about the best implementation, and came up with an approach that merged the application's needs with one of the strengths of the PC—the string instructions—while avoiding the cycle-eaters. In the end, we not only improved performance remarkably but also got better animation, in the process turning a seeming minus—the limitations of the string instructions—into a big plus. All in all, what we've just done is the Zen of assembler working on all levels: knowledge, flexible mind, and implementation.

Try to use the string instructions for all your time-critical code, even when you think they just don't fit. Sometimes they don't—but you can never be sure unless you try... and if they *can* be made to fit, it will pay off *big*.

### Notes on the Animation Implementations

Spend as much time as you wish perusing [Listings 11-33](#listing-11-33) and [11-34](#listing-11-34), but *do not worry* if they don't make complete sense to you right now. The point of this exercise was to illustrate the use of the string instructions in an unusual application, not to get you started with animation. In Volume II of *The Zen of Assembly Language* we'll return to animation in a big way.

The animation listings are not full-featured, flexible implementations, nor were they meant to be. My intent in creating these programs was to contrast the basic operation and raw performance of exclusive-or and block-move animation. Consequently, I've structured the two listings along much the same lines, and while the code is fast, I've avoided further optimizations (notably the use of in-line code) that would have complicated matters. We'll see those additional optimizations in Volume II.

One interesting point to be made about the animation listings is that I've assumed in the drawing routines that images always start on even rows of the screen and are always an even number of rows in height. Many people would consider the routines to be incomplete, since they lack the extra code needed to handle the complications of odd start rows and odd heights in 320x200 4-color graphics mode. Of course, that extra code would slow performance and increase program size, but would be deemed necessary in any "full" animation implementation.

Is the handling of odd start rows and odd heights really necessary, though? Not if you can structure your application so that images can always start on even rows and can always be of even heights, and that's actually easy to do. No one will ever notice whether images move 1 or 2 pixels at a time; the nature of animation is such that the motion of an image appears just as smooth in either case. And why should there be a need for odd image heights? If necessary, images of odd height could be padded out with an extra line. In fact, an extra line can often be used to improve the appearance of an image.

In short, "full" animation implementations will not only run slower than the implementation in [Listings 11-33](#listing-11-33) and [11-34](#listing-11-34) but may not even yield any noticeable benefits. The lesson is this: only add features that slow your code when you're sure you need them. High-performance assembler programming is partly an art of eliminating everything but the essentials.

By the way, [Listings 11-33](#listing-11-33) and [11-34](#listing-11-34) move images a full 4 pixels at a time horizontally, and that's a bit *too* far. 2 pixels is a far more visually attractive distance by which to move animated images, especially those that move slowly. However, because each byte of 320x200 4-color mode display memory controls 4 pixels, alignment of images to start in columns that aren't multiples of 4 is more difficult, although not really that hard once you get the hang of it. Since our goal in this section was to contrast block-move and exclusive-or animation, I didn't add the extra code and complications required to bit-align the images. We will discuss bit-alignment of images at length in Volume II, however.

## A Note on Handling Blocks Larger Than 64 K Bytes

All the string instruction-based code we've seen in this chapter handles only blocks or strings that are 64 K bytes in length or shorter. There's a very good reason for this, of course—the infernal segmented architecture of the 8088—but there are nonetheless times when larger memory blocks are needed.

I'm going to save the topic of handling blocks larger than 64 K bytes for Volume II of *The Zen of Assembly Language*. Why? Well, the trick with code that handles larger memory blocks isn't getting it to work; that's relatively easy if you're willing to perform 32-bit arithmetic and reload the segment registers before each memory access. No, the trick is getting code that handles large memory blocks to work reasonably fast.

We've seen that a key to assembler programming lies in converting difficult problems from approaches ill-suited to the 8088 to ones that the 8088 can handle well, and this is no exception. In this particular application, we need to convert the task at hand from one of independently addressing every byte in the 8088's 1-megabyte address space to one of handling a series of blocks that are each no larger than 64 K bytes, so that we can process up to 64 K bytes at a time very rapidly without touching the segment registers.

The concept is simple, but the implementation is not so simple and requires the flexible mind... and that's why the handling of memory blocks larger than 64 K bytes will have to wait until Volume II.

### Conclusion

This chapter had two objectives. First, I wanted you to get a sense of how and when the string instructions can best be applied. Second, I wanted you to heighten your regard for these instructions, which are the best the 8088 has to offer. With any luck, this chapter has both broadened your horizons for string instruction applications and increased your respect for these unique and uniquely powerful members of the 8088's instruction set.

## Listing 11-1

```nasm
;
; *** Listing 11-1 ***
;
; Copies a string to another string, converting all
; characters to uppercase in the process, using a loop
; containing LODSB and STOSB.
;
jmp Skip
;
SourceString label word
db 'This space intentionally left not blank',0
DestString db 100 dup (?)
;
; Copies one zero-terminated string to another string,
; converting all characters to uppercase.
;
; Input:
; DS:SI = start of source string
; ES:DI = start of destination string
;
; Output:
; none
;
; Registers altered: AL, BX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries. Does not handle
; overlapping strings.
;
CopyStringUpper:
mov bl,'a' ;set up for fast register-register
mov bh,'z' ; comparisons
cld
StringUpperLoop:
lodsb ;get the next character and
; point to the following character
cmp al,bl ;below 'a'?
jb IsUpper ;yes, not lowercase
cmp al,bh ;above 'z'?
ja IsUpper ;yes, not lowercase
and al,not 20h ;is lowercase-make uppercase
IsUpper:
stosb ;put the uppercase character into
; the new string and point to the
; following character
and al,al ;is this the zero that marks the
; end of the string?
jnz StringUpperLoop ;no, do the next character
ret
;
Skip:
call ZTimerOn
mov si,offset SourceString ;point DS:SI to the
; string to copy from
mov di,seg DestString
mov es,di ;point ES:DI to the
mov di,offset DestString ; string to copy to
call CopyStringUpper ;copy & convert to
; uppercase
call ZTimerOff
```

## Listing 11-2

```nasm
;
; *** Listing 11-2 ***
;
; Copies a string to another string, converting all
; characters to uppercase in the process, using a loop
; containing non-string instructions.
;
jmp Skip
;
SourceString label word
db 'This space intentionally left not blank',0
DestString db 100 dup (?)
;
; Copies one zero-terminated string to another string,
; converting all characters to uppercase.
;
; Input:
; DS:SI = start of source string
; ES:DI = start of destination string
;
; Output:
; none
;
; Registers altered: AL, BX, SI, DI
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
CopyStringUpper:
mov bl,'a' ;set up for fast register-register
mov bh,'z' ; comparisons
StringUpperLoop:
mov al,[si] ;get the next character
inc si ;point to the following character
cmp al,bl ;below 'a'?
jb IsUpper ;yes, not lowercase
cmp al,bh ;above 'z'?
ja IsUpper ;yes, not lowercase
and al,not 20h ;is lowercase-make uppercase
IsUpper:
mov es:[di],al ;put the uppercase character into
; the new string
inc di ;point to the following character
and al,al ;is this the zero that marks the
; end of the string?
jnz StringUpperLoop ;no, do the next character
ret
;
Skip:
call ZTimerOn
mov si,offset SourceString ;point DS:SI to the
; string to copy from
mov di,seg DestString
mov es,di ;point ES:DI to the
mov di,offset DestString ; string to copy to
call CopyStringUpper ;copy & convert to
; uppercase
call ZTimerOff
```

## Listing 11-3

```nasm
;
; *** Listing 11-3 ***
;
; Converts all characters in a string to uppercase,
; using a loop containing LODSB and STOSB and using
; two pointers.
;
jmp Skip
;
SourceString label word
db 'This space intentionally left not blank',0
;
; Copies one zero-terminated string to another string,
; converting all characters to uppercase.
;
; Input:
; DS:SI = start of source string
; ES:DI = start of destination string
;
; Output:
; none
;
; Registers altered: AL, BX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
CopyStringUpper:
mov bl,'a' ;set up for fast register-register
mov bh,'z' ; comparisons
cld
StringUpperLoop:
lodsb ;get the next character and
; point to the following character
cmp al,bl ;below 'a'?
jb IsUpper ;yes, not lowercase
cmp al,bh ;above 'z'?
ja IsUpper ;yes, not lowercase
and al,not 20h ;is lowercase-make uppercase
IsUpper:
stosb ;put the uppercase character into
; the new string and point to the
; following character
and al,al ;is this the zero that marks the
; end of the string?
jnz StringUpperLoop ;no, do the next character
ret
;
Skip:
call ZTimerOn
mov si,offset SourceString ;point DS:SI to the
; string to convert
mov di,ds
mov es,di ;point ES:DI to the
mov di,si ; same string
call CopyStringUpper ;convert to
; uppercase in place
call ZTimerOff
```

## Listing 11-4

```nasm
;
; *** Listing 11-4 ***
;
; Converts all characters in a string to uppercase,
; using a loop containing non-string instructions
; and using only one pointer.
;
jmp Skip
;
SourceString label word
db 'This space intentionally left not blank',0
;
; Converts a string to uppercase.
;
; Input:
; DS:SI = start of string
;
; Output:
; none
;
; Registers altered: AL, BX, SI
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
StringToUpper:
mov bl,'a' ;set up for fast register-register
mov bh,'z' ; comparisons
StringToUpperLoop:
mov al,[si] ;get the next character
cmp al,bl ;below 'a'?
jb IsUpper ;yes, not lowercase
cmp al,bh ;above 'z'?
ja IsUpper ;yes, not lowercase
and al,not 20h ;is lowercase-make uppercase
IsUpper:
mov [si],al ;put the uppercase character back
inc si ; into the string and point to the
; following character
and al,al ;is this the zero that marks the
; end of the string?
jnz StringToUpperLoop ;no, do the next character
ret
;
Skip:
call ZTimerOn
mov si,offset SourceString ;point to the string
; to convert
call StringToUpper ;convert it to
; uppercase
call ZTimerOff
```

## Listing 11-5

```nasm
; *** Listing 11-5 ***
;
; Sets the high bit of every element in a byte
; array using LODSB and STOSB.
;
jmp Skip
;
;
ARRAY_LENGTH equ 1000
ByteArray db ARRAY_LENGTH dup (?)
;
Skip:
call ZTimerOn
mov si,offset ByteArray ;point to the array
mov di,ds ; as both source and
mov es,di ; destination
mov di,si
mov cx,ARRAY_LENGTH
mov ah,80h ;bit pattern to OR
cld
SetHighBitLoop:
lodsb ;get the next byte
or al,ah ;set the high bit
stosb ;save the byte
loop SetHighBitLoop
call ZTimerOff
```

## Listing 11-6

```nasm
; *** Listing 11-6 ***
;
; Sets the high bit of every element in a byte
; array by ORing directly to memory.
;
jmp Skip
;
;
ARRAY_LENGTH equ 1000
ByteArray db ARRAY_LENGTH dup (?)
;
Skip:
call ZTimerOn
mov si,offset ByteArray ;point to the array
mov cx,ARRAY_LENGTH
mov al,80h ;bit pattern to OR
SetHighBitLoop:
or [si],al ;set the high bit
inc si ;point to the next
; byte
loop SetHighBitLoop
call ZTimerOff
```

## Listing 11-7

```nasm
;
; *** Listing 11-7 ***
;
; Copies overlapping blocks of memory with MOVS.
; To the greatest possible extent, the copy is
; performed a word at a time.
;
jmp Skip
;
TEST_LENGTH1 equ 501 ;sample copy length #1
TEST_LENGTH2 equ 1499 ;sample copy length #2
TestArray db 1500 dup (0)
;
; Copies a block of memory CX bytes in length. A value
; of 0 means "copy zero bytes," since it wouldn't make
; much sense to copy one 64K block to another 64K block
; in the same segment, so the maximum length that can
; be copied is 64K-1 bytes and the minimum length
; is 0 bytes. Note that both blocks must be in DS. Note
; also that overlap handling is not guaranteed if either
; block wraps at the end of the segment.
;
; Input:
; CX = number of bytes to clear
; DS:SI = start of block to copy
; DS:DI = start of destination block
;
; Output:
; none
;
; Registers altered: CX, DX, SI, DI, ES
;
; Direction flag cleared
;
BlockCopyWithOverlap:
mov dx,ds ;source and destination are in the
mov es,dx ; same segment
cmp si,di ;which way do the blocks overlap, if
; they do overlap?
jae LowToHigh
;source is not below destination, so
; we can copy from low to high

;source is below destination, so we
; must copy from high to low
add si,cx ;point to the end of the source
dec si ; block
add di,cx ;point to the end of the destination
dec di ; block
std ;copy from high addresses to low
shr cx,1 ;divide by 2, copying the odd-byte
; status to the Carry flag
jnc CopyWordHighToLow ;no odd byte to copy
movsb ;copy the odd byte
CopyWordHighToLow:
dec si ;point one word lower in memory, not
dec di ; one byte
rep movsw ;move the rest of the block
cld
ret
;
LowToHigh:
cld ;copy from low addresses to high
shr cx,1 ;divide by 2, copying the odd-byte
; status to the Carry flag
jnc CopyWordLowToHigh ;no odd byte to copy
movsb ;copy the odd byte
CopyWordLowToHigh:
rep movsw ;move the rest of the block
ret
;
Skip:
call ZTimerOn
;
; First run the case where the destination overlaps & is
; higher in memory.
;
mov si,offset TestArray
mov di,offset TestArray+1
mov cx,TEST_LENGTH1
call BlockCopyWithOverlap
;
; Now run the case where the destination overlaps & is
; lower in memory.
;
mov si,offset TestArray+1
mov di,offset TestArray
mov cx,TEST_LENGTH2
call BlockCopyWithOverlap
call ZTimerOff
```

## Listing 11-8

```nasm
;
; *** Listing 11-8 ***
;
; Copies overlapping blocks of memory with
; non-string instructions. To the greatest possible
; extent, the copy is performed a word at a time.
;
jmp Skip
;
TEST_LENGTH1 equ 501 ;sample copy length #1
TEST_LENGTH2 equ 1499 ;sample copy length #2
TestArray db 1500 dup (0)
;
; Copies a block of memory CX bytes in length. A value
; of 0 means "copy zero bytes," since it wouldn't make
; much sense to copy one 64K block to another 64K block
; in the same segment, so the maximum length that can
; be copied is 64K-1 bytes and the minimum length
; is 0 bytes. Note that both blocks must be in DS. Note
; also that overlap handling is not guaranteed if either
; block wraps at the end of the segment.
;
; Input:
; CX = number of bytes to clear
; DS:SI = start of block to copy
; DS:DI = start of destination block
;
; Output:
; none
;
; Registers altered: AX, CX, DX, SI, DI
;
BlockCopyWithOverlap:
jcxz BlockCopyWithOverlapDone
;guard against zero block size,
; since LOOP will execute 64K times
; when started with CX=0
mov dx,2 ;amount by which to adjust the
; pointers in the word-copy loop
cmp si,di ;which way do the blocks overlap, if
; they do overlap?
jae LowToHigh
;source is not below destination, so
; we can copy from low to high

;source is below destination, so we
; must copy from high to low
add si,cx ;point to the end of the source
dec si ; block
add di,cx ;point to the end of the destination
dec di ; block
shr cx,1 ;divide by 2, copying the odd-byte
; status to the Carry flag
jnc CopyWordHighToLow ;no odd byte to copy
mov al,[si] ;copy the odd byte
mov [di],al
dec si ;advance both pointers
dec di
CopyWordHighToLow:
dec si ;point one word lower in memory, not
dec di ; one byte
HighToLowCopyLoop:
mov ax,[si] ;copy a word
mov [di],ax
sub si,dx ;advance both pointers 1 word
sub di,dx
loop HighToLowCopyLoop
ret
;
LowToHigh:
shr cx,1 ;divide by 2, copying the odd-byte
; status to the Carry flag
jnc LowToHighCopyLoop ;no odd byte to copy
mov al,[si] ;copy the odd byte
mov [di],al
inc si ;advance both pointers
inc di
LowToHighCopyLoop:
mov ax,[si] ;copy a word
mov [di],ax
add si,dx ;advance both pointers 1 word
add di,dx
loop LowToHighCopyLoop
BlockCopyWithOverlapDone:
ret
;
Skip:
call ZTimerOn
;
; First run the case where the destination overlaps & is
; higher in memory.
;
mov si,offset TestArray
mov di,offset TestArray+1
mov cx,TEST_LENGTH1
call BlockCopyWithOverlap
;
; Now run the case where the destination overlaps & is
; lower in memory.
;
mov si,offset TestArray+1
mov di,offset TestArray
mov cx,TEST_LENGTH2
call BlockCopyWithOverlap
call ZTimerOff
```

## Listing 11-9

```nasm
;
; *** Listing 11-9 ***
;
; Counts the number of times the letter 'A'
; appears in a byte-sized array, using non-string
; instructions.
;
jmp Skip
;
ByteArray label byte
db 'ARRAY CONTAINING THE LETTER ''A'' 4 TIMES'
ARRAY_LENGTH equ ($-ByteArray)
;
; Counts the number of occurrences of the specified byte
; in the specified byte-sized array.
;
; Input:
; AL = byte of which to count occurrences
; CX = array length (0 means 64K)
; DS:DI = array to count byte occurrences in
;
; Output:
; DX = number of occurrences of the specified byte
;
; Registers altered: CX, DX, DI
;
; Note: Does not handle arrays that are longer than 64K
; bytes or cross segment boundaries.
;
ByteCount:
sub dx,dx ;set occurrence counter to 0
dec di ;compensate for the initial
; upcoming INC DI
and cx,cx ;64K long?
jnz ByteCountLoop ;no
dec cx ;yes, so handle first byte
; specially, since JCXZ will
; otherwise conclude that
; we're done right away
inc di ;point to first byte
cmp [di],al ;is this byte the value
; we're looking for?
jz ByteCountCountOccurrence
;yes, so count it
ByteCountLoop:
jcxz ByteCountDone ;done if we've checked all
; the bytes in the array
dec cx ;count off the byte we're
; about to check
inc di ;point to the next byte to
; check
cmp [di],al ;see if this byte contains
; the value we're counting
jnz ByteCountLoop ;no match
ByteCountCountOccurrence:
inc dx ;count this occurrence
jmp ByteCountLoop ;check the next byte, if any
ByteCountDone:
ret
;
Skip:
call ZTimerOn
mov al,'A' ;byte of which we want a
; count of occurrences
mov di,offset ByteArray
;array we want a count for
mov cx,ARRAY_LENGTH ;# of bytes to check
call ByteCount ;get the count
call ZTimerOff
```

## Listing 11-10

```nasm
;
; *** Listing 11-10 ***
;
; Counts the number of times the letter 'A'
; appears in a byte-sized array, using REPNZ SCASB.
;
jmp Skip
;
ByteArray label byte
db 'ARRAY CONTAINING THE LETTER ''A'' 4 TIMES'
ARRAY_LENGTH equ ($-ByteArray)
;
; Counts the number of occurrences of the specified byte
; in the specified byte-sized array.
;
; Input:
; AL = byte of which to count occurrences
; CX = array length (0 means 64K)
; DS:DI = array to count byte occurrences in
;
; Output:
; DX = number of occurrences of the specified byte
;
; Registers altered: CX, DX, DI, ES
;
; Direction flag cleared
;
; Note: Does not handle arrays that are longer than 64K
; bytes or cross segment boundaries. Does not handle
; overlapping strings.
;
ByteCount:
push ds
pop es ;SCAS uses ES:DI
sub dx,dx ;set occurrence counter to 0
cld
and cx,cx ;64K long?
jnz ByteCountLoop ;no
dec cx ;yes, so handle first byte
; specially, since JCXZ will
; otherwise conclude that
; we're done right away
scasb ;is first byte a match?
jz ByteCountCountOccurrence
;yes, so count it
ByteCountLoop:
jcxz ByteCountDone ;if there's nothing left to
; search, we're done
repnz scasb ;search for the next byte
; occurrence or the end of
; the array
jnz ByteCountDone ;no match
ByteCountCountOccurrence:
inc dx ;count this occurrence
jmp ByteCountLoop ;check the next byte, if any
ByteCountDone:
ret
;
Skip:
call ZTimerOn
mov al,'A' ;byte of which we want a
; count of occurrences
mov di,offset ByteArray
;array we want a count for
mov cx,ARRAY_LENGTH ;# of bytes to check
call ByteCount ;get the count
call ZTimerOff
```

## Listing 11-11

```nasm
;
; *** Listing 11-11 ***
;
; Finds the first occurrence of the letter 'z' in
; a zero-terminated string, using LODSB.
;
jmp Skip
;
TestString label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
;
; Finds the first occurrence of the specified byte in the
; specified zero-terminated string.
;
; Input:
; AL = byte to find
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to first occurrence of byte in string,
; or 0 if the byte wasn't found
;
; Registers altered: AX, SI
;
; Direction flag cleared
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a match on the first byte and failure to find
; the byte would be indistinguishable.
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
FindCharInString:
mov ah,al ;we'll need AL since that's the
; only register LODSB can use
cld
FindCharInStringLoop:
lodsb ;get the next string byte
cmp al,ah ;is this the byte we're
; looking for?
jz FindCharInStringDone
;yes, so we're done
and al,al ;is this the terminating zero?
jnz FindCharInStringLoop
;no, so check the next byte
sub si,si ;we didn't find a match, so return
; 0 in SI
ret
FindCharInStringDone:
dec si ;point back to the matching byte
ret
;
Skip:
call ZTimerOn
mov al,'z' ;byte value to find
mov si,offset TestString
;string to search
call FindCharInString ;search for the byte
call ZTimerOff
```

## Listing 11-12

```nasm
;
; *** Listing 11-12 ***
;
; Finds the first occurrence of the letter 'z' in
; a zero-terminated string, using REPNZ SCASB in a
; double-search approach, first finding the terminating
; zero to determine the string length, and then searching
; for the desired byte.
;
jmp Skip
;
TestString label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
;
; Finds the first occurrence of the specified byte in the
; specified zero-terminated string.
;
; Input:
; AL = byte to find
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to first occurrence of byte in string,
; or 0 if the byte wasn't found
;
; Registers altered: AH, CX, SI, DI, ES
;
; Direction flag cleared
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a match on the first byte and failure to find
; the byte would be indistinguishable.
;
; Note: If the search value is 0, will not find the
; terminating zero in a string that is exactly 64K
; bytes long. Does not handle strings that are longer
; than 64K bytes or cross segment boundaries.
;
FindCharInString:
mov ah,al ;set aside the byte to be found
sub al,al ;we'll search for zero
push ds
pop es
mov di,si ;SCAS uses ES:DI
mov cx,0ffffh ;long enough to handle any string
; up to 64K-1 bytes in length, and
; will handle 64K case except when
; the search value is the terminating
; zero
cld
repnz scasb ;find the terminating zero
not cx ;length of string in bytes, including
; the terminating zero except in the
; case of a string that's exactly 64K
; long including the terminating zero
mov al,ah ;get back the byte to be found
mov di,si ;point to the start of the string again
repnz scasb ;search for the byte of interest
jnz FindCharInStringNotFound
;the byte isn't present in the string
dec di ;we've found the desired value. Point
; back to the matching location
mov si,di ;return the pointer in SI
ret
FindCharInStringNotFound:
sub si,si ;return a 0 pointer indicating that
; no match was found
ret
;
Skip:
call ZTimerOn
mov al,'z' ;byte value to find
mov si,offset TestString
;string to search
call FindCharInString ;search for the byte
call ZTimerOff
```

## Listing 11-13

```nasm
;
; *** Listing 11-13 ***
;
; Finds the first occurrence of the letter 'z' in
; a zero-terminated string, using non-string instructions.
;
jmp Skip
;
TestString label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
;
; Finds the first occurrence of the specified byte in the
; specified zero-terminated string.
;
; Input:
; AL = byte to find
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to first occurrence of byte in string,
; or 0 if the byte wasn't found
;
; Registers altered: AH, SI
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a match on the first byte and failure to find
; the byte would be indistinguishable.
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
FindCharInString:
FindCharInStringLoop:
mov ah,[si] ;get the next string byte
cmp ah,al ;is this the byte we're
; looking for?
jz FindCharInStringDone
;yes, so we're done
inc si ;point to the following byte
and ah,ah ;is this the terminating zero?
jnz FindCharInStringLoop
;no, so check the next byte
sub si,si ;we didn't find a match, so return
; 0 in SI
FindCharInStringDone:
ret
;
Skip:
call ZTimerOn
mov al,'z' ;byte value to find
mov si,offset TestString
;string to search
call FindCharInString ;search for the byte
call ZTimerOff
```

## Listing 11-14

```nasm
; *** Listing 11-14 ***
;
; Finds the first occurrence of the letter 'z' in
; a zero-terminated string, using LODSW and checking
; 2 bytes per read.
;
jmp Skip
;
TestString label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
;
; Finds the first occurrence of the specified byte in the
; specified zero-terminated string.
;
; Input:
; AL = byte to find
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to first occurrence of byte in string,
; or 0 if the byte wasn't found
;
; Registers altered: AX, BL, SI
;
; Direction flag cleared
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a match on the first byte and failure to find
; the byte would be indistinguishable.
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
FindCharInString:
mov bl,al ;we'll need AX since that's the
; only register LODSW can use
cld
FindCharInStringLoop:
lodsw ;get the next 2 string bytes
cmp al,bl ;is the first byte the byte we're
; looking for?
jz FindCharInStringDoneAdjust
;yes, so we're done after we adjust
; back to the first byte of the word
and al,al ;is the first byte the terminating
; zero?
jz FindCharInStringNoMatch ;yes, no match
cmp ah,bl ;is the second byte the byte we're
; looking for?
jz FindCharInStringDone
;yes, so we're done
and ah,ah ;is the second byte the terminating
; zero?
jnz FindCharInStringLoop
;no, so check the next 2 bytes
FindCharInStringNoMatch:
sub si,si ;we didn't find a match, so return
; 0 in SI
ret
FindCharInStringDoneAdjust:
dec si ;adjust to the first byte of the
; word we just read
FindCharInStringDone:
dec si ;point back to the matching byte
ret
;
Skip:
call ZTimerOn
mov al,'z' ;byte value to find
mov si,offset TestString
;string to search
call FindCharInString ;search for the byte
call ZTimerOff
```

## Listing 11-15

```nasm
;
; *** Listing 11-15 ***
;
; Finds the last non-blank character in a string, using
; LODSW and checking 2 bytes per read.
;
jmp Skip
;
TestString label byte
db 'This is a test string with blanks....'
db ' ',0
;
; Finds the last non-blank character in the specified
; zero-terminated string.
;
; Input:
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to last non-blank character in string,
; or 0 if there are no non-blank characters in
; the string
;
; Registers altered: AX, BL, DX, SI
;
; Direction flag cleared
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a return pointer to the first byte and failure
; to find a non-blank character would be
; indistinguishable.
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
FindLastNonBlankInString:
mov dx,1 ;so far we haven't found a non-blank
; character
mov bl,' ' ;put our search character, the space
; character, in a register for speed
cld
FindLastNonBlankInStringLoop:
lodsw ;get the next 2 string bytes
and al,al ;is the first byte the terminating
; zero?
jz FindLastNonBlankInStringDone
;yes, we're done
cmp al,bl ;is the second byte a space?
jz FindLastNonBlankInStringNextChar
;yes, so check the next character
mov dx,si ;remember where the non-blank was
dec dx ;adjust back to first byte of word
FindLastNonBlankInStringNextChar:
and ah,ah ;is the second byte the terminating
; zero?
jz FindLastNonBlankInStringDone
;yes, we're done
cmp ah,bl ;is the second byte a space?
jz FindLastNonBlankInStringLoop
;yes, so check the next 2 bytes
mov dx,si ;remember where the non-blank was
jmp FindLastNonBlankInStringLoop
;check the next 2 bytes
FindLastNonBlankInStringDone:
dec dx ;point back to the last non-blank
; character, correcting for the
; 1-byte overrun of LODSW
mov si,dx ;return pointer to last non-blank
; character in SI
ret
;
Skip:
call ZTimerOn
mov si,offset TestString ;string to search
call FindLastNonBlankInString ;search for the byte
call ZTimerOff
```

## Listing 11-16

```nasm
;
; *** Listing 11-16 ***
;
; Finds the last non-blank character in a string, using
; REPNZ SCASB to find the end of the string and then using
; REPZ SCASW from the end of the string to find the last
; non-blank character.
;
jmp Skip
;
TestString label byte
db 'This is a test string with blanks....'
db ' ',0
;
; Finds the last non-blank character in the specified
; zero-terminated string.
;
; Input:
; DS:SI = zero-terminated string to search
;
; Output:
; SI = pointer to last non-blank character in string,
; or 0 if there are no non-blank characters in
; the string
;
; Registers altered: AX, CX, SI, DI, ES
;
; Direction flag cleared
;
; Note: Do not pass a string that starts at offset 0 (SI=0),
; since a return pointer to the first byte and failure
; to find a non-blank character would be
; indistinguishable.
;
; Note: If there is no terminating zero in the first 64K-1
; bytes of the string, it is assumed without checking
; that byte #64K-1 (the 1 byte in the segment that
; wasn't checked) is the terminating zero.
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
FindLastNonBlankInString:
push ds
pop es
mov di,si ;SCAS uses ES:DI
sub al,al ;first we'll search for the
; terminating zero
mov cx,0ffffh ;we'll search the longest possible
; string
cld
repnz scasb ;find the terminating zero
dec di ;point back to the zero
cmp [di],al ;make sure this is a zero.
; (Remember, ES=DS)
jnz FindLastNonBlankInStringSearchBack
; not a zero. The string must be
; exactly 64K bytes long, so we've
; come up 1 byte short of the zero
; that we're assuming is at byte
; 64K-1. That means we're already
; pointing to the byte before the
; zero
dec di ;point to the byte before the zero
inc cx ;don't count the terminating zero
; as one of the characters we've
; searched through (and have to
; search back through)
FindLastNonBlankInStringSearchBack:
std ;we'll search backward
not cx ;length of string, not including
; the terminating zero
mov ax,2020h ;now we're looking for a space
shr cx,1 ;divide by 2 to get a word count
jnc FindLastNonBlankInStringWord
scasb ;see if the odd byte is the last
; non-blank character
jnz FindLastNonBlankInStringFound
;it is, so we're done
FindLastNonBlankInStringWord:
jcxz FindLastNonBlankInStringNoMatch
;if there's nothing left to check,
; there are no non-blank characters
dec di ;point back to the start of the
; next word, not byte
repz scasw ;find the first non-blank character
jz FindLastNonBlankInStringNoMatch
;there is no non-blank character in
; this string
inc di ;undo 1 byte of SCASW's overrun, so
; this looks like SCASB's overrun
cmp [di+2],al ;which of the 2 bytes we just
; checked was the last non-blank
; character?
jz FindLastNonBlankInStringFound
inc di ;the byte at the higher address was
; the last non-blank character, so
; adjust by 1 byte
FindLastNonBlankInStringFound:
inc di ;point to the non-blank character
; we just found, correcting for
; overrun of SCASB running from high
; addresses to low
mov si,di ;return pointer to the last
; non-blank in SI
cld
ret
FindLastNonBlankInStringNoMatch:
sub si,si ;return that we didn't find a
; non-blank character
cld
ret
;
Skip:
call ZTimerOn
mov si,offset TestString ;string to search
call FindLastNonBlankInString ;search for the
; last non-blank
; character
call ZTimerOff
```

## Listing 11-17

```nasm
;
; *** Listing 11-17 ***
;
; Demonstrates the calculation of the offset of the word
; matching a keystroke in a look-up table when SCASW is
; used, where the 2-byte overrun of SCASW must be
; compensated for. The offset in the look-up table is used
; to look up the corresponding address in a second table;
; that address is then jumped to in order to handle the
; keystroke.
;
; This is a standalone program, not to be used with PZTIME
; but rather assembled, linked, and run by itself.
;
stack segment para stack 'STACK'
db 512 dup (?)
stack ends
;
code segment para public 'CODE'
assume cs:code, ds:nothing
;
; Main loop, which simply calls VectorOnKey until one of the
; key handlers ends the program.
;
start proc near
call VectorOnKey
jmp start
start endp
;
; Gets the next 16-bit key code from the BIOS, looks it up
; in KeyLookUpTable, and jumps to the corresponding routine
; according to KeyJumpTable. When the jumped-to routine
; returns, is will return to the code that called
; VectorOnKey. Ignores the key if the key code is not in the
; look-up table.
;
; Input: none
;
; Output: none
;
; Registers altered: AX, CX, DI, ES
;
; Direction flag cleared
;
; Table of 16-bit key codes this routine handles.
;
KeyLookUpTable label word
dw 0011bh ;Esc to exit
dw 01c0dh ;Enter to beep
;*** Additional key codes go here ***
KEY_LOOK_UP_TABLE_LENGTH_IN_WORDS equ (($-KeyLookUpTable)/2)
;
; Table of addresses to jump to when corresponding key codes
; in KeyLookUpTable are found.
;
KeyJumpTable label word
dw EscHandler
dw EnterHandler
;*** Additional addresses go here ***
;
VectorOnKey proc near
WaitKeyLoop:
mov ah,1 ;BIOS key status function
int 16h ;invoke BIOS to see if
; a key is pending
jz WaitKeyLoop ;wait until key comes along
sub ah,ah ;BIOS get key function
int 16h ;invoke BIOS to get the key
push cs
pop es
mov di,offset KeyLookUpTable
;point ES:DI to the table of keys
; we handle, which is in the same
; segment as this code
mov cx,KEY_LOOK_UP_TABLE_LENGTH_IN_WORDS
;# of words to scan
cld
repnz scasw ;look up the key
jnz WaitKeyLoop ;it's not in the table, so
; ignore it
jmp cs:[KeyJumpTable+di-2-offset KeyLookUpTable]
;jump to the routine for this key
; Note that:
; DI-2-offset KeyLookUpTable
; is the offset in KeyLookUpTable of
; the key we found, with the -2
; needed to compensate for the
; 2-byte (1-word) overrun of SCASW
VectorOnKey endp
;
; Code to handle Esc (ends the program).
;
EscHandler proc near
mov ah,4ch ;DOS terminate program function
int 21h ;exit program
EscHandler endp
;
; Code to handle Enter (beeps the speaker).
;
EnterHandler proc near
mov ax,0e07h ;AH=0E is BIOS print character
; function, AL=7 is bell (beep)
; character
int 10h ;tell BIOS to beep the speaker
ret
EnterHandler endp
;
code ends
end start
```

## Listing 11-18

```nasm
;
; *** Listing 11-18 ***
;
; Demonstrates the calculation of the element number in a
; look-up table of a byte matching the ASCII value of a
; keystroke when SCASB is used, where the 1-count
; overrun of SCASB must be compensated for. The element
; number in the look-up table is used to look up the
; corresponding address in a second table; that address is
; then jumped to in order to handle the keystroke.
;
; This is a standalone program, not to be used with PZTIME
; but rather assembled, linked, and run by itself.
;
stack segment para stack 'STACK'
db 512 dup (?)
stack ends
;
code segment para public 'CODE'
assume cs:code, ds:nothing
;
; Main loop, which simply calls VectorOnASCIIKey until one
; of the key handlers ends the program.
;
start proc near
call VectorOnASCIIKey
jmp start
start endp
;
; Gets the next 16-bit key code from the BIOS, looks up just
; the 8-bit ASCII portion in ASCIIKeyLookUpTable, and jumps
; to the corresponding routine according to
; ASCIIKeyJumpTable. When the jumped-to routine returns, it
; will return directly to the code that called
; VectorOnASCIIKey. Ignores the key if the key code is not
; in the look-up table.
;
; Input: none
;
; Output: none
;
; Registers altered: AX, CX, DI, ES
;
; Direction flag cleared
;
; Table of 8-bit ASCII codes this routine handles.
;
ASCIIKeyLookUpTable label word
db 02h ;Ctrl-B to beep
db 18h ;Ctrl-X to exit
;*** Additional ASCII codes go here ***
ASCII_KEY_LOOK_UP_TABLE_LENGTH equ ($-ASCIIKeyLookUpTable)
;
; Table of addresses to jump to when corresponding key codes
; in ASCIIKeyLookUpTable are found.
;
ASCIIKeyJumpTable label word
dw Beep
dw Exit
;*** Additional addresses go here ***
;
VectorOnASCIIKey proc near
WaitASCIIKeyLoop:
mov ah,1 ;BIOS key status function
int 16h ;invoke BIOS to see if
; a key is pending
jz WaitASCIIKeyLoop ;wait until key comes along
sub ah,ah ;BIOS get key function
int 16h ;invoke BIOS to get the key
push cs
pop es
mov di,offset ASCIIKeyLookUpTable
;point ES:DI to the table of keys
; we handle, which is in the same
; segment as this code
mov cx,ASCII_KEY_LOOK_UP_TABLE_LENGTH
;# of bytes to scan
cld
repnz scasb ;look up the key
jnz WaitASCIIKeyLoop ;it's not in the table, so
; ignore it
mov di,ASCII_KEY_LOOK_UP_TABLE_LENGTH-1
sub di,cx ;calculate the # of the element we
; found in ASCIIKeyLookUpTable.
; The -1 is needed to compensate for
; the 1-count overrun of SCAS
shl di,1 ;multiply by 2 in order to perform
; the look-up in word-sized
; ASCIIKeyJumpTable
jmp cs:[ASCIIKeyJumpTable+di]
;jump to the routine for this key
VectorOnASCIIKey endp
;
; Code to handle Ctrl-X (ends the program).
;
Exit proc near
mov ah,4ch ;DOS terminate program function
int 21h ;exit program
Exit endp
;
; Code to handle Ctrl-B (beeps the speaker).
;
Beep proc near
mov ax,0e07h ;AH=0E is BIOS print character
; function, AL=7 is bell (beep)
; character
int 10h ;tell BIOS to beep the speaker
ret
Beep endp
;
code ends
end start
```

## Listing 11-19

```nasm
;
; *** Listing 11-19 ***
;
; Tests whether several characters are in the set
; {A,Z,3,!} by using REPNZ SCASB.
;
jmp Skip
;
; List of characters in the set.
;
TestSet db "AZ3!"
TEST_SET_LENGTH equ ($-TestSet)
;
; Determines whether a given character is in TestSet.
;
; Input:
; AL = character to check for inclusion in TestSet
;
; Output:
; Z if character is in TestSet, NZ otherwise
;
; Registers altered: DI, ES
;
; Direction flag cleared
;
CheckTestSetInclusion:
push ds
pop es
mov di,offset TestSet
;point ES:DI to the set in which to
; check inclusion
mov cx,TEST_SET_LENGTH
;# of characters in TestSet
cld
repnz scasb ;search the set for this character
ret ;the success status is already in
; the Zero flag
;
Skip:
call ZTimerOn
mov al,'A'
call CheckTestSetInclusion ;check 'A'
mov al,'Z'
call CheckTestSetInclusion ;check 'Z'
mov al,'3'
call CheckTestSetInclusion ;check '3'
mov al,'!'
call CheckTestSetInclusion ;check '!'
mov al,' '
call CheckTestSetInclusion ;check space, so
; we get a failed
; search
call ZTimerOff
```

## Listing 11-20

```nasm
;
; *** Listing 11-20 ***
;
; Tests whether several characters are in the set
; {A,Z,3,!} by using the compare-and-jump approach.
;
jmp Skip
;
; Determines whether a given character is in the set
; {A,Z,3,!}.
;
; Input:
; AL = character to check for inclusion in the set
;
; Output:
; Z if character is in TestSet, NZ otherwise
;
; Registers altered: none
;
CheckTestSetInclusion:
cmp al,'A' ;is it 'A'?
jz CheckTestSetInclusionDone ;yes, we're done
cmp al,'Z' ;is it 'Z'?
jz CheckTestSetInclusionDone ;yes, we're done
cmp al,'3' ;is it '3'?
jz CheckTestSetInclusionDone ;yes, we're done
cmp al,'!' ;is it '!'?
CheckTestSetInclusionDone:
ret ;the success status is already in
; the Zero flag
;
Skip:
call ZTimerOn
mov al,'A'
call CheckTestSetInclusion ;check 'A'
mov al,'Z'
call CheckTestSetInclusion ;check 'Z'
mov al,'3'
call CheckTestSetInclusion ;check '3'
mov al,'!'
call CheckTestSetInclusion ;check '!'
mov al,' '
call CheckTestSetInclusion ;check space, so
; we get a failed
; search
call ZTimerOff
```

## Listing 11-21

```nasm
;
; *** Listing 11-21 ***
;
; Compares two word-sized arrays of equal length to see
; whether they differ, and if so where, using REPZ CMPSW.
;
jmp Skip
;
WordArray1 dw 100 dup (1), 0, 99 dup (2)
ARRAY_LENGTH_IN_WORDS equ (($-WordArray1)/2)
WordArray2 dw 100 dup (1), 100 dup (2)
;
; Returns pointers to the first locations at which two
; word-sized arrays of equal length differ, or zero if
; they're identical.
;
; Input:
; CX = length of the arrays (they must be of equal
; length)
; DS:SI = the first array to compare
; ES:DI = the second array to compare
;
; Output:
; DS:SI = pointer to the first differing location in
; the first array if there is a difference,
; or SI=0 if the arrays are identical
; ES:DI = pointer to the first differing location in
; the second array if there is a difference,
; or DI=0 if the arrays are identical
;
; Registers altered: SI, DI
;
; Direction flag cleared
;
; Note: Does not handle arrays that are longer than 32K
; words or cross segment boundaries.
;
FindFirstDifference:
cld
jcxz FindFirstDifferenceSame
;if there's nothing to
; check, we'll consider the
; arrays to be the same.
; (If we let REPZ CMPSW
; execute with CX=0, we
; may get a false match
; because CMPSW repeated
; zero times doesn't alter
; the flags)
repz cmpsw ;compare the arrays
jz FindFirstDifferenceSame ;they're identical
dec si ;the arrays differ, so
dec si ; point back to first
dec di ; difference in both arrays
dec di
ret
FindFirstDifferenceSame:
sub si,si ;indicate that the strings
mov di,si ; are identical
ret
;
Skip:
call ZTimerOn
mov si,offset WordArray1 ;point to the two
mov di,ds ; arrays to be
mov es,di ; compared
mov di,offset WordArray2
mov cx,ARRAY_LENGTH_IN_WORDS
;# of words to check
call FindFirstDifference ;see if they differ
call ZTimerOff
```

## Listing 11-22

```nasm
;
;
; *** Listing 11-22 ***
;
; Compares two word-sized arrays of equal length to see
; whether they differ, and if so where, using LODSW and
; SCASW.
;
jmp Skip
;
WordArray1 dw 100 dup (1), 0, 99 dup (2)
ARRAY_LENGTH_IN_WORDS equ (($-WordArray1)/2)
WordArray2 dw 100 dup (1), 100 dup (2)
;
; Returns pointers to the first locations at which two
; word-sized arrays of equal length differ, or zero if
; they're identical.
;
; Input:
; CX = length of the arrays (they must be of equal
; length)
; DS:SI = the first array to compare
; ES:DI = the second array to compare
;
; Output:
; DS:SI = pointer to the first differing location in
; the first array if there is a difference,
; or SI=0 if the arrays are identical
; ES:DI = pointer to the first differing location in
; the second array if there is a difference,
; or DI=0 if the arrays are identical
;
; Registers altered: AX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle arrays that are longer than 32K
; words or cross segment boundaries.
;
FindFirstDifference:
cld
jcxz FindFirstDifferenceSame
;if there's nothing to
; check, we'll consider the
; arrays to be the same.
; (If we let LOOP
; execute with CX=0, we'll
; get 64 K repetitions)
FindFirstDifferenceLoop:
lodsw
scasw ;compare the next two words
jnz FindFirstDifferenceFound
;the arrays differ
loop FindFirstDifferenceLoop
;the arrays are the
; same so far
FindFirstDifferenceSame:
sub si,si ;indicate that the strings
mov di,si ; are identical
ret
FindFirstDifferenceFound:
dec si ;the arrays differ, so
dec si ; point back to first
dec di ; difference in both arrays
dec di
ret
;
Skip:
call ZTimerOn
mov si,offset WordArray1 ;point to the two
mov di,ds ; arrays to be
mov es,di ; compared
mov di,offset WordArray2
mov cx,ARRAY_LENGTH_IN_WORDS
;# of words to check
call FindFirstDifference ;see if they differ
call ZTimerOff
```

## Listing 11-23

```nasm
;
; *** Listing 11-23 ***
;
; Compares two word-sized arrays of equal length to see
; whether they differ, and if so, where, using non-string
; instructions.
;
jmp Skip
;
WordArray1 dw 100 dup (1), 0, 99 dup (2)
ARRAY_LENGTH_IN_WORDS equ (($-WordArray1)/2)
WordArray2 dw 100 dup (1), 100 dup (2)
;
; Returns pointers to the first locations at which two
; word-sized arrays of equal length differ, or zero if
; they're identical.
;
; Input:
; CX = length of the arrays (they must be of equal
; length)
; DS:SI = the first array to compare
; ES:DI = the second array to compare
;
; Output:
; DS:SI = pointer to the first differing location in
; the first array if there is a difference,
; or SI=0 if the arrays are identical
; ES:DI = pointer to the first differing location in
; the second array if there is a difference,
; or DI=0 if the arrays are identical
;
; Registers altered: AX, SI, DI
;
; Note: Does not handle arrays that are longer than 32K
; words or cross segment boundaries.
;
FindFirstDifference:
jcxz FindFirstDifferenceSame
;if there's nothing to
; check, we'll consider the
; arrays to be the same
FindFirstDifferenceLoop:
mov ax,[si]
cmp es:[di],ax ;compare the next two words
jnz FindFirstDifferenceFound ;the arrays differ
inc si
inc si ;point to the next words to
inc di ; compare
inc di
loop FindFirstDifferenceLoop ;the arrays are the
; same so far
FindFirstDifferenceSame:
sub si,si ;indicate that the strings
mov di,si ; are identical
FindFirstDifferenceFound:
ret
;
Skip:
call ZTimerOn
mov si,offset WordArray1 ;point to the two
mov di,ds ; arrays to be
mov es,di ; compared
mov di,offset WordArray2
mov cx,ARRAY_LENGTH_IN_WORDS
;# of words to check
call FindFirstDifference ;see if they differ
call ZTimerOff
```

## Listing 11-24

```nasm
;
; *** Listing 11-24 ***
;
; Determines whether two zero-terminated strings differ, and
; if so where, using REP SCASB to find the terminating zero
; to determine one string length, and then using REPZ CMPSW
; to compare the strings.
;
jmp Skip
;
TestString1 label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
TestString2 label byte
db 'This is a test string that is '
db 'a'
db 'terminated with a zero byte...',0
;
; Compares two zero-terminated strings.
;
; Input:
; DS:SI = first zero-terminated string
; ES:DI = second zero-terminated string
;
; Output:
; DS:SI = pointer to first differing location in
; first string, or 0 if the byte wasn't found
; ES:DI = pointer to first differing location in
; second string, or 0 if the byte wasn't found
;
; Registers altered: AL, CX, DX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
; Note: If there is no terminating zero in the first 64K-1
; bytes of a string, the string is treated as if byte
; 64K is a zero without checking, since if it isn't
; the string isn't zero-terminated at all.
;
CompareStrings:
mov dx,di ;set aside the start of the second
; string
sub al,al ;we'll search for zero in the second
; string to see how long it is
mov cx,0ffffh ;long enough to handle any string
; up to 64K-1 bytes in length. Any
; longer string will be treated as
; if byte 64K is zero
cld
repnz scasb ;find the terminating zero
not cx ;length of string in bytes, including
; the terminating zero except in the
; case of a string that's exactly 64K
; long including the terminating zero
mov di,dx ;get back the start of the second
; string
shr cx,1 ;get count in words
jnc CompareStringsWord
;if there's no odd byte, go directly
; to comparing a word at a time
cmpsb ;compare the odd bytes of the
; strings
jnz CompareStringsDifferentByte
;we've already found a difference
CompareStringsWord:
;there's no need to guard against
; CX=0 here, since we know that if
; CX=0 here, the preceding CMPSB
; must have successfully compared
; the terminating zero bytes of the
; strings (which are the only bytes
; of the strings), and the Zero flag
; setting of 1 from CMPSB will be
; preserved by REPZ CMPSW if CX=0,
; resulting in the correct
; conclusion that the strings are
; identical
repz cmpsw ;compare the rest of the strings a
; word at a time for speed
jnz CompareStringsDifferent ;they're not the same
sub si,si ;return 0 pointers indicating that
mov di,si ; the strings are identical
ret
CompareStringsDifferent:
;the strings are different, so we
; have to figure which byte in the
; word just compared was the first
; difference
dec si ;point back to the second byte of
dec di ; the differing word in each string
dec si ;point back to the differing byte in
dec di ; each string
lodsb
scasb ;compare that first byte again
jz CompareStringsDone
;if the first bytes are the same,
; then it must have been the second
; bytes that differed. That's where
; we're pointing, so we're done
CompareStringsDifferentByte:
dec si ;the first bytes differed, so point
dec di ; back to them
CompareStringsDone:
ret
;
Skip:
call ZTimerOn
mov si,offset TestString1 ;point to one string
mov di,seg TestString2
mov es,di
mov di,offset TestString2 ;point to other string
call CompareStrings ;and compare the strings
call ZTimerOff
```

## Listing 11-25

```nasm
;
; *** Listing 11-25 ***
;
; Determines whether two zero-terminated strings differ, and
; if so where, using LODS/SCAS.
;
jmp Skip
;
TestString1 label byte
db 'This is a test string that is '
db 'z'
db 'terminated with a zero byte...',0
TestString2 label byte
db 'This is a test string that is '
db 'a'
db 'terminated with a zero byte...',0
;
; Compares two zero-terminated strings.
;
; Input:
; DS:SI = first zero-terminated string
; ES:DI = second zero-terminated string
;
; Output:
; DS:SI = pointer to first differing location in
; first string, or 0 if the byte wasn't found
; ES:DI = pointer to first differing location in
; second string, or 0 if the byte wasn't found
;
; Registers altered: AX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
CompareStrings:
cld
CompareStringsLoop:
lodsw ;get the next 2 bytes
and al,al ;is the first byte the terminating
; zero?
jz CompareStringsFinalByte
;yes, so there's only one byte left
; to check
scasw ;compare this word
jnz CompareStringsDifferent ;the strings differ
and ah,ah ;is the second byte the terminating
; zero?
jnz CompareStringsLoop ;no, continue comparing
;the strings are the same
CompareStringsSame:
sub si,si ;return 0 pointers indicating that
mov di,si ; the strings are identical
ret
CompareStringsFinalByte:
scasb ;does the terminating zero match in
; the 2 strings?
jz CompareStringsSame ;yes, the strings match
dec si ;point back to the differing byte
dec di ; in each string
ret
CompareStringsDifferent:
;the strings are different, so we
; have to figure which byte in the
; word just compared was the first
; difference
dec si
dec si ;point back to the first byte of the
dec di ; differing word in each string
dec di
lodsb
scasb ;compare that first byte again
jz CompareStringsDone
;if the first bytes are the same,
; then it must have been the second
; bytes that differed. That's where
; we're pointing, so we're done
dec si ;the first bytes differed, so point
dec di ; back to them
CompareStringsDone:
ret
;
Skip:
call ZTimerOn
mov si,offset TestString1 ;point to one string
mov di,seg TestString2
mov es,di
mov di,offset TestString2 ;point to other string
call CompareStrings ;and compare the strings
call ZTimerOff
```

## Listing 11-26

```nasm
;
; *** Listing 11-26 ***
;
; Determines whether two zero-terminated strings differ
; ignoring case-only differences, and if so where, using
; LODS.
;
jmp Skip
;
TestString1 label byte
db 'THIS IS A TEST STRING THAT IS '
db 'Z'
db 'TERMINATED WITH A ZERO BYTE...',0
TestString2 label byte
db 'This is a test string that is '
db 'a'
db 'terminated with a zero byte...',0
;
; Macro to convert the specified register to uppercase if
; it is lowercase.
;
TO_UPPER macro REGISTER
local NotLower
cmp REGISTER,ch ;below 'a'?
jb NotLower ;yes, not lowercase
cmp REGISTER,cl ;above 'z'?
ja NotLower ;yes, not lowercase
and REGISTER,bl ;lowercase-convert to uppercase
NotLower:
endm
;
; Compares two zero-terminated strings, ignoring differences
; that are only uppercase/lowercase differences.
;
; Input:
; DS:SI = first zero-terminated string
; ES:DI = second zero-terminated string
;
; Output:
; DS:SI = pointer to first case-insensitive differing
; location in first string, or 0 if the byte
; wasn't found
; ES:DI = pointer to first case-insensitive differing
; location in second string, or 0 if the byte
; wasn't found
;
; Registers altered: AX, BL, CX, DX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
CompareStringsNoCase:
cld
mov cx,'az' ;for fast register-register
; comparison in the loop
mov bl,not 20h ;for fast conversion to
; uppercase in the loop
CompareStringsLoop:
lodsw ;get the next 2 bytes
mov dx,es:[di] ; from each string
inc di ;point to the next word in the
inc di ; second string
TO_UPPER al ;convert the first byte from each
TO_UPPER dl ; string to uppercase
cmp al,dl ;do the first bytes match?
jnz CompareStringsDifferent1 ;the strings differ
and al,al ;is the first byte the terminating
; zero?
jz CompareStringsSame
;yes, we're done with a match
TO_UPPER ah ;convert the second byte from each
TO_UPPER dh ; string to uppercase
cmp ah,dh ;do the second bytes match?
jnz CompareStringsDifferent ;the strings differ
and ah,ah ;is the second byte the terminating
; zero?
jnz CompareStringsLoop
;no, do the next 2 bytes
CompareStringsSame:
sub si,si ;return 0 pointers indicating that
mov di,si ; the strings are identical
ret
CompareStringsDifferent1:
dec si ;point back to the second byte of
dec di ; the word we just compared
CompareStringsDifferent:
dec si ;point back to the first byte of the
dec di ; word we just compared
ret
;
Skip:
call ZTimerOn
mov si,offset TestString1 ;point to one string
mov di,seg TestString2
mov es,di
mov di,offset TestString2 ;point to other string
call CompareStringsNoCase ;and compare the
; strings without
; regard for case
call ZTimerOff
```

## Listing 11-27

```nasm
;
; *** Listing 11-27 ***
;
; Determines whether two zero-terminated strings differ
; ignoring case-only differences, and if so where, using
; LODS, with an XLAT-based table look-up to convert to
; uppercase.
;
jmp Skip
;
TestString1 label byte
db 'THIS IS A TEST STRING THAT IS '
db 'Z'
db 'TERMINATED WITH A ZERO BYTE...',0
TestString2 label byte
db 'This is a test string that is '
db 'a'
db 'terminated with a zero byte...',0
;
; Table of conversions between characters and their
; uppercase equivalents. (Could be just 128 bytes long if
; only 7-bit ASCII characters are used.)
;
ToUpperTable label word
CHAR=0
rept 256
if (CHAR lt 'a') or (CHAR gt 'z')
db CHAR ;not a lowercase character
else
db CHAR and not 20h
;convert in the range 'a'-'z' to
; uppercase
endif
CHAR=CHAR+1
endm
;
; Compares two zero-terminated strings, ignoring differences
; that are only uppercase/lowercase differences.
;
; Input:
; DS:SI = first zero-terminated string
; ES:DI = second zero-terminated string
;
; Output:
; DS:SI = pointer to first case-insensitive differing
; location in first string, or 0 if the byte
; wasn't found
; ES:DI = pointer to first case-insensitive differing
; location in second string, or 0 if the byte
; wasn't found
;
; Registers altered: AX, BX, DX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle strings that are longer than 64K
; bytes or cross segment boundaries.
;
CompareStringsNoCase:
cld
mov bx,offset ToUpperTable
CompareStringsLoop:
lodsw ;get the next 2 bytes
mov dx,es:[di] ; from each string
inc di ;point to the next word in the
inc di ; second string
xlat ;convert the first byte in the
; first string to uppercase
xchg dl,al ;set aside the first byte &
xlat ; convert the first byte in the
; second string to uppercase
cmp al,dl ;do the first bytes match?
jnz CompareStringsDifferent1 ;the strings differ
and al,al ;is this the terminating zero?
jz CompareStringsSame
;yes, we're done, with a match
mov al,ah
xlat ;convert the second byte from the
; first string to uppercase
xchg dh,al ;set aside the second byte &
xlat ; convert the second byte from the
; second string to uppercase
cmp al,dh ;do the second bytes match?
jnz CompareStringsDifferent ;the strings differ
and ah,ah ;is this the terminating zero?
jnz CompareStringsLoop
;no, do the next 2 bytes
CompareStringsSame:
sub si,si ;return 0 pointers indicating that
mov di,si ; the strings are identical
ret
CompareStringsDifferent1:
dec si ;point back to the second byte of
dec di ; the word we just compared
CompareStringsDifferent:
dec si ;point back to the first byte of the
dec di ; word we just compared
ret
;
Skip:
call ZTimerOn
mov si,offset TestString1 ;point to one string
mov di,seg TestString2
mov es,di
mov di,offset TestString2 ;point to other string
call CompareStringsNoCase ;and compare the
; strings without
; regard for case
call ZTimerOff
```

## Listing 11-28

```nasm
;
; *** Listing 11-28 ***
;
; Searches a text buffer for a sequence of bytes by checking
; for the sequence with CMPS starting at each byte of the
; buffer that potentially could start the sequence.
;
jmp Skip
;
; Text buffer that we'll search.
;
TextBuffer label byte
db 'This is a sample text buffer, suitable '
db 'for a searching text of any sort... '
db 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 '
db 'End of text... '
TEXT_BUFFER_LENGTH equ ($-TextBuffer)
;
; Sequence of bytes that we'll search for.
;
SearchSequence label byte
db 'text...'
SEARCH_SEQUENCE_LENGTH equ ($-SearchSequence)
;
; Searches a buffer for the first occurrence of a specified
; sequence of bytes.
;
; Input:
; CX = length of sequence of bytes to search for
; DX = length of buffer to search in
; DS:SI = start of sequence of bytes to search for
; ES:DI = start of buffer to search
;
; Output:
; ES:DI = pointer to start of first occurrence of
; desired sequence of bytes in the buffer, or
; 0:0 if the sequence wasn't found
;
; Registers altered: AX, BX, CX, DX, SI, DI, BP
;
; Direction flag cleared
;
; Note: Does not handle search sequences or text buffers
; that are longer than 64K bytes or cross segment
; boundaries.
;
; Note: Assumes non-zero length of search sequence (CX > 0),
; and search sequence shorter than 64K (CX <= 0ffffh).
;
; Note: Assumes buffer is longer than search sequence
; (DX > CX). Zero length of buffer is taken to mean
; that the buffer is 64K bytes long.
;
FindSequence:
cld
mov bp,si ;set aside the sequence start
; offset
mov ax,di ;set aside the buffer start offset
mov bx,cx ;set aside the sequence length
sub dx,cx ;difference between buffer and
; search sequence lengths
inc dx ;# of possible sequence start bytes
; to check in the buffer
FindSequenceLoop:
mov cx,bx ;sequence length
shr cx,1 ;convert to word for faster search
jnc FindSequenceWord ;do word search if no odd
; byte
cmpsb ;compare the odd byte
jnz FindSequenceNoMatch ;odd byte doesn't match,
; so we havent' found the
; search sequence here
FindSequenceWord:
jcxz FindSequenceFound
;since we're guaranteed to
; have a non-zero length,
; the sequence must be 1
; byte long and we've
; already found that it
; matched
repz cmpsw ;check the rest of the
; sequence a word at a time
; for speed
jz FindSequenceFound ;it's a match
FindSequenceNoMatch:
mov si,bp ;point to the start of the search
; sequence again
inc ax ;advance to the next buffer start
; search location
mov di,ax ;point DI to the next buffer start
; search location
dec dx ;count down the remaining bytes to
; search in the buffer
jnz FindSequenceLoop
sub di,di ;return 0 pointer indicating that
mov es,di ; the sequence was not found
ret
FindSequenceFound:
mov di,ax ;point to the buffer location at
; which the first occurrence of the
; sequence was found
ret
;
Skip:
call ZTimerOn
mov si,offset SearchSequence
;point to search sequence
mov cx,SEARCH_SEQUENCE_LENGTH
;length of search sequence
mov di,seg TextBuffer
mov es,di
mov di,offset TextBuffer
;point to buffer to search
mov dx,TEXT_BUFFER_LENGTH
;length of buffer to search
call FindSequence ;search for the sequence
call ZTimerOff
```

## Listing 11-29

```nasm
;
; *** Listing 11-29 ***
;
; Searches a text buffer for a sequence of bytes by using
; REPNZ SCASB to identify bytes in the buffer that
; potentially could start the sequence and then checking
; only starting at those qualified bytes for a match with
; the sequence by way of REPZ CMPS.
;
jmp Skip
;
; Text buffer that we'll search.
;
TextBuffer label byte
db 'This is a sample text buffer, suitable '
db 'for a searching text of any sort... '
db 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 '
db 'End of text... '
TEXT_BUFFER_LENGTH equ ($-TextBuffer)
;
; Sequence of bytes that we'll search for.
;
SearchSequence label byte
db 'text...'
SEARCH_SEQUENCE_LENGTH equ ($-SearchSequence)
;
; Searches a buffer for the first occurrence of a specified
; sequence of bytes.
;
; Input:
; CX = length of sequence of bytes to search for
; DX = length of buffer to search in
; DS:SI = start of sequence of bytes to search for
; ES:DI = start of buffer to search
;
; Output:
; ES:DI = pointer to start of first occurrence of
; desired sequence of bytes in the buffer, or
; 0:0 if the sequence wasn't found
;
; Registers altered: AL, BX, CX, DX, SI, DI, BP
;
; Direction flag cleared
;
; Note: Does not handle search sequences or text buffers
; that are longer than 64K bytes or cross segment
; boundaries.
;
; Note: Assumes non-zero length of search sequence (CX > 0),
; and search sequence shorter than 64K (CX <= 0ffffh).
;
; Note: Assumes buffer is longer than search sequence
; (DX > CX). Zero length of buffer (DX = 0) is taken
; to mean that the buffer is 64K bytes long.
;
FindSequence:
cld
lodsb ;get the first byte of the search
; sequence, which we'll leave in AL
; for faster searching
mov bp,si ;set aside the sequence start
; offset plus one
dec cx ;we don't need to compare the first
; byte of the sequence with CMPS,
; since we'll do it with SCAS
mov bx,cx ;set aside the sequence length
; minus 1
sub dx,cx ;difference between buffer and
; search sequence lengths plus 1
; (# of possible sequence start
; bytes to check in the buffer)
mov cx,dx ;put buffer search length in CX
jnz FindSequenceLoop ;start normally if the
; buffer isn't 64Kb long
dec cx ;the buffer is 64K bytes long-we
; have to check the first byte
; specially since CX = 0 means
; "do nothing" to REPNZ SCASB
scasb ;check the first byte of the buffer
jz FindSequenceCheck ;it's a match for 1 byte,
; at least-check the rest
FindSequenceLoop:
repnz scasb ;search for the first byte of the
; search sequence
jnz FindSequenceNotFound
;it's not found, so there are no
; possible matches
FindSequenceCheck:
;we've got a potential (first byte)
; match-check the rest of this
; candidate sequence
push di ;remember the address of the next
; byte to check in case it's needed
mov dx,cx ;set aside the remaining length to
; search in the buffer
mov si,bp ;point to the rest of the search
; sequence
mov cx,bx ;sequence length (minus first byte)
shr cx,1 ;convert to word for faster search
jnc FindSequenceWord ;do word search if no odd
; byte
cmpsb ;compare the odd byte
jnz FindSequenceNoMatch
;odd byte doesn't match,
; so we haven't found the
; search sequence here
FindSequenceWord:
jcxz FindSequenceFound
;since we're guaranteed to have
; a non-zero length, the
; sequence must be 1 byte long
; and we've already found that
; it matched
repz cmpsw ;check the rest of the sequence a
; word at a time for speed
jz FindSequenceFound ;it's a match
FindSequenceNoMatch:
pop di ;get back the pointer to the next
; byte to check
mov cx,dx ;get back the remaining length to
; search in the buffer
and cx,cx ;see if there's anything left to
; check
jnz FindSequenceLoop ;yes-check next byte
FindSequenceNotFound:
sub di,di ;return 0 pointer indicating that
mov es,di ; the sequence was not found
ret
FindSequenceFound:
pop di ;point to the buffer location at
dec di ; which the first occurrence of the
; sequence was found (remember that
; earlier we pushed the address of
; the byte after the potential
; sequence start)
ret
;
Skip:
call ZTimerOn
mov si,offset SearchSequence
;point to search sequence
mov cx,SEARCH_SEQUENCE_LENGTH
;length of search sequence
mov di,seg TextBuffer
mov es,di
mov di,offset TextBuffer
;point to buffer to search
mov dx,TEXT_BUFFER_LENGTH
;length of buffer to search
call FindSequence ;search for the sequence
call ZTimerOff
```

## Listing 11-30

```nasm
;
; *** Listing 11-30 ***
;
; Searches a text buffer for a sequence of bytes by checking
; for the sequence with non-string instructions starting at
; each byte of the buffer that potentially could start the
; sequence.
;
jmp Skip
;
; Text buffer that we'll search.
;
TextBuffer label byte
db 'This is a sample text buffer, suitable '
db 'for a searching text of any sort... '
db 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 '
db 'End of text... '
TEXT_BUFFER_LENGTH equ ($-TextBuffer)
;
; Sequence of bytes that we'll search for.
;
SearchSequence label byte
db 'text...'
SEARCH_SEQUENCE_LENGTH equ ($-SearchSequence)
;
; Searches a buffer for the first occurrence of a specified
; sequence of bytes.
;
; Input:
; CX = length of sequence of bytes to search for
; DX = length of buffer to search in
; DS:SI = start of sequence of bytes to search for
; ES:DI = start of buffer to search
;
; Output:
; ES:DI = pointer to start of first occurrence of
; desired sequence of bytes in the buffer, or
; 0:0 if the sequence wasn't found
;
; Registers altered: AX, BX, CX, DX, SI, DI, BP
;
; Note: Does not handle search sequences or text buffers
; that are longer than 64K bytes or cross segment
; boundaries.
;
; Note: Assumes non-zero length of search sequence (CX > 0),
; and search sequence shorter than 64K (CX <= 0ffffh).
;
; Note: Assumes buffer is longer than search sequence
; (DX > CX). Zero length of buffer is taken to mean
; that the buffer is 64K bytes long.
;
FindSequence:
mov bp,si ;set aside the sequence start
; offset
mov bx,cx ;set aside the sequence length
sub dx,cx ;difference between buffer and
; search sequence lengths
inc dx ;# of possible sequence start bytes
; to check in the buffer
FindSequenceLoop:
push di ;remember the address of the current
; byte in case it's needed
mov cx,bx ;sequence length
shr cx,1 ;convert to word for faster search
jnc FindSequenceWord ;do word search if no odd
; byte
mov al,[si]
cmp es:[di],al ;compare the odd byte
jnz FindSequenceNoMatch ;odd byte doesn't match,
; so we havent' found the
; search sequence here
inc si ;odd byte matches, so point
inc di ; to the next byte in the
; buffer and sequence
FindSequenceWord:
jcxz FindSequenceFound
;since we're guaranteed to
; have a non-zero length,
; the sequence must be 1
; byte long and we've
; already found that it
; matched
FindSequenceWordCompareLoop:
mov ax,[si] ;compare the remainder of
cmp es:[di],ax ; the search sequence to
jnz FindSequenceNoMatch ; this part of the
inc si ; buffer a word at a time
inc si ; for speed
inc di
inc di
loop FindSequenceWordCompareLoop
FindSequenceFound: ;it's a match
pop di ;point to the buffer location at
; which the first occurrence of the
; sequence was found (remember that
; earlier we pushed the address of
; the potential sequence start)
ret
FindSequenceNoMatch:
pop di ;get back the pointer to the current
; byte
inc di ;point to the next buffer start
; search location
mov si,bp ;point to the start of the search
; sequence again
dec dx ;count down the remaining bytes to
; search in the buffer
jnz FindSequenceLoop
sub di,di ;return 0 pointer indicating that
mov es,di ; the sequence was not found
ret
;
Skip:
call ZTimerOn
mov si,offset SearchSequence
;point to search sequence
mov cx,SEARCH_SEQUENCE_LENGTH
;length of search sequence
mov di,seg TextBuffer
mov es,di
mov di,offset TextBuffer
;point to buffer to search
mov dx,TEXT_BUFFER_LENGTH
;length of buffer to search
call FindSequence ;search for the sequence
call ZTimerOff
```

## Listing 11-31

```nasm
;
; *** Listing 11-31 ***
;
; Compares two arrays of 16-bit signed values in order to
; find the first point at which the arrays cross, using
; non-repeated CMPSW.
;
jmp Skip
;
; The two arrays that we'll compare.
;
ARRAY_LENGTH equ 200
;
Array1 label byte
TEMP=-100
rept ARRAY_LENGTH
dw TEMP
TEMP=TEMP+1
endm
;
Array2 label byte
TEMP=100
rept ARRAY_LENGTH
dw TEMP
TEMP=TEMP-1
endm
;
; Compares two buffers to find the first point at which they
; cross. Points at which the arrays become equal are
; considered to be crossing points.
;
; Input:
; CX = length of arrays in words (they must be of
; equal length)
; DS:SI = start of first array
; ES:DI = start of second array
;
; Output:
; DS:SI = pointer to crossing point in first array,
; or SI=0 if there is no crossing point
; ES:DI = pointer to crossing point in second array,
; or DI=0 if there is no crossing point
;
; Registers altered: AX, CX, SI, DI
;
; Direction flag cleared
;
; Note: Does not handle arrays that are longer than 64K
; bytes or cross segment boundaries.
;
FindCrossing:
cld
jcxz FindCrossingNotFound
;if there's nothing to compare, we
; certainly can't find a crossing
mov ax,[si] ;compare the first two points to
cmp ax,es:[di] ; make sure that the first array
; doesn't start out below the second
; array
pushf ;remember the original relationship
; of the arrays, so we can put the
; pointers back at the end (can't
; use LAHF because it doesn't save
; the Overflow flag)
jnl FindCrossingLoop ;the first array is above
; the second array
xchg si,di ;swap the array pointers so that
; SI points to the initially-
; greater array
FindCrossingLoop:
cmpsw ;compare the next element in each
; array
jng FindCrossingFound ;if SI doesn't point to a
; greater value, we've found
; the first crossing
loop FindCrossingLoop ;check the next element in
; each array
FindCrossingNotFound:
popf ;clear the flags we pushed earlier
sub si,si ;return 0 pointers to indicate that
mov di,si ; no crossing was found
ret
FindCrossingFound:
dec si
dec si ;point back to the crossing point
dec di ; in each array
dec di
popf ;get back the original relationship
; of the arrays
jnl FindCrossingDone
;SI pointed to the initially-
; greater array, so we're all set
xchg si,di ;SI pointed to the initially-
; less array, so swap SI and DI to
; undo our earlier swap
FindCrossingDone:
ret
;
Skip:
call ZTimerOn
mov si,offset Array1 ;point to first array
mov di,seg Array2
mov es,di
mov di,offset Array2 ;point to second array
mov cx,ARRAY_LENGTH ;length to compare
call FindCrossing ;find the first crossing, if
; any
call ZTimerOff
```

## Listing 11-32

```nasm
;
; *** Listing 11-32 ***
;
; Compares two arrays of 16-bit signed values in order to
; find the first point at which the arrays cross, using
; non-string instructions.
;
jmp Skip
;
; The two arrays that we'll compare.
;
ARRAY_LENGTH equ 200
;
Array1 label byte
TEMP=-100
rept ARRAY_LENGTH
dw TEMP
TEMP=TEMP+1
endm
;
Array2 label byte
TEMP=100
rept ARRAY_LENGTH
dw TEMP
TEMP=TEMP-1
endm
;
; Compares two buffers to find the first point at which they
; cross. Points at which the arrays become equal are
; considered to be crossing points.
;
; Input:
; CX = length of arrays in words (they must be of
; equal length)
; DS:SI = start of first array
; ES:DI = start of second array
;
; Output:
; DS:SI = pointer to crossing point in first array,
; or SI=0 if there is no crossing point
; ES:DI = pointer to crossing point in second array,
; or DI=0 if there is no crossing point
;
; Registers altered: BX, CX, DX, SI, DI
;
; Note: Does not handle arrays that are longer than 64K
; bytes or cross segment boundaries.
;
FindCrossing:
jcxz FindCrossingNotFound
;if there's nothing to compare, we
; certainly can't find a crossing
mov dx,2 ;amount we'll add to the pointer
; registers after each comparison,
; kept in a register for speed
mov bx,[si] ;compare the first two points to
cmp bx,es:[di] ; make sure that the first array
; doesn't start out below the second
; array
pushf ;remember the original relationship
; of the arrays, so we can put the
; pointers back at the end (can't
; use LAHF because it doesn't save
; the Overflow flag)
jnl FindCrossingLoop ;the first array is above
; the second array
xchg si,di ;swap the array pointers so that
; SI points to the initially-
; greater array
FindCrossingLoop:
mov bx,[si] ;compare the next element in
cmp bx,es:[di] ; each array
jng FindCrossingFound ;if SI doesn't point to a
; greater value, we've found
; the first crossing
add si,dx ;point to the next element
add di,dx ; in each array
loop FindCrossingLoop ;check the next element in
; each array
FindCrossingNotFound:
popf ;clear the flags we pushed earlier
sub si,si ;return 0 pointers to indicate that
mov di,si ; no crossing was found
ret
FindCrossingFound:
popf ;get back the original relationship
; of the arrays
jnl FindCrossingDone
;SI pointed to the initially-
; greater array, so we're all set
xchg si,di ;SI pointed to the initially-
; less array, so swap SI and DI to
; undo our earlier swap
FindCrossingDone:
ret
;
Skip:
call ZTimerOn
mov si,offset Array1 ;point to first array
mov di,seg Array2
mov es,di
mov di,offset Array2 ;point to second array
mov cx,ARRAY_LENGTH ;length to compare
call FindCrossing ;find the first crossing, if
; any
call ZTimerOff
```

## Listing 11-33

```nasm
;
; *** Listing 11-33 ***
;
; Illustrates animation based on exclusive-oring.
; Animates 10 images at once.
; Not a general animation implementation, but rather an
; example of the strengths and weaknesses of exclusive-or
; based animation.
;
; Make with LZTIME.BAT, since this program is too long to be
; handled by the precision Zen timer.
;
jmp Skip
;
DELAY equ 0 ;set to higher values to
; slow down for closer
; observation
REPETITIONS equ 500 ;# of times to move and
; redraw the images
DISPLAY_SEGMENT equ 0b800h ;display memory segment
; in 320x200 4-color
; graphics mode
SCREEN_WIDTH equ 80 ;# of bytes per scan line
BANK_OFFSET equ 2000h ;offset from the bank
; containing the even-
; numbered lines on the
; screen to the bank
; containing the odd-
; numbered lines
;
; Used to count down # of times images are moved.
;
RepCount dw REPETITIONS
;
; Complete info about one image that we're animating.
;
Image struc
XCoord dw ? ;image X location in pixels
XInc dw ? ;# of pixels to increment
; location by in the X
; direction on each move
YCoord dw ? ;image Y location in pixels
YInc dw ? ;# of pixels to increment
; location by in the Y
; direction on each move
Image ends
;
; List of images to animate.
;
Images label Image
Image <64,4,8,4>
Image <144,0,56,2>
Image <224,-4,104,0>
Image <64,4,152,-2>
Image <144,0,8,-4>
Image <224,-4,56,-2>
Image <64,4,104,0>
Image <144,0,152,2>
Image <224,-4,8,4>
Image <64,4,56,2>
ImagesEnd label Image
;
; Pixel pattern for the one image this program draws,
; a 32x32 3-color square.
;
TheImage label byte
rept 32
dw 0ffffh, 05555h, 0aaaah, 0ffffh
endm
IMAGE_HEIGHT equ 32 ;# of rows in the image
IMAGE_WIDTH equ 8 ;# of bytes across the image
;
; Exclusive-ors the image of a 3-color square at the
; specified screen location. Assumes images start on
; even-numbered scan lines and are an even number of
; scan lines high. Always draws images byte-aligned in
; display memory.
;
; Input:
; CX = X coordinate of upper left corner at which to
; draw image (will be adjusted to nearest
; less-than or equal-to multiple of 4 in order
; to byte-align)
; DX = Y coordinate of upper left corner at which to
; draw image
; ES = display memory segment
;
; Output: none
;
; Registers altered: AX, CX, DX, SI, DI, BP
;
XorImage:
push bx ;preserve the main loop's pointer
shr dx,1 ;divide the row # by 2 to compensate
; for the 2-bank nature of 320x200
; 4-color mode
mov ax,SCREEN_WIDTH
mul dx ;start offset of top row of image in
; display memory
shr cx,1 ;divide the X coordinate by 4
shr cx,1 ; because there are 4 pixels per
; byte
add ax,cx ;point to the offset at which the
; upper left byte of the image will
; go
mov di,ax
mov si,offset TheImage
;point to the start of the one image
; we always draw
mov bx,BANK_OFFSET-IMAGE_WIDTH
;offset from the end of an even line
; of the image in display memory to
; the start of the next odd line of
; the image
mov dx,IMAGE_HEIGHT/2
;# of even/odd numbered row pairs to
; draw in the image
mov bp,IMAGE_WIDTH/2
;# of words to draw per row of the
; image. Note that IMAGE_WIDTH must
; be an even number since we XOR
; the image a word at a time
XorRowLoop:
mov cx,bp ;# of words to draw per row of the
; image
XorColumnLoopEvenRows:
lodsw ;next word of the image pattern
xor es:[di],ax ;XOR the next word of the
; image into the screen
inc di ;point to the next word in display
inc di ; memory
loop XorColumnLoopEvenRows
add di,bx ;point to the start of the next
; (odd) row of the image, which is
; in the second bank of display
; memory
mov cx,bp ;# of words to draw per row of the
; image
XorColumnLoopOddRows:
lodsw ;next word of the image pattern
xor es:[di],ax ;XOR the next word of the
; image into the screen
inc di ;point to the next word in display
inc di ; memory
loop XorColumnLoopOddRows
sub di,BANK_OFFSET-SCREEN_WIDTH+IMAGE_WIDTH
;point to the start of the next
; (even) row of the image, which is
; in the first bank of display
; memory
dec dx ;count down the row pairs
jnz XorRowLoop
pop bx ;restore the main loop's pointer
ret
;
; Main animation program.
;
Skip:
;
; Set the mode to 320x200 4-color graphics mode.
;
mov ax,0004h ;AH=0 is mode select fn
;AL=4 selects mode 4,
; 320x200 4-color mode
int 10h ;invoke the BIOS video
; interrupt to set the mode
;
; Point ES to display memory for the rest of the program.
;
mov ax,DISPLAY_SEGMENT
mov es,ax
;
; We'll always want to count up.
;
cld
;
; Start timing.
;
call ZTimerOn
;
; Draw all the images initially.
;
mov bx,offset Images ;list of images
InitialDrawLoop:
mov cx,[bx+XCoord] ;X coordinate
mov dx,[bx+YCoord] ;Y coordinate
call XorImage ;draw this image
add bx,size Image ;point to next image
cmp bx,offset ImagesEnd
jb InitialDrawLoop ;draw next image, if
; there is one
;
; Erase, move, and redraw each image in turn REPETITIONS
; times.
;
MainMoveAndDrawLoop:
mov bx,offset Images ;list of images
ImageMoveLoop:
mov cx,[bx+XCoord] ;X coordinate
mov dx,[bx+YCoord] ;Y coordinate
call XorImage ;erase this image (it's
; already drawn at this
; location, so this XOR
; erases it)
mov cx,[bx+XCoord] ;X coordinate
cmp cx,4 ;at left edge?
ja CheckRightMargin ;no
neg [bx+XInc] ;yes, so bounce
CheckRightMargin:
cmp cx,284 ;at right edge?
jb MoveX ;no
neg [bx+XInc] ;yes, so bounce
MoveX:
add cx,[bx+XInc] ;move horizontally
mov [bx+XCoord],cx ;save the new location
mov dx,[bx+YCoord] ;Y coordinate
cmp dx,4 ;at top edge?
ja CheckBottomMargin ;no
neg [bx+YInc] ;yes, so bounce
CheckBottomMargin:
cmp dx,164 ;at bottom edge?
jb MoveY ;no
neg [bx+YInc] ;yes, so bounce
MoveY:
add dx,[bx+YInc] ;move horizontally
mov [bx+YCoord],dx ;save the new location
call XorImage ;draw the image at its
; new location
add bx,size Image ;point to the next image
cmp bx,offset ImagesEnd
jb ImageMoveLoop ;move next image, if there
; is one

if DELAY
mov cx,DELAY ;slow down as specified
loop $
endif
dec [RepCount] ;animate again?
jnz MainMoveAndDrawLoop ;yes
;
call ZTimerOff ;done timing
;
; Return to text mode.
;
mov ax,0003h ;AH=0 is mode select fn
;AL=3 selects mode 3,
; 80x25 text mode
int 10h ;invoke the BIOS video
; interrupt to set the mode
```

## Listing 11-34

```nasm
;
; *** Listing 11-34 ***
;
; Illustrates animation based on block moves.
; Animates 10 images at once.
; Not a general animation implementation, but rather an
; example of the strengths and weaknesses of block-move
; based animation.
;
; Make with LZTIME.BAT, since this program is too long to be
; handled by the precision Zen timer.
;
jmp Skip
;
DELAY equ 0 ;set to higher values to
; slow down for closer
; observation
REPETITIONS equ 500 ;# of times to move and
; redraw the images
DISPLAY_SEGMENT equ 0b800h ;display memory segment
; in 320x200 4-color
; graphics mode
SCREEN_WIDTH equ 80 ;# of bytes per scan line
BANK_OFFSET equ 2000h ;offset from the bank
; containing the even-
; numbered lines on the
; screen to the bank
; containing the odd-
; numbered lines
;
; Used to count down # of times images are moved.
;
RepCount dw REPETITIONS
;
; Complete info about one image that we're animating.
;
Image struc
XCoord dw ? ;image X location in pixels
XInc dw ? ;# of pixels to increment
; location by in the X
; direction on each move
YCoord dw ? ;image Y location in pixels
YInc dw ? ;# of pixels to increment
; location by in the Y
; direction on each move
Image ends
;
; List of images to animate.
;
Images label Image
Image <60,4,4,4>
Image <140,0,52,2>
Image <220,-4,100,0>
Image <60,4,148,-2>
Image <140,0,4,-4>
Image <220,-4,52,-2>
Image <60,4,100,0>
Image <140,0,148,2>
Image <220,-4,4,4>
Image <60,4,52,2>
ImagesEnd label Image
;
; Pixel pattern for the one image this program draws,
; a 32x32 3-color square. There's a 4-pixel-wide blank
; fringe around each image, which makes sure the image at
; the old location is erased by the drawing of the image at
; the new location.
;
TheImage label byte
rept 4
dw 5 dup (0) ;top blank fringe
endm
rept 32
db 00h ;left blank fringe
dw 0ffffh, 05555h, 0aaaah, 0ffffh
db 00h ;right blank fringe
endm
rept 4
dw 5 dup (0) ;bottom blank fringe
endm
IMAGE_HEIGHT equ 40 ;# of rows in the image
; (including blank fringe)
IMAGE_WIDTH equ 10 ;# of bytes across the image
; (including blank fringe)
;
; Block-move draws the image of a 3-color square at the
; specified screen location. Assumes images start on
; even-numbered scan lines and are an even number of
; scan lines high. Always draws images byte-aligned in
; display memory.
;
; Input:
; CX = X coordinate of upper left corner at which to
; draw image (will be adjusted to nearest
; less-than or equal-to multiple of 4 in order
; to byte-align)
; DX = Y coordinate of upper left corner at which to
; draw image
; ES = display memory segment
;
; Output: none
;
; Registers altered: AX, CX, DX, SI, DI, BP
;
BlockDrawImage:
push bx ;preserve the main loop's pointer
shr dx,1 ;divide the row # by 2 to compensate
; for the 2-bank nature of 320x200
; 4-color mode
mov ax,SCREEN_WIDTH
mul dx ;start offset of top row of image in
; display memory
shr cx,1 ;divide the X coordinate by 4
shr cx,1 ; because there are 4 pixels per
; byte
add ax,cx ;point to the offset at which the
; upper left byte of the image will
; go
mov di,ax
mov si,offset TheImage
;point to the start of the one image
; we always draw
mov ax,BANK_OFFSET-SCREEN_WIDTH+IMAGE_WIDTH
;offset from the end of an odd line
; of the image in display memory to
; the start of the next even line of
; the image
mov bx,BANK_OFFSET-IMAGE_WIDTH
;offset from the end of an even line
; of the image in display memory to
; the start of the next odd line of
; the image
mov dx,IMAGE_HEIGHT/2
;# of even/odd numbered row pairs to
; draw in the image
mov bp,IMAGE_WIDTH/2
;# of words to draw per row of the
; image. Note that IMAGE_WIDTH must
; be an even number since we draw
; the image a word at a time
BlockDrawRowLoop:
mov cx,bp ;# of words to draw per row of the
; image
rep movsw ;draw a whole even row with this one
; repeated instruction
add di,bx ;point to the start of the next
; (odd) row of the image, which is
; in the second bank of display
; memory
mov cx,bp ;# of words to draw per row of the
; image
rep movsw ;draw a whole odd row with this one
; repeated instruction
sub di,ax
;point to the start of the next
; (even) row of the image, which is
; in the first bank of display
; memory
dec dx ;count down the row pairs
jnz BlockDrawRowLoop
pop bx ;restore the main loop's pointer
ret
;
; Main animation program.
;
Skip:
;
; Set the mode to 320x200 4-color graphics mode.
;
mov ax,0004h ;AH=0 is mode select fn
;AL=4 selects mode 4,
; 320x200 4-color mode
int 10h ;invoke the BIOS video
; interrupt to set the mode
;
; Point ES to display memory for the rest of the program.
;
mov ax,DISPLAY_SEGMENT
mov es,ax
;
; We'll always want to count up.
;
cld
;
; Start timing.
;
call ZTimerOn
;
; There's no need to draw all the images initially with
; block-move animation.
;
; Move and redraw each image in turn REPETITIONS times.
; Redrawing automatically erases the image at the old
; location, thanks to the blank fringe.
;
MainMoveAndDrawLoop:
mov bx,offset Images ;list of images
ImageMoveLoop:
mov cx,[bx+XCoord] ;X coordinate
cmp cx,0 ;at left edge?
ja CheckRightMargin ;no
neg [bx+XInc] ;yes, so bounce
CheckRightMargin:
cmp cx,280 ;at right edge?
jb MoveX ;no
neg [bx+XInc] ;yes, so bounce
MoveX:
add cx,[bx+XInc] ;move horizontally
mov [bx+XCoord],cx ;save the new location
mov dx,[bx+YCoord] ;Y coordinate
cmp dx,0 ;at top edge?
ja CheckBottomMargin ;no
neg [bx+YInc] ;yes, so bounce
CheckBottomMargin:
cmp dx,160 ;at bottom edge?
jb MoveY ;no
neg [bx+YInc] ;yes, so bounce
MoveY:
add dx,[bx+YInc] ;move horizontally
mov [bx+YCoord],dx ;save the new location
call BlockDrawImage ;draw the image at its
; new location
add bx,size Image ;point to the next image
cmp bx,offset ImagesEnd
jb ImageMoveLoop ;move next image, if there
; is one

if DELAY
mov cx,DELAY ;slow down as specified
loop $
endif
dec [RepCount] ;animate again?
jnz MainMoveAndDrawLoop ;yes
;
call ZTimerOff ;done timing
;
; Return to text mode.
;
mov ax,0003h ;AH=0 is mode select fn
;AL=3 selects mode 3,
; 80x25 text mode
int 10h ;invoke the BIOS video
; interrupt to set the mode
```