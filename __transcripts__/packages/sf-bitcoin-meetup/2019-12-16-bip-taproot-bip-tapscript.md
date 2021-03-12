---
layout: default
parent: Sf Bitcoin Meetup
title: 2019 12 16 Bip Taproot Bip Tapscript
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %} 2019-12-16

bip-taproot

Pieter Wuille (sipa)

slides: <https://prezi.com/view/AlXd19INd3isgt3SvW8g/>

<https://twitter.com/kanzure/status/1208438837845929987>

<https://twitter.com/SFBitcoinDevs/status/1206678306721894400>

bip-taproot:
<https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki>

bip-tapscript:
<https://github.com/sipa/bips/blob/bip-schnorr/bip-tapscript.mediawiki>

bip-schnorr:
<https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki>

Please try to find seats. Today we have a special treat. We have Pieter
here, who is well known as a contributor to Bitcoin Core, stackexchange,
the mailing list, and he has been around forever. He's the author of
multiple BIPs. Today he is going to talk about taproot and tapscript
which is basically what the whole Schnorr movement thing became. He's
probably just giving us an update and all the small details and
everything else as well. We'd like to thank our sponsors: River
Financial, Square Crypto and Digital Garage for making this possible.

## Introduction

Thank you, Mark. My name is Pieter Wuille. I do bitcoin stuff. I work at
Blockstream. Today I am going to give an update on the status of really
three BIPs that a number of us have been working on for a while, at
least 1.5 years. These BIPs are coauthored together with Anthony Towns,
Jonas Nick, Tim Ruffing and many other people.

Over the past few weeks, Bitcoin Optech has organized structured
<a href="https://github.com/ajtowns/taproot-review">taproot review
sessions</a>
(<a href="https://www.coindesk.com/an-army-of-bitcoin-devs-is-battle-testing-upgrades-to-privacy-and-scaling">news</a>)
(and
<a href="https://bitcoinops.org/workshops/#taproot-workshop">workshops</a>
and
<a href="https://diyhpl.us/wiki/transcripts/bitcoinops/schnorr-taproot-workshop-2019/notes/">workshop
transcript</a> and
<a href="https://github.com/bitcoinops/taproot-workshop">here</a>) which
has brought in attention and lots of comments from lots of people which
have been very useful.

Of course, the original idea of taproot is due to
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html">Greg
Maxwell who came up with it a year or two ago</a>. Thanks to him as
well. And all the other people have been involved in this, too.

I always make <a href="https://prezi.com/view/AlXd19INd3isgt3SvW8g/">my
slides</a> at the very last minute. These people have not seen my
slides. If there's anything wrong on them, that's on me.

## Agenda

Okay, so what will this talk be about? I wanted to talk about the actual
BIPs and the actual changes that we're proposing to make to bitcoin to
bring taproot, Schnorr signatures, and merkle trees, and a whole bunch
of other things. I am mostly not going to talk about taproot as an
abstract concept.
<a href="https://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2018-07-09-taproot-schnorr-signatures-and-sighash-noinput-oh-my/">I
previously gave a talk about taproot</a> here I think 1.5 years ago in
July 2018. So if you want to know more about the history or the
reasoning why we want this sort of thing, then please go have a look at
that talk. Here, I am going to talk about a lot of the other things that
were brought in that we realized we had to change along the way or that
we could and should. I am going to try to justify those things.

I think we're nearing the end of-- we're nearing the point where these
BIPs are getting ready to be an actual proposal for bitcoin. But still
feel free, if you have comments, then you're more than welcome to post
them on <a href="https://github.com/sipa/bips">my github repository</a>,
on the
<a href="https://lists.linuxfoundation.org/mailman/listinfo/bitcoin-dev">mailing
list</a>, on <a href="http://gnusha.org/bitmetas">IRC</a>, or here in
person to make them and I'm happy to answer any questions.

So during this talk, I am really going to go over step-by-step a whole
bunch of small and less small details that were introduced. Feel free to
raise your hand at any time if you have questions. As I said, I am not
going to talk so much about taproot as a concept, but this might mean
that the justification or rationale for things is not clear, so feel
free to ask. Okay.

## Design goals

Why do we want something like taproot? The reason is that we have
realized it is possible to improve the privacy, efficiency and
flexibility of the <a href="https://en.bitcoin.it/wiki/Script">bitcoin
script system</a>, and doing so without changing the security
assumptions.

Primarily what we're focusing on in terms of privacy is something I'm
calling "policy privacy". There are many types of information that we
leak on the network when we're making transactions. Some of them
on-chain, and some of them just during, like by revealing things on the
p2p network. But this is only addressing one of them; namely the fact
that when you create a script on-chain, you create an output that
commits to a script and you spend it, you reveal what that script is to
the network. That means that if tomorrow some wallet provider comes up
with a fancy 4-of-7 multisig checklocktimeverify script and you start
using it, then any transaction you're doing when someone on the network
sees a 4-of-7 multisig with checklocktimeverify then they can probably
guess with high certainty that you are using that particular wallet
provider.

We're trying to address policy privacy, which is about not leaking the
policy of the spendability conditions of your outputs to the network.
The ideal that we're trying to achieve here would be possible in theory
with a recursive zero-knowledge proof construction where you show
something like "I know the hash of a program and I know the inputs to
that program that will satisfy it" but without revealing what either of
those inputs or program are. There are good reasons not to do that. One
is, all the generic zero-knowledge proof constructions that we could use
are either very large, computationally expensive, rely on new security
assumptions, need a trusted setup, all kinds of stuff that we'd really
rather avoid at this stage. But things are progressing fast in that
domain and I hope that at some point we'll actually be able to do
something moon mathy that completely hides the policy.

I'd like to, when working on proposals for bitcoin, I like to focus on
things that are likely to be accepted. That's another reason to focus on
things that don't change the security assumptions. Right now, bitcoin at
the very least requires ECDSA whose security requires the
<a href="https://en.wikipedia.org/wiki/Discrete_logarithm">discrete
logarithm</a> over the
<a href="https://github.com/bitcoin-core/secp256k1">secp256k1</a> group.
It makes a whole bunch of assumptions about hash functions, both
standard and non-standard ones. We're not changing those assumptions at
all. In fact, we're reducing them. Schnorr signatures, which I'll show
that we use, are actually relying on fewer assumptions.

I think it's important to point out that it's not even-- a question you
could ask is, well, but it should be possible to say "optionally
introduce a feature that people can use that changes the security
assumptions". Probably that is what we want to do at some point
eventually, but even--- if I don't trust some new digital signature
scheme that offers some new awesome features that we may want to use,
and you do, so you use the wallet that uses it, effectively your coins
become at risk and if I'm interacting with you... Eventually the whole
ecosystem of bitcoin transactions is relying on.... say several million
coins are somehow encumbered by security assumptions that I don't trust.
Then I probably won't have faith in the currency anymore. What I'm
trying to get at is that the security assumptions of the system are not
something you can just choose and take. It must be something that really
the whole ecosystem accepts. For that reason, I'm just focusing on not
changing them at all, because that's obviously the easiest thing to
argue for. The result of this is that we end up exploring to the extent
possible all the possible things that are possible with these, and we
have discovered some pretty neat things along the way.

Over the past few years, a whole bunch of technologies and techniques
have been invented that could be used to improve the efficiency,
flexibility or privacy of bitcoin Script in some way. There's merkle
trees and MASTs, taproot,
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html">graftroot</a>,
generalized taproot (also known as groot). Then there's some ideas about
new opcodes, new sighash modes such as
<a href="https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki">SIGHASH_NOINPUT</a>,
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-March/015838.html">cross-input
aggregation</a> which is actually what started all of this... A problem
is that there's a tradeoff. The tradeoff is, on the one hand we want to
have-- it turns out that combining more things actually gives you better
efficiency and privacy especially when we're talking about policy
privacy, say there's a dozen possible things that interact and every two
months there's a new soft-fork that introduces some new feature....
clearly, you're going to be revealing more to the network because you're
using script version 7 or something, and it added this new feature, and
you must have had a reason to migrate to script version 7. This makes
for an automatic incentive to combine things together. Also, the fact
that probably not-- people will not want to go through an upgrade
changing their wallet logic every couple of months. When you introduce a
change like this, you want to make it large enough that people are
effectively incentivized to adopt it. On the other hand, putting
everything all at once together becomes really complex, becomes hard to
explain, and is just from an engineering perspective and a political
perspective too, a really hard thing. "Here's 300 pages of specification
of a new thing, take it or leave it" is really not how you want to do
things.

The balance we end up with is combining some things by focusing on just
one thing, and its dependencies, bugfixes and extensions to it, but
let's not do everything at once. In particular, we avoid things that can
be done independently. If we can argue that doing some feature as a new
soft-fork independently is just as good as doing it at the same time,
then we avoid it. As you'll see, there's a whole bunch of extension
mechanisms that we prefer over adding features themselves. In
particular, there will be a way to easily add new sighash modes, and as
a result we don't have to worry about having those integrated into the
proposal right away. Also new opcodes; we're not really adding new
opcodes because there's an extension mechanism that will easily let us
do that later.

So the focus is on merkle trees, merkleized abstract syntax trees
(MASTs), and taproot, and the things that are required to make those
efficient. We'll look at what are the benefits that those can give us,
and makre sure they're usable. In particular, this is a focus on things
that are usable without interactive setup. So both merkle trees and
taproot have the advantage that I can just get your public key and
compute an address and have people send to it and none of you need to
run to your safe, hardware wallets or vaults or something else.

Another thing that is possible is graftroot, which can in some cases
offer much better savings and complexity than taproot, but it has the
disadvantage that you inherently need access to the private keys. That's
not a reason not to do that, but it's a reason to avoid it in the first
steps.

So let's actually stop talking about abstract stuff and move on.

## Taproot

What is taproot? Trying to make all output scripts and most spends
indistinguishable. How are we going to do that? Instead of having
separate concepts for pay-to-pubkey and pay-to-scripthash, we combine
them into one and make every output both. Every output will be spendable
by one key and zero or more scripts. We're going to make it in such a
way that spending with just a public key will be super efficient: it
will only require a single signature on-chain. The downside is that
spending with scripts will be slightly less efficient. It's a very
interesting tradeoff you can make where you can actually choose to make
one branch more efficient than the others in the script.

What is the justification for doing so? With Schnorr signatures, one key
can be easily an aggregate of multiple keys. With multiple keys, it's
easy to say that in most spends of fairly complex scripts on-chain it
could be replaced with a script branch of "everyone agrees". Clearly if
I take all the public keys involved in a script and they all agree, then
that should be sufficient to spend, regardles of what the scripts
actually are. It doesn't even need to be all of them anyway, say it's a
2-of-3 where 2 of the keys are online and one is offline... you make the
efficient side the two keys that are online, in 99% of the cases you'll
be able to use that branch.

How are we constructing an output? On the slide, you can see s1, s2, and
s3. Those correspond to three possible scripts that we want to spend
things with. Then we put them into a merkle tree. This is a very simple
merkle tree of three elements where we compute m2 as the inner node over
s2 and s3. Then m1 is the merkle root that combines it with s1. But
then, as a layer on top of that, instead of using that merkle root
directly in an output, we're going to instead use it to tweak the public
key p. So p corresponds to the public key that might be an aggregate of
multiple keys, but it's our happy path. We really hope that pretty much
all the time we'll be able to spend with p alone. Our output becomes q
which is this formula that takes p and combines it with the merkle root
and multiplies it by the generator and adds it. The end result is a new
public key that has been tweaked with the merkle root.

So we're going to introduce a new witness version. Segwit as defined in
bip141 offered the possibility of having multiple script versions. So
we're going to use that instead of using v0 as we've used so far, we
define a new one which is script v1. Its program is not a hash, it is in
fact the x-coordinate of that point q. There's some interesting
observations here. One, we just store the x-coordinate and not the
y-coordinate. A common intuition that people have is that by dropping
the y-coordinate we're actually reducing the key space in half. So
people think well maybe this is 1/2 bits reduction in security. It's
easy to prove that this is in fact no reduction in security at all. The
intuition is that, if you have an algorithm to break a public key given
just an x-coordinate you would in fact always use it. You would even use
it on public keys that also had a y-coordinate. So it is true that
there's some structure in public keys, and we're exploiting that by just
storing the x-coordinate, but that structure is always there and it can
always be exploited. It's easy to actually prove this. Jonas Nick wrote
a blog post about that not so long ago, which is an interesting read and
gives a glimpse of the proofs that are used in this sort of thing.

As I said, we're defining witness v1. Other witness versions remain
unencumbered, obviously, because we don't want to say anything yet about
v2 and beyond. But also, we want to keep other lengths unencumbered... I
believe this was a mistake we made in witness v0, which is only valid
with 20 or 32 bytes hash ... a 20 byte corresponds to a public key hash,
and the 32 bytes refers to scripthash. The space of witness versions and
their programs is limited. It's sad that we've removed the possibility
to use v0. There's only 16 versions. To avoid that, we leave other
lengths unencumbered but the downside is that this exposes us to -- a
couple of months ago, a
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-December/017521.html">issue</a>
was discovered in
<a href="https://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2017-03-29-new-address-type-for-segwit-addresses/">bech32</a>
(<a href="https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki">bip173</a>)
where it is under some circumstances possible to insert characters into
an address without invalidating it. I've
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-December/017521.html">posted
on the mailing list</a> a strategy and an analysis that shows how to fix
that bech32 problem. It's unfortunate though that we're now becoming
exposed by not doing this encumberence.

As I said, the output is an x-coordinate directly. It's not a hash. This
is perhaps the most controversial part that I expect of the taproot
proposal. There is a common thing that people say. They say, oh bitcoin
is quantum resistant because it hashes public keys. I think that
statement is nonsense. There's several reasons why it's nonsense. First,
it makes assumptions about how fast quantum computers are. Clearly when
spending an output, you're revealing the public key. If within that time
it can be attacked, then it can be attacked. Plus, there are several
million bitcoin right now available on outputs that are actually have
known public keys and can be spent with known public keys. There's no
real reason to assume that number will go down. The reason for that is
that really, any interesting use of the bitcoin protocol involves
revealing public keys. If you're using lightning, you're revealing
public keys. If you're using multisig, you're revealing public keys to
your cosigners. If you're using various kinds of lite clients, they are
sending public keys to their servers. It's just an unreasonable
assumption that.... simply said, we cannot treat public keys as secret.

In this proposal, we exploit that by just putting the point directly in
the outputs and this just saves us 32 bytes because now we don't need to
reveal both the hash and the outputs((??)). Yes, to save the bytes.
Yeah.

Q: What is the reason for not putting the parity of the y-coordinate in
there?

A: It's just to save the bytes. A better justification is that it
literally adds no security, so why should we waste a byte on it? Also
note that because we're not hashing things, this byte would go in the
output directly where it is not witness discounted.

Also, a relatively recent change to bip-taproot and bip-tapscript is no
<a href="https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki">P2SH</a>
support. The reasons for removing P2SH support are that P2SH only has 80
bits of collision resistance security. If you're jointly constructing a
script with someone else, 80 bits security is not something that we
expect in the long-term to hold up. Another reason is that P2SH is an
inefficient use of the chain... It also reduces our ability of achieving
the goal of making all outputs look identical, because if we have both
P2SH outputs and non-P2SH outputs then that just gratuitously gives up
bits of information or privacy. At this point, I think native segwit
outputs and bech32 have been adopted sufficiently that we expect that by
the time that taproot if and when it gets rolled out, the stragglers at
that point probably either won't upgrade at all.

## More BIP details

I told you that in taproot an output is a public key that we're tweaking
with the merkle root of a tree whose leaves are scripts. Here is the big
trick that makes taproot work. x here is the private key to p. Let's say
p is a single participant. It works for multiples too but it's easier to
show this way. x is the private key to p. Q which is P+H(P && merkle
root)\*G is actually equal to x + that hash times G. In other words, the
private key to Q equals x plus that hash. In other words, if you have
the private key to p and you know the merkle root, then you know the
private key to Q. In the proposal, it says it is possible to spend any
taproot output by just giving a signature with that private key to Q.
Yes, we just pick a fixed parity. At signing time, if the parity is
wrong, you flip your private key just before signing. So literally the
only thing that goes on-chain is a single signature, in the happy case.
Using schnorr signatures, that P can actually be an aggregate and
cooperative spends can just use that single key.

Another advantage of Schnorr signatures is that they are
<a href="https://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2017-09-06-signature-aggregation/">batch
verifiable</a>. If we have hundreds or hundreds of thousands of
signatures, then we can verify them more efficiently than a single one.
This scales approximately with n/log(n). If you have thousands of keys,
then you get a factor of 3x easily which is nice for initial block
download where in theory you could batch verify over multiple blocks at
once.

I lied though-- I told you that the leaves of this merkle tree are
scripts. They are actually not scripts. They are tuples of a version and
a script, so (version, script). The reason for this is simply, we have 5
bits in the header available. We couldn't get rid of the bytes, there
was 5 bits available, so why not use it as an extension mechanism and
say for now why not only define semantics for one of them, and if you
have another one then for now it's treated as ANYONECANSPEND and it's
unencumbered? So in a way you have the versioning number at the segwit
level, which is an exposed version number because your transcation
output reveals what that version number is. But this introduces a new
subversion number which is per script. Every leaf can have a different
version, which is nice for privacy because now you will only reveal----
if you have 10 branches and only one branch needs a new feature
implemented in a new version, then you only reveal that you're using
that new version when you're actually revealing that particular script
branch which is really when you need it.

This is primarily useful for large re-designs of script. I don't think
there's any such thing in the near or mediate-term future, but I think
it's good to have something where we can just swap things out. To make a
clear distinction about where you would use a witness version number
versus this leaf version number, I think the witness version is
something that we want to use for changing the execution structure like
if we want to change the merkle tree or if we want to change things like
cros-input aggregation, graftroot, those changes would need to be a new
witness version and we can't put them into the leaf version number. When
we want to replace the tree structure, use a new witness version number.
If you just want to make some changes to script, you can use the leaf
version number, in fact you might want to use another mechanism which
I'll talk about later.

At this point, only one version is defined, c0. I'll get back on why
that's not just 0 and why it's c0. This version we call tapscript. That
is actually the second BIP which describes the modifications to script
as they apply to scripts under leaf version c0 in taproot.

Another advantage of having this the second layer of versioning is, say
that in a later change, graftroot gets introduced which will need a new
witness version. We might want to reuse the leaf versions because
there's no repercussions on script itself.

When talking about that merkle tree, this is something that Eric over
there observed... we don't actually care about where in the tree our
script is, we only care about the fact that it is somewhere. It's fairly
annoying to come up with a bitwise encoding that tells you go left here,
go right here, go left here, etc. So the idea is, before hashing, the
two branches will be sorted lexigraphically. Pick the lowest first, and
now you don't need to say which side you go on. You just reveal your
leaf and you reveal the hashes to combine it with, and you're done. This
is also a very weak privacy improvement, because it automatically
mungles the tree order. If you change the public keys in it, or
something, so you're not actually revealing in your policy where that
position was. It's not perfect, but it's something of a privacy
improvement.

When spending through the scriptpath, what do you need to do? You need
to reveal the script, the leaf version, inputs to that script and a
merkle branch that proves it was committed to by your root. How do we do
that? You put on the witness stack when spending an input, you give the
inputs to the script, then the script itself (so far the same as P2SH),
but a third element is added called a "control block" which contains
marker bits, then the leaf version. It stores the sign of Q and
unfortunately we do need to reveal the sign of Q otherwise the result is
not batch verifiable. There's two possible equations and you'd get a
combinatorial explosiion if you tried to verify multiples at once, so we
need one bit to convey this sign of Q, and then we need the merkle path.

What these marker bits are for, is that by setting the top two bits of
the first byte of the last stack element in the witness stack, to 1, we
have the remarkable property that we can detect taproot spends without
having access to the inputs. I don't know what it's useful for, but it
feels like there's some static analysis advantage where I can look at a
transaction and an input and say this is a taproot spend and I don't
need to go lookup the UTXO or anything to know that. The reason for this
is that any such bytes at the end would imply a P2WSH spend with a
disabled opcode... so clearly those can't exist.

For verification, we compute S from the leaf and the script as the first
hash, then compute the merkle root from that S together with the path
you provided. Verify that equation, and again this is batch verifiable,
together with the Schnorr signatures. They can all be thrown into one
batch. Then execute the script with the input stack and then fail if it
fails execution.

Another change is that in all... should I stop a moment and ask if there
are questions so far about anything? No, all good?

Murch: If anyone has a question, just raise your hand and I'll bring you
a microphone. Okay, go on.

sipa: We'll give it another five seconds.

## Tagged hashes

Another thing that we're proposing is that instead of just using single
or double sha256 directly, we suggest to use tagged hashes. The idea is
that every hash is given a tag. The goal of this is doing domain
separation. So hashes used for computing a signature hash should we
really don't want them to ever collide or be reinterpretable as a hash
and a merkle tree, or a hash used to a derive a nonce, or a hash to
tweak the public key in taproot. An easy way to do that is by tagging a
hash along with every hash you compute. How we do this is we take your
tag, which is an ASCII string, you hash it, you double that hash which
now becomes 64 bytes, and that is a prefix before you put the data that
you're hashing yourself. Now, because 64 bytes is the block size of
sha256, it means that for any given constant tag, sha256 with that tag
actually just becomes sha256 with a different initialization function.
This costs nothing, because we precompute the new initialization vector
after hashing the tag. Because nowhere today to the best of my knowledge
in bitcoin is anywhere a protocol that hashes two single sha256 hashes
at the beginning of another sha256 hash, this should never collide with
any existing usage either. The rationale here is that bitcoin
transaction merkle tree actually has had vulnerabilities in the past
from not having such a separation. It uses the same algorithm for
hashing the inner nodes and the leaves, and there's a whole bunch of
scary bugs that result from this, some that were more recent, but the
earliest was 2012 where-- you can look up the details. Anyway, these
things are scary and we're providing a simple standard to avoid this.

All the tags that we're using are for the leaves, for combining the
scripts with the version number, that's tapleaf. For the branches of the
merkle tree, it's tapbranch. For the tweaking the public key, it's
taptweak. Then there's one for sighashes and inside Schnorr signatures
we also use them. Many of these are very likely overkill, but I think
it's good practice to start doing this.

## Not a balanced merkle tree

Another observation is that this merkle tree of all the scripts that you
might want to spend a particular output with, doesn't need to be a
balanced merkle tree. We're never even necessarily materializing it.
You're just... the verifier clearly doesn't care if it's balanced,
because he can't even observe the whole tree. You just give it a path.
There's good reasons why you may not want to make it balanced. In
particular, if you have some branches that are far less likely than
others, then you can put them further away and make the more likely ones
further up. There's actually a very simple algorithm for doing this
called Huffman tree which is used in simple compression algorithms where
you make a list of all your possibilities together with their
probabilities and combine the smallest two and put them in a node
together, and you combine them and build a tree, and that's your tree
and it's in fact the optimal one. We do in the BIP today there's a limit
of 128 levels which is clearly more than what is practical for a
balanced tree. You're clearly not going to construct things with
billions of spending possibilities... it would be too hard to even
compute the address, it could easily turn into spending seconds, minutes
and more. But because of this unbalancing, we still want a limit for
spam protection so that you can't just dump kilobytes of text into your
merkle tree. This is something to consider that may make things more
efficient.

## Annex

A final thing is something we're calling the annex. The idea is that,
observe that today in bitcoin transactions you have a locktime and an
nSequence value. The locktime has always been there. The nSequence value
was useless because its original design didn't actually work. Its
semantics were later changed in bip68. To turn them into relative
locktime. The question is, what if we want more of those things? What if
we wanted a way to have an input that could only be spent in a chain
whose blockhash at a certain height is a certain value? The obvious way
would be to do something with like a nLockTime field on a transaction
but there is no such field and we can't easily add one, at least not
without running into all sorts of complications. So what we're doing is
that, the witness stack when spending a taproot output can have a final
element optionally called an annex which is just ignored. Right now
there's no semantics associated with it, except that it is signed by all
signatures. You can't add or remove it, as long as your transaction has
signatures. It's also non-standard to use it. This would let you very
elegantly say, if the first byte of the annex is this, then it's given
these certain semantics. Another use case I'll get back to this once I
talk about resource limits in script where you could use an annex to
declare an expensive opcode and pay for it upfront because we may not
want to add very expensive opcodes otherwise. I recognize that this
might be the least obvious part of the presentation; is this clear?

## Tapscript

This is the other BIP that describes the semantics when executing a
script, specifically when the leaf version is c0. This is bip-tapscript.
By now, the reason that it is c0 and not 0 is because of those marker
bits that let you detect that it is a taproot spend even without the
inputs.

We start from the script semantics as they were in
<a href="https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki">bip141
segwit</a> but with a number of changes. The most obvious change is that
well instead of CHECKSIG and CHECKSIGVERIFY these things now use Schnorr
signatures instead of ECDSA signatures. The justification is simply that
we want these to be able to be batch verifiable and therefore there's no
reason to support both. Another difference is that CHECKMULTISIG is no
longer available. The reason for that is that, so CHECKMULTISIG today
you give it a list of public keys and then a list of signatures and it
finds which signature matches which public key. This retry behavior is
not something you can batch verify. A possibility would be to just to
declare which keys belongs to which signature, but it turns out that
wasn't actually much more efficient than not having an opcode for that
at all. So instead, our proposal adds CHECKSIGADD which is the same as
CHECKSIG but it increments an accumulator with whether or not the
signature check was successful. So this lets you implement a
CHECKMULTISIG as key CHECKSIG key CHECKSIGADD key CHECKSIGADD 5 equal
and this will do the same thing. It's a few more opcode bytes, but for
simple things you'll probably want to turn it into a merkle tree where
every branch is one combination of your multisig anyway which is more
efficient.

The next thing to discuss is OP_SUCCESS. In bitcoin script today, we
have a whole bunch of NOP opcodes. These were probably added
specifically with the intent of having an upgrade mechanism so that we
can easily add new opcodes to the bitcoin script language. So far they
have been used twice, for checklocktimeverify and checksequenceverify.
The problem with redefining these NOPs is that in order to be soft-fork
compatible they can only do one of two things: abort or not do anything
at all. This is the reason why CHECKLOCKTIMEVERIFY and
CHECKSEQUENCEVERIFY opcodes today don't pop their argument from the
stack and you always need an OP_DROP after that. It's because their
redefinition of a NOP as a result they cannot modify the stack in any
way.

A solution to that is instead of having an opcode that doesn't do
anything (a NOP), have an opcode that just returns TRUE and that's
OP_SUCCESS. That might sound unsafe, but it's exactly as unsafe as
having new witness versions that are undefined. You won't use them until
you have a use for them, and you won't use them until they have defined
locked-in semantics on the network. We're taking a whole swath of
disabled and never-defined opcode numbers, and turning them into "return
TRUE". Later, these opcodes can be redefined to be anything, because
everything is soft-fork compatible with "return TRUE".

Bram: Is that "return TRUE" at parse time?

sipa: It is at parse time. There's a preprocessing step where the script
gets decoded before execution and even if you have an OP_SUCCESS in an
unexecuted IF branch for example, it still means "return TRUE". This
means it can be used to introduce completely new semantics, it can
change resource limits, it change the parse function in a way if you say
the first byte of your script becomes OP_SUCCESS and so on. So this is
just the most powerful way of doing it.

Of all the different upgrade mechanisms that are in this proposal,
OP_SUCCESS is the one I don't want to lose. The leaf versions can
effectively be subsumed by OP_SUCCESS just start your script with an
opcode like OP_UPGRADE and now your script has completely new semantics.
This is really powerful and should make it much easier to add new
opcodes that do this or that.

Another thing is upgradeable pubkey types. The idea is that if you have
a public key that is passed to CHECKSIG, CHECKSIGVERIFY or CHECKSIGADD,
that is not the usual 32 bytes (not 33 bytes anymore, it's 32 bytes
because also there we're just using the x-coordinate). If it's not 32
then we treat that public key as an unknown public key type whose
signature check will automatically succeed. This means that you can do
things like introduce a new digital signature scheme without introducing
new opcodes every time. Maybe more short-term, it means that it's also
usable to introduce new signature hashing schemes where otherwise you
would have to say oh I have slightly different sighash semantics like
SIGHASH_NOINPUT or
<a href="https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-May/016929.html">ANYPREVOUT</a>
or whatever it's called these days. Introducing them would, every time
you would need to have three new opcodes otherwise. Using upgradeable
public key types, this problem goes away.

Another is making "minimal IF" a consensus rule. "Minimal IF" is
currently a standardness rule that has been in segwit forever and I
haven't seen anyone complain about it. It says that the input to an
OP_IF or an OP_NOTIF in the scripting language has to be exactly TRUE or
FALSE and it cannot be any other number or bytearray. It's really hard
to make non-malleable scripts without it. This is actually something
that we stumbled upon when doing research on miniscript and tried to
formalize what non-malleability in bitcoin script means, and we have to
rely on "Minimal IF" and otherwise you get ridiculous scripts where you
have two or three opcodes before every IF to guarantee they're right. So
that's the justification, it's always been there, we're forced to rely
on it, we better make it a consensus rule.

The last one is new code separator semantics. The OP_CODESEPARATOR has
been in bitcoin forever. It was probably originally intended as a
delegation mechanism because it restricted what part of the script was
being signed by signatures, and you could actually implement a
delegation mechanism with this. Unfortunately, in mid 2010 when the
execution of the scriptsig and the scriptpubkey was split and the
CODESEPARATOR in one doesn't influence the signatures in the other,
anymore, then that functionality was broken. So we looked at that and
thought, well what is it useful for? There's one thing that it can still
be used for, namely where you have multiple execution branches through
your scripts like IF THEN ELSE and you want your signatures to commit to
which of the branch you're taking... by putting a CODESEPARATOR in one
of them, you change what script they are signing, and as a result
indirectly lets you commit to what branch you're taking in the script. I
don't know if anyone using this in production but I've certainly heard
of people thinking about using this. So we're going to keep this part
and drop the rest. We're just going to make signatures sign the last
executed CODESEPARATOR position without those things.

## Resource limits

Okay, resource limits. Bitcoin script today has a 10,000 byte script
size limit which we propose to drop. The reason for this is that it has
no use. There is literally no data structure left in taproot whose size
depends on the size of your script, or any way in which execution time
is more than proportional to your script size. Before, so even in
segwit, every signature hash contains the entire script being executed
which means that if you have a script of 10,000 bytes and they're all
CHECKSIGs, then you're going to do 10,000 CHECKSIGs that each hash
10,000 bytes and this is a new version of the quadratic hashing problem
that was improved before like in segwit there's fewer quadratic hashing
things left than in the legacy system that came before it, but in
taproot they're all gone. We actually just pre-hash the script once, and
then it gets included in the signature hash every time. So with that, we
don't need that anymore.

Also, the 201-non-push ops limit, which is my favorite crazy thing in
bitcoin. We got rid of that too. To show how crazy it is, right now in
bitcoin script there is a rule that says that the total number of
non-push opcodes in your script can be at most 201. This counts executed
and unexecuted opcodes. However, when you execute a CHECKMULTISIG, the
number of public keys participating in that CHECKMULTISIG counts towards
this limit, but only the executed ones. So that's another thing that-- I
discovered this while working on
<a href="https://diyhpl.us/wiki/transcripts/stanford-blockchain-conference/2019/miniscript/">miniscript</a>
where I had a fuzzer constructing random scripts and we had miniscripts
reasoning about oh is this a valid script and handing it to Bitcoin Core
to verify and some didn't pass and it was because of this CHECKMULTISIG
thing. There's really no reason for this weird limit.

Lastly, we removed the block-wide 80,000 sigop limits and replace it
with one-sigop per 50 bytes of witness rule. This accomplishes a really
cool thing that we get rid of multi-dimensional optimization. Right now,
there are two independent resource limits for blocks. There's the weight
and there's the sigops. In normal transactions like the number of sigops
is way below the ratio where this matters. But you can maliciously
construct scripts that no miner will know how to optimize for correctly.
This is very vaguely exploitable. It's just an income reduction, right.
A miner will not be able to find the optimal bin packing solution to
given all these transactions and they have fees and weights and sigops
what's the optimal combination. But the downside is that if they would
try to be smarter, then fee estimation on the network would be harder.
By removing the sigops limit, and basically translating the sigops into
bytes, which is what this does, like every 50 bytes you get a credit of
1 sigop you can do. Given that a public key plus a signature already
costs 96 bytes, there's absolutely no reason why you would ever need to
go over this limit. That way, there's only a single limit that is left
to optimize for, and things get a lot easier.

Maybe you wonder, well, why only sigops? Why not big hashes or other
expensive opcodes? So I did some benchmarking. Like what are the most
complex slowest slow scripts we could come up with in terms of execution
time per byte that don't use CHECKSIGs? It turns out there's fairly
close like within an order of magnitude, which I guess is not really
close, but it's not crazy far off of the worse you can do with just
CHECKSIG. So that's the reason for only counting CHECKSIGs here. But
imagine in the future there's an OP_SNARKVERIFY or OP_X86 or
OP_WEBASSEMBLY that costs orders of magnitude more than a CHECKSIG now
per byte, will want to price those proportionally... it doesn't matter
that it's exactly proportional, but you need to be able to reason that a
block full of these is not going to become an attack vector. So how do
you do that? The problem is, say this SNARKVERIFY costs the equivalent
of 1000 bytes worth of CHECKSIGs but your script isn't 1000 bytes and no
reasonable input to this opcode will be 1000 bytes. You'd be
incentivized to, you know, stuff your transaction with zero bytes or
something just to make it pass. That's actually another justification
for the annex where you can use an annex on the input to say, virtually
increment the cost of this transaction by this many bytes. Just treat it
as as if it was a kilobyte bigger, and I'll even pay fees for it. Now
you can see why it is important that you can statically determine an
input is a taproot spend without having the inputs available, because
otherwise I hand you a transaction on the network and you want to
compute its size, you want to compute its fee rate, and in order to
compute the fee rate you need to know its size, and its size is now
being amended by this annex. But by allowing this to be done statically,
by using a recognizable pattern for taproot spends, this is much easier
and you don't need the UTXO set or anything else available to compute
the size of transactions. Likely this never happens, I have no concrete
ideas for what this would be useful for, but it's an extension mechanism
that we considered and found interesting.

There's also some changes to the signature hashing algorithm as I
alluded to earlier, with the scriptpubkeys. The sizes don't matter
anymore. So we have the same sighash modes, which is the easiest to
argue for. I don't think the existing sighash modes like ALL, SINGLE,
NONE, and ONE, and ANYONECANPAY, I don't think they are particularly
great- most of them are never used. But it's just easier to argue to not
change the semantics, especially with upgradeable pubkeys we can easily
introduce new sighash modes later. So it's less of a concern to do this
perfectly right now. A big one is that inputs commit to all input
amounts, not just the amount of the output they are spending. The reason
for that is a particular attack that exists today on hardware wallets or
offline signing devices where say there's malicious wallet software and
there's a hardware wallet and the malicious software colludes with a
miner to spend all of someone's coins into fees. This is why we commit
to amounts in the first place in segwit, but it is not sufficient
because you actually need to commit to all of them. The reason is that,
as a malicious wallet, you can give the hardware wallet two
transactions. I give it two-- I do two signing attempts. The first time,
I lie about the amount on the first inputs, but I'm honest on the second
one. On the second attempt, I'm honest on the second one but I lie about
the first. So now I can take the signatures from these two, combine them
into one single valid transaction that moves everything into fees. This
attack has been described, it's fairly obscure to pull off. In order to
address it, we should commit to all input amounts and this problem goes
away. Committing to all input amounts and all output amounts effectively
means you're committing to the fee itself.

Another one is committing to the scriptpubkey. There is some issues
where you can lie to a hardware device like oh this is a p2sh versus a
non-p2sh spend. This would in theory make them report fee rates
incorrectly or something. Good practice and this can be completely
avoided by committing to the scriptpubkey.

All variable length data is now pre-hashed. The hashing that you do for
any checksig is now just a constant, up to 200 something bytes. This is
the reason why we can get rid of the script size limits.

In that sighash, at the transaction level, data goes first before the
input level data which allows in theory some additional caching where
you pre-compute that whole part once. Then obviously, all the new
things, you commit to the annex, the leaf hash, and the CODESEPARATOR
position.

That's all I have about changes introduced in tapscript. I have another
slide about future things, but if there's any questions about any of
these things, feel free.

murch: How has the organized taproot review been going? I haven't heard
that much about that.

Bitcoin Optech has organized a public structured taproot review session.
It was a 7-week thing where every week there was a curriculum which was
some subset of the changes to discuss, and once or twice a week there
would be a moment where people could ask questions and review things.
The first weeks went very well. There were tons of questions, very good
suggestions on improvements. The later weeks, it's been dropping off
significantly. I think this is the last week. Tomorrow I think is the
last Q&A session. Overall I'm happy, and this gave a lot of exposure
about these ideas to people who gave lots of good comments, some pull
requests. There were only a few actual semantic changes that came out of
this. Or ideas about that. Most of it has been just about some things
being unclear, rationales being unclear, and requests for stating
motivations better.

Bram: The RETURN TRUE extension semantics... that implies that you're
not doing any delegation? Or you're only delegating to a trusted thing
or a thing of known code at the time you do the call? It makes it so you
can't do a graftroot where you have a graftroot.

sipa: You can, I think.

Bram: You can't do partial delegations. So you're either completely
trusting something, or not calling it.

sipa: First of all, graftroot is not included here so that's not a
concern. Longer-term, I don't see the problem really. When delegating,
you delegate to a new script and if you delegate to a script that has an
OP_RETURN TRUE, then that's your problem- you're not going to do that.

Bram: I know you don't want to support colored coins, but if I did, then
you have this sort of partial delegation- this outer thing that is
enforcing the color, and an inner thing that might support transactions
within the colored coin itself.

sipa: I think you can still do that as long as you structure the
requirements on the caller as an assertion. I think the semantics you
want is that, when delegating you first completely execute that level of
the script. If it returns false, for any reason, you return false, and
then you do all of the invoked delegations.

Bram: The main issue.. and I don't have a good solution for this, is if
you want an extension that returns some value. Like some new function
that returns a new thing, then this extension mechanism makes it hard to
do that.

sipa: Sounds like something to discuss when this becomes an issue.

murch: Any more questions?

## What's next

We need to finish addressing all the comments from review. There's still
a few open issues. This is the last week of the Bitcoin Optech review
sessions. I hope and I think the sentiment in general is positive. I am
hopeful that we will find widespread acceptance for these changes. We'll
have to work on requesting a BIP number, I think we'll have it in the
next few days. Then there's work on reference code, unit tests, and just
review, test vectors, and need to open up a demo pull request to Bitcoin
Core with an implementation... and then some unclear stuff... and then
one day it will activate on the network. I am purposefully not going
into details on how activation might happen. I think different people
have different opinions on that aspect. That's something up to the
community to decide how these things make their way forward.

In parallel, once it's clear that these things will happen, there's work
that can be done on reference implementations for wallets and libraries.
Things like
<a href="https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki">bip174
PSBT extensions</a>, I know Elichai has been working on that. Miniscript
will probably want to extend that to support taproot. Given the whole
structure introducing with the merkle tree, and the tweaked key, just
getting an overview of what a script is or what a policy is enorced by a
particular output becomes harder so we will want better and better tools
to actually be able to deal with this. There will be fewer and fewer
ways that you can do these things manually.

That was my talk, thank you very much.

## Q&A

Q: What's the type of pushback if any you are seeing?

sipa: I think I've seen some comments on putting public keys directly in
an output... but even that has been far less than I had feared.

murch: So deploy in 2 weeks?

sipa: No, I think the most valuable feedback is critical review
including questions like "why are you doing this, is this part
necessary" or comments like "this is a bad idea". I haven't heard any
such comments. That might be due to either the right people haven't
looked at it yet, or the proposal is already polished enough that we're
past that stage. I think all kinds of comments are useful. It's very
hard to judge where acceptance lies. I can go on twitter and ask hey all
in favor... I don't think that the responses would be very useful, even
the positive ones. This is a hard question. In the end, I think we'll
know consensus when we see it.

Q: Around this merkle tree and creating more complex scripts, what is
the future looking like there with tools and wallets to actually build
those things? What's the work that needs to be done there?

sipa: Yeah, so, it's interesting that at this point we're really
focusing on defining the semantics primarily the consensus side of
things. But there's a whole lot of work to do on integration and on the
signing side. A lot of it is optional. I suspect that many wallets will
just implement the single key signing because that's all they care about
and single key signing isn't all that much harder in taproot than
something else. At the same time, there's a lot of more flexibility and
possibilities that are there.

Q: Given that the depth of the tree can be 128, you can have quite
sizable scripts. Have you seen anything novel coming out of that in
review?

sipa: There's obvious things like, you take a multisig and you split it
out into a tree where every leaf is one combination of keys and you
aggregate the keys per leaf together. So you just have a couple thousand
leafs... say maybe 5-of-10 or something. I don't know. And then you have
a couple dozen leaves that are each just a single key. It's far more
efficient and much more private than doing it directly as a script that
just gives all the public keys. Really a lot of this depends on what
kind of policies that people want.

Q: In the last major soft-fork, we activated segwit and we saw some
pushback from some stakeholders and ecosystem people that hadn't been
involved that much in design. Are you aware of any activity to reach out
to those subsets of the community like miners in particular?

sipa: No, I am not aware. But everyone is welcome to review. The
ecosystem is probably more well organized now than it was a while ago. I
don't expect it to be a problem. Of course, we don't know if miners will
go along and how long it will take. This is all up to activation
mechanism which is still an open question.

Q: What about potentially simplifying multisig applications; what's
going to be the most common or powerful use case you can get from this?
If you reduce down to a single signature on-chain.

sipa: That's a possibility, but it comes with complexity. The
verification side of things becomes simpler but actually building an
interactive protocol between multiple parties that jointly sign for a
single key is more complicated than signing a multisig today. It has
pitfalls. I wouldn't be surprised if in practice it ends up onyl being a
few players that do this. That's fine, because things are still
indistinguishable.

Q: A quick question about the annex. Is that strictly for making
transactions being able to weight their incentives properly...

sipa: That's one reason. There's another example which is the anti-fee
sniping thing. Right now there's a technique for anti-fee sniping where
you put a locktime on a transaction for a block not far in the past. If
a miner tries to intentionally reorg the chain to grab the fees of this
perhaps high-fee-paying transaction, then they wouldn't be able to do so
and they would still need to wait. If there was a way to make a
transaction that says this is only valid in a chain that builds off of
this blockhash, then this becomes a lot more powerful. I don't know if
it's sufficiently desirable, but it's an example of something that you
could do.

Q: You could even create a bounty to vote against a split or something
like that, because you commit to only one side of the split.

sipa: This is fork protection, too. I think that's the context in which
it was originally proposed.

1h 26min

## Other resources

<http://gnusha.org/bitmetas/>