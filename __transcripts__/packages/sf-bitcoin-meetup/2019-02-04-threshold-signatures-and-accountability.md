---
layout: default
parent: Sf Bitcoin Meetup
title: 2019 02 04 Threshold Signatures And Accountability
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %} Threshold signatures and
accountability

Andrew Poelstra

<https://twitter.com/kanzure/status/1092665108960968704>

Sorry for the delays. We're finally ready to talk. Welcome to our talk.
We'd like to thank our sponsors including Blockstream and Digital
Garage. Today we have Andrew Poelstra to talk about threshold signatures
and accountability. He is known for working on mimblewimble, Bitcoin
Core, libsecp256k1, and at Blockstream. Thank you.

## Introduction

Cool, thank you Mark for that introduction. Sorry about the delays.
Let's get started. I'll try to not spend 90 minutes on this talk. I'll
make an effort to not keep you here past midnight. I'm kidding.

Today I am here to talk about threshold sigantures and accountability
and a lot of the signature work I've been doing with Pieter Wuille, Tim
Ruffing and Jonas Nick and many other people over the past year.

I'll start by talking about Schnorr signatures which was the buzzword
for 2018 and probably 2019 as well. I'll also talk about
multisignatures, threshold signatures, some cool tricks around threshold
signatures. I'll do this in a cool algebraic fashion. My slides are
going to get increasingly complicated, but I'll pay decreasingly as much
attention to what's being written there. On the first slide, I'll just
explain everything. The point is the shapes and the colors which I hope
will communicate how much fun it is to work on this kind of stuff.

## Schnorr signatures

This is a Schnorr signature. It has two values (s, r) which are computed
by the shown equations here. The way it works is that we start with a
key pair where you have a secret key x (all of my secret values in red)
and a public key called P. The way to get from the secret key to the
public key is that you have some elliptic curve group generator. It's an
object that you can add to itself in some formal sense of add, many
times. So the public key P is x \* G. If I give you some curve point,
you can't tell how many time G is added to it, and it preserves the
secrecy.

To make a Schnorr signature, we make an ephemeral key pair called a
nonce. This is the value R. I think of a new secret key k that I use for
just this signature. I get this object R. I am going to hash up all this
data, my public key, my nonce, and this extra auxiliary input called a
message and in a signature the message is the most important. I throw
all of these into a sha256 hash or something. This gives me a challenge.
With this random number, I am going to produce a signature. I'm going to
produce this linear equation s = k + ex using this challenge and my two
secret keys. To verify this equation, you do the following- this is the
verification equation. The verification equation is the same except they
multiply every term in the equation by G. So it's sG = kG + exG. If you
want it to be verifiable, you stick G's in there. When I multiply
everything by G, the public node sG, kG is my public nonce, sG is my
public key, and e my public node.... everyone can verify this. I as the
knower of x and G can produce a signature, but everyone who knows my
public key can verify.

## Sign-to-contract

I am going to do a diversion from Schnorr signatures to talk about
sign-to-contract. I originally said I would create a nonce, but instead
I'm going to take another message called c here and I'm going to tweak
my public nonce and my secret nonce by a hash of my original public
nonce and this new message c. This new R equation turns this R quantity
from being an ephemeral key to also being a commitment to some auxiliary
data c.

This is useful in a few situations. For example, this is useful in
timestamping where you can turn a signature into a commitment. You have
now committed some data on the blockchain with no additional data being
used. The resulting point or the resulting signature is s and R is
exactly the same as a signature without a commitment.

So that's useful for timestamping, and also for another thing that I
will get to at the end of my talk. It's useful for a weird application
which I will try to describe here.

## Sign-to-contract replay attack

Let me say, though, that suppose you have a hardware wallet producing
these signatures. You normally give your hardware wallet a message, it
knows your secret key, it will produce secret nonces, and it computes an
ephemeral secret value k. There's a couple of important properties that
k needs to have if it is going to work as an ephemeral secret key. k
needs to be uniformly random, and if there's any little bit of bias,
then if you produce enough signatures and publish enough of them,
someone could take the small amount of bias and eventually exploit that
to solve for all the different k values and your secret keys. This has
to be uniformly random.

In a hardware wallet, you take your message and your secret key and you
hash that up to produce your random value. The hash function we assume
is random and this gets a uniformly produced value k, assuming our hash
function is secure. But we don't need a random number generator- these
are hard to implement in hardware, and they are impossible to verify.
It's not nice to require a random number generator. Also, this is
stateless. In other contexts, the hardware wallet has to keep track of
something and make sure it's incrementing a value. But this is a
stateless random number generator without requiring a hardware random
number generator.

But this interacts badly with sign-to-contract if you try to do that.
Suppose that you have your hardware wallet here, and it's producing this
value k by hashing up the secret and the message like a good hardware
wallet should be doing. And you ask it to do sign-to-contract with this
commitment c, and it produces the value k by hashing, and hten tweaks k
by adding an extra hash. And then it produces a signature, that's great.

So suppose you go to the hardware walle tand ask it to re-sign the same
message. I ask it to commit to a different value c prime. So the
hardware wallet will produce two different signatures. Because I tweaked
the commitment, I have managed to change the public nonce, and the hash
challenge is going to change. The difference between these two--- the
two questions both use k, the same secret nonce k, so the difference
between these two equations does not have k anymore. So by counting the
number of red entities, the last equation there has only one red entry--
if you shuffle everything around, you can see that you can solve for x.

If you have a hardware wallet that supports sign-to-contract, and you
ask it to sign with different commitments, then you're going to have
problems.

But there's a fix. There's always a simple fix. The simple fix in this
case is that when a hardware wallet is generating a nonce, it should
hash the message and the commitment and a new commitment. If you change
the message or the commitment, you end up with a different k. So k =
H(x||m||c)).

## Sign-to-contract as an anti-nonce-sidechannel measure

I believe this method originates from Greg Maxwell. It's a way of using
sign-to-contract to prevent sidechannels. Suppose that k was biased in
some way, and you leak your secret key. A hardware wallet could produce
random nonces, and then each ... with that nonce with some secret key
that only some attacker knows, and thereby introduce a bias that is only
known by someone with an HMAC key. So it's possible to do this attack
and it's slightly biased in only a way that only some bad guy can tell.
If you have a piece of hardware generating such signatures, there's no
way to verify that this is even happening. This is a risk right now for
every single hardware wallet that is being used right now. The random
numbers that it is generating may not be generated honestly.

In general, the way to detect this is to create some dummy signatures
with a secret key that is extracted, and then you check for k and make
sure it was generated deterministically. If you have keys that you are
really using in the blockchain, you don't want to pull those secret keys
out of the hardware wallet and doing analysis. That's not prudent.

If you trust your host computer, you can ask the host computer to
introduce some additional randomness into your nonce. So even if the
hardware wallet is trying to produce nonces with detectable bias, you
can totally clear that out. The trick is to use the sign-to-contract
equation, and even if k is biased,that hash is not biased. You have
something with questionable uniformity, and you add it to something with
unquestionable uniformity, and then you're great. Here, I need c to be
unique. It needs to be high entropy and something that someone can't
guess. So you give a hardware wallet some c to commit to, and maybe you
put it in a merkle tree with some other uniformly random data. You're
going to require the hardware walle to produce a commitment to something
sufficiently random, and then you aren't going to publish what that
commitment is. We're not going to publish c or everything about c. We
just throw the data away. The resulting nonce will have no trace of c
being in there. As long as the host was acting honestly, the resulting
nonce will be useless to anyone trying to attack this. So the security
goes down to either k is acting correctly, or the host is okay and is
able to reliably generate randomness with enough entropy.

If you do this the naive way, it doesn't work. If you give it c in
advance, then instead of using a hash function it can keep trying k
values until it finds one where it still has a detectable bias. If it
only has to produce 1 bit of bias, it only has to strike twice on
average. You don't want to tell the hardware wallet what c is going to
be. We want the hardware wallet to commit to it, and then re-randomize
it.

So instead of sending it c, we commit to some randomness we want it to
commit to. The hardware wallet returns an untweaked value, and then we
let it continue from there. Since you gave it a commitment to c, we're
able to ensure that if we tweak c in some way, it's not susceptible to
replay attacks. Since it doesn't know what c is, it doesn't know what
the nonce is that it will eventually produce. This winds up being
implicit in this slide; you have 3 steps on a hardware wallet, which
allows you to eliminate any possibility of a nonce sidechannel bias
using a biased nonce.

## Schnorr multisignatures

I am going to move on from sign-to-contract to talk about Schnorr
multisignatures. Recall the Schnorr equation from the first slide. I've
changed the Schnorr verification equation. Say we have n parties. Each
one has their own secret key. They are going to produce a joint key by
adding up all their secret keys and adding up all their public keys.
Because of the linearity of my private public mapping function, I can
just add secrets and add the corresponding public things together and
all the algebra will work out.

This is actually a way to produce a Schnorr multisignature. So during
key setup time, everyone produces a secret key, everyone throws their
public keys at each other. Here, P is the sum of all the public keys. At
signing time, everyone adds up the signatures, R is the sum of
everyone's contributions to R. Then we have a public key, a nonce, a
message, and we can compute a challenge e = H(P, R, m), and then they
can all compute a signature. And then you can add all the signatures
together.

Our verification equation as always just involves inserting G's into
that equation. If the final signature doesn't verify, you can go through
all the individual partial signatures and find the party that is
misbehaving and then kick them off and then restart. If you have already
started the multisignature protocol, I mean it depends on what your
protocol is what you do about cheaters. But it's always detectable.

But there's a problem here. When you generate the public key, it's a
serious problem. The problem is that if some party knows everyone else's
public keys, and they choose their public key to cancel out everyone
else's public keys, you can see this second line here where you add up
everything- if someone poisons the well here, they can produce a final
public key that is entirely controlled by them. So the sum of the public
keys can have contributions from every signer, but if someone cancels
out the other contributions... the result is something that looks like a
multisignature but it's really just the adversary's key.

There's a simple fix. It's not that simple of an equation, but it took
us over a year to get it. It took us two papers. One that was rejected,
and then another time our paper spawned someone else to write a paper
proving that our technique couldn't be secure. If you get through peer
review and someone produces an impossibility result, they claimed it was
impossible for our proof to be correct. But it's fine, there was a
simple fix. I finally got the email, that the paper has been through
peer review and it got published today.

Let me start with the simple fix.

The first simple fix is that we're going to hash up .... yeah, okay. The
first part of the simple fix that we're actually going to deploy. Thank
you, Greg. We're going to hash up everyone's public keys here. We're
going to get this thing that is re-randomized every time a public key
changes. And then every single signer produces something called a musig
coefficient. They take the hash of everyone's public key, and they hash
it out with their index in the signing protocol. If anyone tries to
change their public key in response to the other people's chosen public
keys in an attempt to cancel them out, this giant hash will change, and
then everyone's musig coefficient is going to change. And then our sum
of the public keys-- now has a random scalar factor on everyone's public
keys. There's no way to cancel this out. You can prove this in the sense
of provable security. There's a paper that was published today by us
that goes through all the gory details. Intuitively, that's the thing.
If you change your key, you mess up everyone's randomizers, and
everyone's equations that you were hoping to achieve there you
destroyed. So you add the musig coefficients into all the sums in the
equations.

## Verifiable secret sharing

I don't expect you to follow. This one has a quadratic number of secret
values that I am going to throw into an implicit matrix. Suppose that
somebody wants to shard their secret key. One of the participants in
this giant multisig wants to give the other participants some objects
that would allow some other people to reconstruct their secret key. And
nobody is going to reconstruct a secret key, they will just produce
signatures. Say you have 10 parties thta want to produce signatures, and
someone doesn't want to participate, but they want it so that the other
9 parties can reproduce the secret.

The way to do it is with something called Shamir secret sharing. They
choose a giant random polynomial. They choose a whole bunch of
coefficients. These gamma i k values are uniformly random. They choose
something that is the order of (k-1) where k is the number of
participants. They evaluate this polynomial at some small integers like
1, 2, 3, and 4. Then they give these evaluations to some people. If you
can imagine this, you can see a matrix of all these random values, and
inverting that matrix and solving for the secret from all these
equations for all the different secret shards. It's certainly possible
to do this. This is called Agrande interpolation and you get this final
equation at the bottom.

Given any subset of k parties, rather than 9-of-10 imagine 5-of-10 or
something.... it lets you choose a subset of 5 participants, you can
compute the Lagrange coefficients (lambda i j) you multiply those by the
secrets... and now you will see that you are summing over the people you
are signing is going to be the subset of the total participants.

In verifiable secret sharing, you throw G's into those equations. Using
verifiable secret sharing for threshold signatures has an issue; if one
party gives shards to all the other participants but in an inconsistent
way where it's not actually possible to reproduce a public key, you need
extra rounds of interaction and you can't detect it. If multiple people
are combining things, you either need to use a lot of memory and extra
communication, or else you find you're unable to tell who is
misbehaving. A goal in these kinds of schemes is that you want a way for
people to shard their keys in a way that anyone can verify that this
sharding was done correctly without needing to do too much
reconstruction.

So they take those coefficients, you mutliply them by G, and you get
public coefficients that you publish to all the other participants. So
they publish their public coefficients. They can verify these. Now they
multiply theirs in, and then they do the left hand side there, the left
most term is the participant public key, they add it up and.. they get
an equation which should be true if the issuing party is being honest.
That's verifiable secret sharing.

Using verifiable secret sharing, it's possible to do a multisignature.
If we start from a joint public key that is owned by all these
participants... the first equation on this slide is just my original
musig equation. I have some people in a party, they add up all their
contributions to the key and get a final key. The bottommost equation
says rather than summing up from everyone, I could take just a subset of
signers like say 5 of the 10, and just that set of signers is now
somehow able to sum up their contribution. It depends on the secret they
were issued during key setup, implicitly every party gets a verifiable
secret share, they give shares to each party, and then using
quadratically many shares communicating around, you have an equation to
work out, and now a subset of the original signers is able to sign.

What's cool about this is that after you multiply by these Lagrange
coefficients, you produce this, and it looks like my original
multisignature equation.

## Signing with verifiable secret sharing

The way that threshold signatures extend to multisignatures is two extra
steps. First there's a key setup, and all these shards passing around.
Second, when they start assigning, everyone participating in the
signature, which is the subset of the total set of possible signers,
everyone participating gets these red boxes... it's a sum that looks
complex on a slide, but it's just a few multiplications and adding up
the contributions from everyone. It doesn't take a lot of memory, it's
very straightforward, and we can throw in our G's thing if you want to
verify these things- you can verify the final signature.

The extension to threshold signatures is conceptually once you are past
the key setup, threshold signatures aren't different from
multisignatures in terms of implementation complexity or how you think
about security of the multisigning protocol. That's something I might go
into later afterwards if I feel like I still want to talk. The signing
protocol here... I've been waving my hands about how people throw nonces
into their pot, and compute a challenge, but that's a whole other set of
things that leak keys and stuff, but there's simple solutions that took
us a while to get to.

## Accountability

In the last slide, I mentioned this cool feature that once you decide
who is producing the signature, you throw in the Lagrange coefficients
and everyone creates the red box objects. After you add up all the red
box objects, you get the original public key. And you get a signature of
the total public key. The thing is that, the sum doesn't actually depend
on the specific set of signers used. These red box objects add up to the
same thing no matter how they were computed. The result is that the
signature doesn't have any trace of the original set of signers in it.
But there's no way to tweak this such that the signature has a trace of
the signers. So even if you know what all the red box objects, or which
signers were participating, you can't distinguish one signature created
with one subset from another subset of signers. This is called
accountability.

Accountability is like, if you have a 2-of-3 threshold signing policy,
and one of your 3 keys was a cold wallet key, it would be nice if there
was some evidence after the fact that the cold key was compromised. I
see a Bitgo employee nodding along with me here. With unaccountable
threshold signatures, you can't distinguish a signature produced using
hot keys versus one that was created in an illegitimate way.

What would an accountable threshold signature look like? Well, we have
one in bitcoin. You can take a bunch of signatures and concatenate them.
If you want to require that any 5 of 10 people sign some message, then
you can just ask 5 of them to sign the message separately, take all
their signatures and validate each one one after another. And if you're
Satoshi, maybe you validate them against different public keys, why not,
validation is cheap right....

But the thing is that, ignoring all the goofiness around the
CHECKMULTISIG opcode, which an eerily high number of you laughed about a
second ago, but here... It's possible to do better than having linear
growth in your threshold signature size, like every time you add a
signer you add an entire signature, but you can combine keys in various
way and get merkle trees and get a polylog sized threshold signature.
But the cool thing about the Schnorr multisignatures is that they are
constant sized and they look the same as a single signature. It seems
like you can't get a constant-sized accountable signature. There's vague
information theoretic reasons this should be impossible: as you increase
your threshold signature size, say 500 of 1000, you get this
combinatorial explosion in how many different admissable signing sets
there are. If you have a constant sized signature here, how are you
going to fit combinatorially much information in there? It's just an
intuition. You have to choose between constant-sized which is awesome
for privacy and efficiency and it hides your signing set and your
policy, and choosing between that and accountability.

## Semi-accountability: Sign-to-contract for accountability in threshold signatures

I'll describe a compromise here. This is what compromise looks like in
mathematics, between accountability and unaccountability. So I'm going
to bring back the sign-to-contract mechanism. I'm going to do
sign-to-contract in a multisignature. All of your signers can check that
the final signature is doing a sign-to-contract and what data they are
committing to, before they produce their signature. If anyone doesn't
like what is being committed to, they will see what is being computed to
because otherwise they can't produce R. So this multisigning business
gives us a cool ability where you can say not only does a signature
commit to something, but it's committing in a way that every person in
the signature agreed to this.

You can prove that if you extend a Schnorr signature by adding this
sign-to-contract data, then you actually get a strong signature on the
auxiliary data. The signature on the message you're signing is also a
signature on the committed data. This is a cool feature that I am going
to exploit here. What I am going to do is take my unaccountable constant
sized threshold signature, and in parallel to producing this I am going
to produce an accountable signature where I have everyone sign the data
and then I am going to take this giant accountable signature which is
growing with the set of the signers, and I'm going to commit that in my
unaccountable signature. My individual signers can enforce this data is
there. If you have an HSM where you can trust it with some business
logic, you can get some accountability on the hardware assuming it's not
completely borked. Assuming that at least one party in this
multisignature is being honest and is willing to reveal the commitment
to the auditor or to the public, then it's a hardware-enforced
"unaccountable" signature... It's not that, forget I said that, that's
not the name.

## Open questions

Can we construct a commitment that can be reconstructed or brute-forced
by third parties?

Can we get deniability, i.e. can a non-participant prove
non-participation without help?

Extension to BLS which has no space for committing data?

Suppose you have a 2-of-3 multisig and 2 people try to produce a
signature, but the 3rd person who might be some stilted party in an
escrow arrangement, has no way of seeing what the commitment is, and hte
third party can't produce a commitment and what's kind of worse is that
the third party here can't even prove they weren't involved. It would be
cool for one thing if it were possible for someone not involved in a
signature to reproduce a commitment and figure out which of 2 or 3
parties were involved. This seems impossible. But the less impossible
thing is, can you get deniability? Could one of the people who could
have in principle be involved in the signature, could they prove that
none of their data went into the commitment?

And what happens if we bring in pairing-based BLS signatures? These are
kind of neat objects. BLS signatures let you do non-interactive
multisignatures. It's unaccountability on steroids. You produce a
contribution to a multisignature, and anyone can take that and add their
signature and you don't need to know they were involved or who they
were. The result is a constant-sized signature. There's no way to
control that. Kind of a neat open question, is there any way to add a
form of accountability in this way to the BLS signature scheme? I don't
know. It's not the kind of problem I worry about. I don't spend a lot of
time thinking about pairings, but a lot of people in this space do.

A fourth open problem that occurred to me while I was talking.... I
mentioned that one example of an accountable threshold signature is if
you just concatenate individual signatures. But here's a fun question:
instead of creating like a separate signature during a threshold
signing, what if we just use the partial signatures on the second to
last line, could those signatures be used for the purposes of being
accountable? People are saying yes, I believe that's how science works.
Yes, this is how science works. This is what we call a proof. It's
social proof. So we have a social proof that you just sign-to-contract
all the partial signatures. Oh, there's a circularity here. You guys.
The issue, for those of you who aren't playing along with me..... yeah,
we can do that. Okay, so. I see, there's a simple fix here. The issue is
that these partial signatures are using this hash e which depends on the
commitment, and if I want to commit to the partial signatures then I'm
stuck because I can't produce those without already committing, but
there's multiple simple fixes from Greg Maxwell and Oleg Andreev. So
it's salvageable.

That's all I wanted to say, happy to take questions for a while. Thank
you all.

## Q&A

Chaincode Labs just announced a residency in 2019. A few weeks of
learning, and 3 months of working on open-source projects in Bitcoin. If
you are thinking about this space, then this would be the appropriate
residency. Any questions?

Q: You talked about deniability. Is there a way to force the other two
parties to make the commitment or not at all?

A: No. The other two parties have the ability to produce signatures that
commit to anything at all or nothing at all. There's no way to tweak the
keys or to tweak the verification algorithm that would enforce that
while still having the threshold policy property.

Q: Do these Lagrange multipliers... you said I should be able to deny
being part of signing, but I gave up those Lagrange multipliers.

A: No matter what set of Lagrange multipliers were used, you will still
get the same final signatures. The signature will be produced even if
you lie about the Lagrange multipliers. They still work, they still add
up, you just weren't part of it.

Q: About those commitments to the polynomial terms... remember there was
a paper by Girraro that served as a follow-up to the verifiable secret
sharing where they used Pedersen commitments with extra blinding factors
to all these terms, to solve some security problem with the initial
keys.

A: I remember that. Pieter, do you remember that?

GM: I think that's a rogue key attack. I don't think it arises in the
construction you're describing because you're using it on top of
authenticated keys. But this would be good to check.

A: We believe that because our original key was produced with musig,
there's no way to do key cancelation even in the sharding procedure. It
would be better to use a threshold scheme that was protected against
that. We should definitely check.

Q: About the paper you published in May 2018... I was looking at figure
3, yes this is the MuSig paper. It was the size of the bitcoin
blockchain with and without multisignatures. Like if they had
implemented musig from the beginning. I think the comparison was that
today it stands at 130 gigabytes and with that feature it would have
been 90 gigabytes. But you also mentioned on storage savings that if we
had also implemented key aggregation that would be more storage savings.
Could you talk more about this?

A: There's key aggregation and signature aggregation. Today I am talking
about key aggregation to get constant-sized multisignatures. The other
thing is signature aggregation where it is possible to aggregate
signatures thesmelves after they have already been added to a
transaction, and then you can get some further savings that way.
Signature aggregation has complex interactions with other parts of the
protocol, like how soft-forks are implemented and some issues with blind
signatures. Signature aggregation is probably a ways off, compared to
key aggregation where we have only a few engineering issues to solve.

A: That figure 3 only accounts for cross-input signature aggregation and
not key aggregation because we can't actually predict how people would
have...

A: There's two disadvantages of the approach that Andrew is talking
about. One is you lose accountability, and if bitcoin had that from the
start then maybe not everyone would have used that. It also requires at
least one extra interaction when signing which is inconvenient with a
hardware wallet stored in the safe. So some users may not use it. It's
difficult to estimate what the actual usage of these technologies would
be.

A: That's correct, but not an answer to question. It also equally makes
cross-input aggregation harder, though. But the figure was made by
counting every signature in every transaction and reducing it to 1. But
it was hard to count how many public keys were involved, because those
are in scripts that are committed to by the outputs. Plus in practice
what you would see if multisignatures were involved, you wouldn't use
multisignatures on the chain, you would use single key signatures. So
there's a problem of people will actually change behavior if this
wasn't-- in a non-observable way.

A: We often can come to agreement about a solution to these problems,
but we almost never agree what the problems are. So we are all
independently trying to fix our hobby horse, and depending on which one
of us gets the microphone we will talk about it differently. Pieter drew
figure 3. He had every right to answer the question about figure 3.
