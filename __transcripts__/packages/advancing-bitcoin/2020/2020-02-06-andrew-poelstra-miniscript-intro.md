---
layout: default
parent: Advancing Bitcoin 2020
grand_parent: Advancing Bitcoin
title: 2020 02 06 Andrew Poelstra Miniscript Intro
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %} Name: Andrew Poelstra

Topic: Introduction to Miniscript

Location: Advancing Bitcoin

Date: February 6th 2020

Video: https://www.youtube.com/watch?v=eTUuwASdUBE

Slides: https://www.dropbox.com/s/vgh5vaooqqbgg1v/andrew-poelstra.pdf

## Intro

Hi everyone. Some of you were at my Bitcoin dev
[presentation](https://diyhpl.us/wiki/transcripts/london-bitcoin-devs/2020-02-04-andrew-poelstra-miniscript/)
about Miniscript a couple of days where I managed to spend over 2 hours
giving a presentation about this. I think I got this one down to 20
minutes but no promises. I will keep an eye on the clock.

## Descriptors

It is very nice having this scheduled immediately after Andrew Chow’s
[talk](https://diyhpl.us/wiki/transcripts/advancing-bitcoin/2020/2020-02-06-andrew-chow-descriptor-wallets/)
about output descriptors because there is a good transition here. As
Andrew mentioned the way that the Bitcoin Core wallet used to work and
still works is this bag of keys thing. You have a whole bunch of
elliptic curve keys and these are interpreted in whatever way happens to
be convenient at the time. This results in a messy user model and
potentially weird things like the ability for third parties to
manipulate addresses in unexpected ways. Descriptors solve this
basically by being much more precise about the way that keys are
supposed to be used and also by being more expressive. But as Andrew
hinted there are more ways that descriptors could be expressive and that
brings us into Miniscript. There are things like multisignatures,
various multi key policies and it would be nice if descriptors could
support now that we have some nice engineer readable representation of
things.

There are a pile of use cases even for ordinary users. This is not
necessarily things like Lightning or complicated protocols or anything
where you might need multiple keys. Ordinary users who just want to have
multiple redundant hardware wallets like Stepan (Snigirev) was talking
about might want to use multisignatures. People who might want to have
some backup key, some dead man’s switch where if they lose their hot
keys after a time they can still recover their coins. That is a sensible
thing to want to do. That would require a timelock or something. Then
these things might also be embedded in some larger scheme. You can
imagine somebody who wants to do this complicated multisignature for
their own purposes but they are a participant in some larger scheme.
Like they are in a split custody wallet, they have got some counterparty
who is going to countersign all of their transactions. Maybe they would
like to do something interesting and the counterparty hopefully
shouldn’t have to be decoding weird scripts to understand what the user
is doing. Or maybe they are part of a custody set or a board for a
company that holds a lot of Bitcoin like Blockstream say. They have
their own complicated multi key set up. Everybody who works at
Blockstream has their own weird complicated multi key thing. Wouldn’t it
be great if they could all bring their policies to the table and we
could just combine those and have some threshold? We say 3-of-5 people
or 8-of-10 or however many we can get. Each one of those individual
policies was something complicated. These are the kind of things that
are very difficult to do today because we don’t have a good way to
represent complicated scripts.

The way that we do represent scripts is through this thing called
Bitcoin script. I have been using this word script but what I am trying
to hint at are spending policies. But the way it is implemented is
Bitcoin script. Script can do all sorts of cool things. One interesting
example is Peter Todd has some coins out there that can be taken if you
can find a hash collision. There is one for SHA1 that has been taken.
There are others for SHA2 and RIPEMD which have not been taken. That is
a cool thing you can do with script. In practice what people typically
do is they use script to check signatures with certain keys, they check
for hash preimages for things like atomic swaps and Lightning HTLCs and
they check timelocks for things like these dead man switches or backouts
or claim a refund constructions. Then they compose these in various ways
called monotone functions. A monotone function is just ANDs and ORs and
thresholds. Putting things together into some tree of ANDs and ORs and
thresholds.

Conceptually these monotone functions of various spending conditions,
these are spending policies, they’re not really scripts, they’re not
really programs that you are running. So you can imagine that if Bitcoin
was differently designed you could directly encode a list of spending
policies. I have given an example here in a suggestively descriptor like
format.

`(pk(A),or(pk(B),or(pk(C),older(1000))))`

You could imagine having some sort of multisignature policy. We’ve got
these three keys A, B and C. We have got a 1000 block timeout and we put
them together in this way. Wouldn’t it be cool if you could put that
into a descriptor? Then the question becomes how do you map something
sensible looking like that into Bitcoin Script? That is where Miniscript
is going to come in.

## Miniscript

The idea behind Miniscript is that we can take a policy like that, we
all have these components, keys, hashes and so forth and we map these
directly into Script. We have these script templates which when executed
do the required check. You also have these script templates that will
work as ANDs or ORs so you can compose these. You can take a template
that has a couple of gaps and fill in other script components, literally
an encoding of these spending policies in Bitcoin Script. It is cool
because conceptually there are two ways to read this. You can read this
as a tree of ANDs and ORs or you can read this as a program to be
executed on the Bitcoin blockchain.

## Script: High-level problems

To summarize this situation I am describing Script works by manipulating
these opaque blobs of data. It is a very low level stack machine. You
give it a bunch of objects, some of them represent booleans, some of
them represent signatures, some represent public keys or hashes. Some
represent numbers that you can add together, you can do various
operations on them. It is really not thinking in terms of spending
policies. The difficulty with the current situation is exactly this. The
script model is this low level stack machine that doesn’t have any
obvious mapping at all to the model that I am trying to describe where
you have got a collection of different spending conditions. Miniscript
as we will see does match this user model.

Let me summarize the problems with Script as I see them. A giant list of
many problems that I see with Script real quick. When you are in this
stack based, low level opcode model it is difficult to argue that your
script is correct, that it is actually going to do what you expect. It
is difficult to argue that it is secure meaning that it won’t do what
you don’t expect. It is difficult to argue about malleability. That’s
something I will talk about in a couple of slides. If you are a wallet
implementer you run into problems with fee estimation. It is difficult
to estimate how large a witness might be when you are satisfying a given
script. It is difficult to know if you have got multiple parties
involved which public keys require signatures, how many signatures you
need and do you require hashlocks and do you require a certain timelock
to be signed and so forth? Even if you can figure out that data,
figuring out how to assemble it into a witness is non-trivial because
Bitcoin Script has a lot of different opcodes that require different
things in different orders and in different formats. If you are trying
to write software that could work with arbitrary scripts you’ve got
quite a difficult program analysis on your hands.

## Miniscript

These problems with Script directly translate into problems for
designing something like Miniscript. The idea is that I can just encode
my spending policies as scripts. But scripts have these problems in
designing these fragments, these templates. One is there are many ways
to write every fragment. If I am trying to do a public key check I can
use a CHECKSIG operator, I could use CHECKSIGVERIFY operator which is
slightly different semantics. I could use the CHECKMULTISIG operator and
throw a single public key in there or something weird like that. There
are three ways to write a pubkey check. The CHECKMULTISIG one is
obviously inefficient. There is no reason to do that but the other two
might be usable in different contexts. It is hard to composes these
fragments. The CHECKSIG operator if it succeeds will put a 1 on the
stack, if it fails it will put a 0 on the stack. CHECKSIGVERIFY if it
succeeds will put nothing on the stack and if it fails it will abort the
script. So if I am trying to write a script fragment that represents a
boolean AND or a boolean OR I need to consider that for some things I
plug in I’ll have a 1 or 0 sitting around and for other things I plug in
I won’t. Maybe I need different kinds of AND and OR fragments that can
handle these different possibilities. There is a design problem here and
there is an analysis problem of making sure that the actual composition
of fragments that a user comes up with is something sensible. Another
constraint is that I can’t just throw away every fragment except for
one. I can’t do something silly, inefficient and simple because cost is
really critical here. Every byte matters here. As Monty Python said
“Every byte is precious” on Bitcoin. Users really care about the size of
their scriptPubKeys, they care about the size of the witnesses that they
produce to spend their coins and those are actually two different
things. When they are composing things they may care about the size of
the witness if they are not satisfying something. If you have got an OR
of two different branches, maybe you satisfy one but dissatisfy the
other. There are all these different cost dimensions when considering
the size of fragments. When you are designing a Miniscript you need to
consider for every fragment how likely it is to actually be satisfied,
how likely it is to be dissatisfied and how likely it is to be skipped
completely over if it is a IF branch that doesn’t get called. This means
that there are trade-offs. You can’t just pick one fragment that is
obviously the best. Even if you know what your policy looks like there
is not one fragment that is obviously the best. You really need to
support all these different fragments and you need a way for the user to
verify that what they are doing is not only sensible and not only
efficient but actually the most efficient thing for the specific use
case that they are considering. A final point on this slide is actually
getting an optimal script might involve some funky things. If you see
the same public key occur twice then maybe you want to rearrange it so
it only occurs once. You completely restructure what your predicate
looks like. That is out of scope for Miniscript. Doing whole program
optimization things like this would be cool and in theory it is possible
but it breaks this nice, simple user model where you’ve got a tree of
spending conditions and you produce a policy that way. If you are doing
any advanced manipulation on these things then we would lose that simple
engineer readable, easy to reason about, easy to check consistency
properties.

## Miniscript - Malleability

Then let’s get into malleability. Beyond thinking about correctness
which in some sense is a straightforward engineering problem and about
optimality which is also in some sense engineering, we have this issue
of malleability. This was a really hot word before SegWit was around.
First of all what malleability is is the ability for some third party
given a Bitcoin transaction to replace the witnesses with some other
witnesses that are also valid. You get a different transaction that
still does exactly the same thing, it is still a valid transaction but
it is different in some way. Before SegWit this was really serious
because the witnesses would go into the transaction ID so if you were
chaining multiple offchain transactions this third party would break the
whole chain and you were in a lot of trouble. Post SegWit malleability
is still a thing, it is no longer a protocol breaking thing but it is an
efficiency thing in two big ways. One is that it can affect transaction
propagation properties. If somebody changes out a witness and some nodes
have seen a transaction with one witness and others see a transaction
with another witness one of those is going to get into a block and this
can interfere with compact block reconciliation. At some point you do
need the full witness and stuff and if your node is surprised by what
the witness in the blockchain turns out to be that is an inefficiency
for the network. From a user perspective if there is potential to
replace a witness with a larger witness that means that your transaction
as it hits the network is larger than the transaction you created and so
the fee rate that the network sees is lower than the fee rate you set.
Now you are surprised by your transaction having a lower priority than
you wanted it to. This is actually potentially a security issue because
it means that your transaction might not propagate when you expected it
to. Or it might not be accepted into people’s mempool when you expected
it would be.

## Miniscript - Tractability

So to make the task of designing Miniscript tractable we set a few
design rules here which more or less I have already said. The first is
that we are going to assume the standard Bitcoin Core mempool policy
rules. Standardness is the term we used to use. The reason we do this is
that there are a lot of anti-malleability things in Core’s policy rules
that are not in Bitcoin’s consensus. One prominent example of this is
that there is an IF opcode in Bitcoin. If you give it a 1 it will do the
thing, if you give it a 0 it will not do the thing. There are many, many
ways to encode 1, there are many ways to encode 0. If you have a script
that is using this OP_IF and you provide a 0 or 1 in your witness, in
principle some third party could change that 0 into a 0 that is twenty
zeros in a row. Or could change your 1 into any nonzero value or
something like that and make your transaction bigger and then you are in
trouble. You can prevent this, you can make the transaction invalid in
this case by using this trick. You use OP_SIZE followed by
OP_EQUALVERIFY. This is the one little bit of Bitcoin Script that I am
going to throw in here. SIZE EQUALVERIFY is a really cute trick. SIZE
puts the size of your top most stack element onto the stack. EQUALVERIFY
checks the two things are equal. What SIZE EQUALVERIFY does is if your
top element on the stack is equal to its own bit size that’s great. If
its not it will fail the transaction. As it turns out in Bitcoin Script
there are only two values that are equal to their own size. That is 0
which is the empty string which has size 0 and 1 which is the byte 1
that has size 1. We could put SIZE EQUALVERIFY in front of every single
IF and we could also put extra validity checks and length checks in
front of all our signature checks for example. There are a bunch of
other guards that we could put into all of our script fragments but we
don’t want to do that because we are wasting bytes there. In practice
this is not really an issue because the network is so overwhelmingly
enforcing these policy rules that really only miners could exploit this
kind of malleability. Miners already have the ability to mess with
transactions. If miners want to hurt network propagation they can just
produce a whole bunch of transactions that no-one has seen before and
stuff blocks full of these. Now compact blocks doesn’t work. Miners
already have the ability. The problems with malleability are already
things that miners can do so we are going to stick with policy rules. As
I mentioned we are not going to try any common subexpression
elimination, we are not going to try to manipulate our program, we are
not going to try to rearrange things. First of all this is very
difficult in general to find the optimal encoding of a program. Secondly
it breaks analyzability. Then also we are going to assume for the
purpose of automatic reasoning about malleability, which is something we
do in Miniscript that I will touch on but not go into too much detail,
we are going to assume that people’s keys are not reused within a
specific script. The reason for this is that if somebody uses the same
key twice then if they sign in one branch, a third party could see that
signature and copy it and use it in a different branch. This potentially
introduces a malleability vector where otherwise there wouldn’t be one.
This is very difficult to reason about in automated ways. You have
really got to know which branches are disjoint from other branches. In
general it seems like an intractable problem. We also assume no key
reuse for the purpose of fee estimation. This appears in some edge cases
when you are thinking about how to estimate witness size. We are going
to throw that away for the purpose of having something that we can
describe and we can deploy.

## Miniscript and Spending Policies

The final design thing in Miniscript that we are forced into is to
separate out this notion of the Policy language that is something that I
showed you several slides ago where you’ve got these ANDs and ORs and so
forth, from Miniscript itself. The idea is that Policy language is high
level, that is the engineer or maybe even user readable thing. It
directly describes which spending conditions are available and in what
combinations. In these policies we also allow labelling different
branches with different probabilities for certain branches being taken.
This allows us to take a policy, compile it down to a Miniscript and
make choices about which specific script fragments we want to use based
on the likelihood of certain branches being taken. We get a lot of
optimization ability here. We have this Policy language, kind of high
level. This is distinct but very similar to Miniscript itself which I am
going to show you in the next couple of slides. Miniscript really does
directly correspond to Bitcoin Script. It is an alternate encoding of a
subset of Bitcoin Script where this encoding is clearly a tree of ANDs
and ORs and thresholds. We have Miniscript itself which really is this
re-encoding of Script. The difference between Miniscript and the Policy
language is first of all there are no probabilities in Miniscript.
Secondly in Miniscript we explicitly label which fragments we are using
in all cases when there are multiple options.

Let me show you what this looks like. Here is an example of a policy.
Rather than writing it in the string descriptor format I have the
ability to draw these nice pictures. This is the cool thing about
Miniscript that you can’t do with regular scripts. Conceptually what
I’ve got is this tree of objects rather than having a bunch of
instructions for a stack machine. I get this nice format. Here is the
policy. You can see I’ve got four keys going around, I’ve got a hash
preimage, I’ve got a timelock. There is various analysis I might want to
do on this policy. Say I am the owner of public key 4 (pk_4) on the
right there and my job is to be a countersigner. I don’t really care
what else is going on here. What I want to know is can this script be
spent without my permission, without me signing off on it? Or after the
coins have been sitting still for a 1000 blocks maybe I am willing to
back off. I can easily go through this and say “Do there exist any ways
to spend this where my key does not appear?” Let’s go through all of the
spending possibilities. The top level I have this AND so both of these
need to happen. On the right hand side I have got an AND so both of
these need to happen. The only way to spend these coins, I can
immediately visually see that both of these branches are satisfied. I
can see that one of them is exactly the policy that I want. So I know
without a doubt that there is no secret way these coins can be spent out
of my control even though I didn’t look at most of the script. All I did
was run through the branches and check there existed one in every
possible spending path where my key was involved or this alternate
timeout.

Here is what a Miniscript looks like. These are almost the same picture.
You can see that the difference here is that in Miniscript I have
labeled all the ANDs and ORs and things with specifically which script
fragment I want to use. I have also labelled them with B’s and V’s,
there are K’s and W’s too. These represent types which I am not going to
go into in this talk. Miniscript has a type system that allows automatic
verification of correctness. Then I have also got these c’s, vc’s and
jv’s and stuff. What these represent are script fragments that don’t
really have any semantic meaning but they allow me to encode things in a
more efficient way. v for example means that you add an OP_VERIFY at the
end of something which turns a 1 or 0 result like what CHECKSIG does
into a pass, fail result like what CHECKSIGVERIFY does. v means put a
VERIFY on it. j is this weird wrapper where we look to see whether or
not the user has given us zero. If the user has given us zero we skip
over whatever we are wrapping. If they give us a nonzero we pass that as
the input to the fragment. The reason we do this is for a hash preimage
we want to give the user the option either to provide a valid hash
preimage or to provide the byte string zero which means skip over this,
fail the check. This j wrapper gives us exactly those semantics. You can
see that has nothing to do with the policy but it is clearly important
for actually encoding things as Script. At the bottom here I have got
the actual Bitcoin Script and opcodes. That is what it looks like.

## Miniscript and PSBT

For time reasons I am going to skip over this slide. Basically one of
Andrew’s (Chow) projects is
[PSBT](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki)
which is an unsigned transaction wallet interchange format. Miniscript
was designed to work with PSBT. They both make each other more powerful.
There are some very generic multi wallet tooling that we can write using
the one-two punch of Miniscript and PSBT. That is all I am going to say
here.

## Workshop Tomorrow

Let me quickly advertise my workshop and then I will wrap up. Tomorrow
morning I am doing a
[workshop](https://diyhpl.us/wiki/transcripts/advancing-bitcoin/2020/2020-02-07-andrew-poelstra-miniscript/)
9:30 to 12:30. Unlike the other workshops here this is not going to be
super programming focused. The reason being that my Miniscript library
is written in the Rust programming language and I don’t think it is
reasonable to expect people to either already know Rust or to learn it
in the next couple of days. It has a very high learning curve famously.
There is a Python implementation that James Chiang wrote but I don’t
think it is quite finished. It is a
[PR](https://github.com/bitcoin/bitcoin/pull/17975) to Bitcoin Core as
part of the Python test framework. It is not quite finished and I
haven’t looked over it and I don’t really know how to use it. That would
be a good thing to use for a workshop but we don’t have access to it.
Instead I am going to do something a little bit more theoretical. We are
going to go through a lot of the weird oddities in Bitcoin Script. Maybe
we will look at
[interpreter.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/script/interpreter.cpp)
which is one of the weirdest and oldest files in Bitcoin Core and see
some of the weird things the script interpreter does that might confound
various kinds of analysis. We will look in more depth at things like
correctness and malleability and surprising foot guns that you encounter
when working with Script. Even when working with Miniscript there are
surprising things that sometimes the Miniscript type system can catch
for you but nonetheless they are often non-obvious and subtle things.
Over the course of development I often would try to do weird things with
Miniscript and my code would refuse to sign for transactions, it would
reject it somehow. I think the code is broken but when I trace through
the way it was failing, it was showing me that there was some
malleability vector in what I was trying to do. There was an actual
attack that I had not considered. It is really cool we can do this in an
automated way because this is really subtle stuff and really foot gunny
and non-obvious.

## Conclusions

To wrap up, there is this thing called Miniscript. It makes spending
policies as encoded in Bitcoin Script. Human readable, let’s say
engineer readable, I like Andrew’s term. And machine analyzable. We can
do all sorts of cool things, in particular things like constructing
witnesses, determining what keys are required, estimating witness size
even under certain constraints. These become tractable and in fact we
can do this in a very generic way where somebody like me could write
some code that does this and then everyone could just use it. We are no
longer reinventing the wheel for every single individual weird policy.
You also get this composability thing that I mentioned way back at the
beginning of the talk. If you have got some larger Miniscript as we saw
with the picture I can take some little fragment that I care about for
my purposes and not really care about what the other participants are
doing as long as the overall script is using my policy in the way that I
want it to. We get this composability where I can participate in
custodying the coins that Blockstream controls say, and I show up to the
table with some crazy multisignature timelocked whatever thing and all
everyone else needs to do is that is me. Somehow that big blob is me.
They know that I haven’t somehow backdoored it by doing this complicated
thing. All I need to know is that I am contributing. You can’t move the
coins without me or some large number of people. We will go into this in
a fair bit more detail tomorrow morning. Thank you all for listening.

## Q&A

Q - You were saying there was a problem with public key reuse. Have you
thought about using the very non-famous
[OP_CODESEPARATOR](https://bitcoin.stackexchange.com/questions/34013/what-is-op-codeseparator-used-for)
operator which allows you to really use only one public key for one
person but sign a specific branch of your script?

A - This is a really cool question. The question is about using
OP_CODESEPARATOR which is this obscure opcode that lets you basically
tag what you are signing in a technical way so that a signature for one
part of your script can’t be reused for a signature in another part of
your script. First off this is a little bit inefficient. We are adding
extra bytes by sticking these OP_CODESEPARATORs in places. Some users
maybe don’t care about this. Secondly this actually doesn’t cover all
the cases that we might care about for technical reasons but that I can
maybe go in tomorrow. You can imagine a threshold policy where you have
3-of-5 participants say. There are 5 choose 3 different ways you can
spend this. You can imagine one person signs using the first three
signers. Then somebody changes it to use the last three. Now the middle
signer is in both cases. But OP_CODESEPARATOR because it only appears
once on that key won’t be able to detect this kind of change in witness.
So in that example this probably doesn’t matter because this is not an
example of key reuse. But you can imagine a threshold policy that is
more complicated. You can imagine a threshold of thresholds, something
more elaborate, where this does actually matter. I have a few examples
of these just not in my head right now.

Q - When you talked about j you said that perhaps the user wants the
script to fail. Why would they want that, just not send the transaction?

A - You never want the whole script to fail but you may want some
specific subsets of your script to fail. If you have an OR, one way that
you can do an OR in Bitcoin is you can run a script fragment, you can
store the result, it will be 0 or 1, throw it on the altstack say. Then
you run another fragment and you store that 0 or 1. Then you pull the
two results out and you run the OP_BOOLOR operation on that. The
expectation is that whichever branch the user takes they are going to
fail one of the two things and succeed the other one. This may actually
be more efficient than doing it other ways using like OP_VERIFY or
something like this. In that case you have a subset of your script that
you do want to be able to fail. You want it to put a 0 on the stack that
you then run through OP_BOOLOR to get a 1 and pass the larger script.
That is why. Whenever you have conditionals is why you might want to
fail stuff.

Q - So Andrew Chow said in the previous talk that output script
descriptors describe everything you need to solve them. Does that mean
there is redundancy between descriptors and Miniscript currently as they
are designed?

A - No there is not redundancy. What output descriptors describe is
where do you put the witnesses? If you have a PKH or something you just
throw a signature into your scriptSig. If you are using SegWIt you put
your witnesses in the witness field. If you are not using SegWit you put
it in the scriptSig. That kind of stuff. What Miniscript describes is
how you construct those witnesses in the first place. They complement
each other. Output descriptors as implemented or as on the immediate
roadmap for Core actually don’t support all of these advanced things
that Miniscript does. They only support I think basic multisig and then
all the different ways you can use single signer keys. What Miniscript
gets you is the ability to do way more stuff with output descriptors.
But the two parts that they do are disjoint, they are complementary.