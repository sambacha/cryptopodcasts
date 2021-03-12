---
layout: default
parent: W3 Blockchain Workshop 2016
title: Groups Identity
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %}

The topic that we have been working on is identity for everyone. In
terms of trusted set of parties if you will, that can potentially
provide some level of attestation to the identity and that can be
provided in a digitized manner as to what levels of service an be
provided to that level of identity as well.

---

What belongs in a blockchain? What works in a blockchain? What doesn't
work in a blockchain? How should it be stored? How can we make sure that
there's redundancy? What systems are reliable? Should we use cloud
providers, or distributed hash tables, should we put certain data in the
blockchain, we talked about zone files of the existing domain name
system and how they can be used to have pointers to data store anywhere
on the web. We talked about how in the blockchain you might want to
associate a set of public keys with a domain name, and associate it with
a hash of data where the data might be stored elsewhere. We brought up
how, where hardware devices, we brought up the challenges of private key
storage and protecting private keys. We brought up what happens when
there's some kind of key-compromise and how we can recover from that
scenario. We talked about insuring that people can use multiple
identities in multiple contexts, being able to preserve their privacy,
being able to minimize data that they convey when they interact with
someone else or some other application. I did we believe have a
consensus on the fact that as little as possible should be stored in the
blockchain. The blockchain should be used for very key use-cases, like
associating a public key with a globally unique name with a hash of
data. Everything else could be done off or outside the blockchain. You
can cram things into the blockchain, depending on your use case, but if
you identify that as the common minimum core then that clarifies and
informs how someone would design and implement a blockchain-based
identity system. What would be the next step? I believe the next step
would be for people to, for everyone to come together and decide on some
architectural standard around, like we agree on these principles and
these ways on organizing information on the appropriate roles for the
blockchain and cloud storage and everything else and agreeing upon
standards like using DNS zone files. And hashing those zone files and
putting those hashes on the blockchain. Use the existing standard of DNS
zone files, and the existing format, for encoding pointers to data
stored elsewhere. Things like that, agreeing upon the standards of those
components in the different layers of the public key cryptographic
identity naming and contextual identity layers.

---

Verifiable claims data model. We did not spend that much time talking
about it. We talked about some interesting requirements and concerns.
The idea is whether the idenitty has the right to reveal and conceal
information about themselves. Selective disclosure came up a couple of
times. Privacy was a big concern. Different actors have different
requirements. Issuers of a verifiable claim is going to want to be able
to revoke and edit claims over time. Whereas the receiver of the
verifiable claim will need to be able to store it for a long time. There
was some discussion regarding trying to ... the current legal framework,
can we take it and make verifiable claims happen that way? There was
some healthy debate in our group. There was an assertion that we don't
need to standardize verifiable claims. That the problem is elsewhere.
That blockchain is not a panacea. That was said a number of times. Folks
wanted to know about the economic incentives of building identity
systems. Are these aligned iwth the people going to be using it? What
are the lifetimes of verifiable claims? How long do you have a right to
be forgotten effectively? I think the group is good about not delving
into the perma threads. What is an identity? What composes an identity?
Even the statement where we said a set of attributes makes up an
identity, was controversial. I think we avoided usual pitfalls. I don't
know if we came to any conclusion. More discussion is good.

What are the next steps? A number of folks are going to join the
verifiable claims task force at W3C. That's where you can learn some
more about this work. And that's kind of the first step in participating
if you're interested in expressing, transmitting or verifying verifiable
claims.

What was controversial about the identity being a bundle of attributes?
Ship of theseus. You can show how something can have 100% change in its
attributes over time, and have literally zero attributes from the
original ship. The only thing is persistence. So bundling attributes can
be a problematic way to define identity.

---

Efficient thin client verification of identity

We discussed about efficient thin client verification of identity. Given
a username, what is the set of public keys associated with that
username? How can a thin client get this set securely and correctly? And
how can we do this on a system like bitcoin? The assumption is that the
client has all the blockhashes but not the block content. I would still
like to get the set of public keys. I would like to do so efficiently
without downloading the whole blockchain. There's work on this like
blockstack, namecoin, onename, and others. You collect all the blocks
then query the state. The first problem is the non-membership proof
problem. Blockstack has a merkle root for the entire state of the
system. You can show that a particular operation belongs in the set.
There is also a lite client proof system. A thin client cannot give a
non-membership proof for the root. That's the non-membership problem.
The other problem is the freshness problem. On the non-membership
problem, there might be some solutions like in ethereum where you index
the data in your blockchain and you commit the index as a patricia tree
or sorted merkle tree. Another way is using a linear commitment chain.

Once you have a patricia tree merkle root in the block header, you still
need to know what is the most recent blockhash. There is roughly three
ways to do it. The first one is with proof-of-work. You want to get the
current blockhash. You want to get 6 confirmations of blockhashes to
know that it is confirmed. Or you can query a lot of people for a
blockchain hash chains and see okay this is the longest one, in this
case that's the longest answer for like six blocks away. Another way is
a classical BFT system like tendermint is doing. In that case you can
get an answer about what the most recent blockhash is. The third way is
with an oracle system, like to commit collateral on the chain and if
somehow they cna be enforced to only sign correct blockhashes or else
they get their collateral slashed, then perhaps that's the only way to
get an answer with a guarantee.

Lastly there is something about out-of-band broadcast. Perhaps you might
trust satellites to give you the correct blockhash. There's two things
to consider. There's the non-membership problem like in patricia trees.
Then there's the recency problem with proof-of-work, BFT or oracles.

Next steps? Tendermint should specify its lite client protocol.

---

I would say that our super short summary is "too much identity". We took
a different view. Identity is not good. This is something we're trying
to avoid in many of these systems coming from bitcoin and other
projects. Identity is a powerful tool. You can do a lot with it. We
should have standards where you want the least possible amount of
identity. Only rely on identity when you have gone through the other
options. We already think there's too much identity, unique identifiers,
tracking, etc. Hopefully we can get rid of that with some of this.

You might have a spectrum of identities where you have total anonymity,
or ephemeral key pairs where there are session key pairs perhaps, and
then longer persistent identities that might be linked to a legal or
biological entity. We think best practice would be to use the weakest
and least powerful identity system. Use attribution. Use a set inclusion
proof instead of a unique identity.

If you go to the taco truck and order a chicken taco, if they ask me for
my name, I often give a fake name. They don't care what your name is,
they just need a unique identifier. This is an ephemeral session token.
It does not need to be linked to a legal identity. That's one aspect.

Another attribution aspect is that, say in the future there's a taco
drone delivery service using a darknet market or something. I want an
inclusion proof that the FDA or taco inspectors have inspected the taco
facility. But I don't need to know who's making the taco. All I need to
know is that it has been inspected correctly. Use attribution when it
suffices. Use ephemeral identities when possible. Use one that is not
linked to a legal or biological identity.

Have repudiation when possible. Don't sign, use diffie-hellman. Have
something where it can be deniable afterwards by parties. Data is a
liability. You don't want lots of customer data. You don't want lots of
user data. You want to minimize this to the extent possible.

Next steps? We should have a best practices or sort of strive to work on
this when making applications in the web space or blockchain space.

How does this relate if at all to persistent pseudonymous identities?
That's a fairly powerful identity which can still be unlinked to your
legal identity. A lot of bitcoin is based on pseudonyms where we don't
know who it is, but we see a persistent entity. Having an ephemeeral
identity discarded after each use is safer in many ways. Once you link a
pseudonym to an actual person, you can go back and figure out
everything. You don't have forward secrecy in that scenario.

---

IBM stuff

They have proposed an architecture for hyperledger based on IBM
membership services. It's a combination of centralized and decentralized
services. They use the certificate authority nomenclature and structure
as a way to build this service out. It's a system built with the idea of
long-term identity pieces and short-term identity pieces. When you are
first creating or setting up on the blockchain, actually this gives a
good reason to step back for a second. They built this proposal with the
idea of private or permissioned ledgers in mind. Some of the things you
will hear me talk about, may be more relevant in the private case than
the public case. They have this idea of long-lived certs, called
authorization certs. This gets generated when you're creating an account
or signing on to the blockchain for the first time. Then the idea of
transactional certs where the transactions might have subset of
attributes on it, and only used on a single transaction. The purpose of
transactional certs is to help reduce and prevent correlation so that if
someone is involved in one transaction can't necessarily see the other
transactions you're involved in. One of the ideas was building something
for the first authorization, you might be looking for a generic
attribute, like "this person is a licensed plumber". When they come
back, you want to know that it's the same person coming back.
Specificity might go down over time in terms of who you're dealing with.
We're heavily dependent on third-party validators in terms of our
certificate authority structure. If your counterparty is taking any risk
in th etransaction, they might want to have someone other than you who
has signed off on your attributes. That was a lot of the high-level
pieces that came in.