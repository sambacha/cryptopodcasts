---
layout: default
parent: Stanford 2017
grand_parent: Scalingbitcoin
title: Using The Chain For What Chains Are Good For
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %} Using the Chain for what Chains
are Good For

Andrew Poelstra (andytoshi)

Video: <https://www.youtube.com/watch?t=5757&v=3pd6xHjLbhs>

I am Andrew Poelstra, a cryptographer at Blockstream. I am going to talk
about scriptless scripts. But this is a specific example of a more
general theme, which is using the blockchain as a trust anchor or
commitment layer for smart contracts whose real contents don't really
hit the blockchain. I'll elaborate on what I mean by that, and I'll show
what the benefits are for that.

To give some context, in Bitcoin script, the scripting language which is
used to encode smart contracts which go on to the blockchain and eeryone
verifies the chain and to download and execute all of this contract
code. Everyone validates the chain. Everyone that wants to trustlessly
learn the stat eof the system has to download and parse and validate.
They can't compress or aggregate the data. They can't make it smaller
due to kolmogorov complexity. The contractors doing this, before they
publish, to be ensured that it doesn't get reverted, the transaction
must first be confirmed. This is a poisson process. Once your
transaction lands in a block, you need additional assurances. You never
have perfect finality, too.

In addition to this, the contracts encoded in script have rules that
everyone needs to agree on. If there's any disagreement in the system or
network about which scripts are valid or invalid, would cause a
disagreement on the history of the chain and that's a consensus failure,
and as a result everyone needs to agree on the rules beforehand. So
adding more functionality is often a painful process, as we have learned
recently.

Additionally, I also care about the details of the script. The details
have to be there and stay forever. If people are doing a complex
transaction amongst themselves, tmaybe they don't want that data to be
visible outside of their group. But once your contract is published,
everyone sees it, it has to be kept forever because future validators
need to be able to see it. And related to this, when transactions are
collected into blocks, miners can choose whatever transactions they want
and they are incentivized to go with the highest fee but this incentive
is only within the system. There are external incentives though, and the
real world can interact with the mining process here. Miners that can
see contract details become a target to government and external systems
and they might not want that liability. We don't want them to be able to
see that data, because it's a censorship risk, and it's a liability to
the miners which they don't want to have.

All of these contracts executed by explicitly published code are really
only usin the blockchain for one thing- to get an immutable ordering of
what order tha ttransactions happen in. All that they really care about
is that the transaction is not reversed and not double spent. This is
the core competency of bitcoin blockchain. This is what I mean by my
talk's title. This is what needs to go into the blockchian. If we can do
that in principle, we hsould be able to aoid putting anything else in
the blockchain into there. It should just be inputs and outputs. The
exact conditions under which this can happen can bwe reduced to a ery
small amount of data that doesn't reeal thwat the commitments are.

There's a distinction between alidation and execution. We see this in
two places. In general in computer science, when you talk about turing
machines s turing deciders, there's a post theorem that shows that it's
strictly easier to alidate the execution than it is to.... because you
can proide a witness which shows it. Instead of executing the script. In
crypto, we talk about validation vs execution. In computer science, it's
about how expressive it's required. Verification of something can be
done in zero knowledge where the helper data is not revealed to
everyone, but through the magic of cryptography everyone can verify that
the extra data existed and it was correct.

In addition to execution vs verifiability distinction, there's
verifiability vs public verifiyability. The blockchain verifiers care
about the state of the system like where the coins are, how many coins
there are, what they are assigned to and so on. When they see a
transaction, they want to know that the transaction is authorized and
that they agree. But they don't care too much what it truly means. When
you're transacting, you care about faithful execution and the rules you
want to enforce. But everybody else doesn't care- only that the rules
were followed and that whoever owned the money before was somehow okay
with it moving. This is much more nebulous thing than what the
transactor probably cares about.

There's kind of a general way to do contracting, which Adam Gibson
talked about in a recent or upcoming blog post where you can imagine
instead of doing some complex ethereum contract or series of bitcoin
scripts... suppose you only care about money moving only under external
conditions. You can moe your coins to a ultisig output. Eeryone has to
sign off on it. When you set itup, you do a locktime refund transaction
and so on with a timeout. Under some external conditions, each signer
cares about, checks the conditions and only then do they sign off. What
the blockchain alidators check is that the signatures are present. They
don't care about the external conditions are. They odn't need to
download a description of those conditions or anything like that.

Suppose, though, that the conditions that the signers want to enforce
are not just external things about the world that they can look at.
Suppose they want to enforce conditions on each other, the classic
example is an atomic exchange between blockchains, like giing money to
someone in exchange for other altcoins. And bitoin and tehereun ahae
dierent lofefaidf. And each one on deach chains. And if I sign to gie my
coins away, my ice ers,a we hae to sign simultaneously. We hae to
enforce onditions on each other. I will talk on the next slide about how
todo this.

You can do this in bitcoin script or ehteruem or whateer, ... in the
classic way where in order to take the coins you hae to reeal a hash
preimage. In both blockchain, reealing this hash preimage reelation is a
requirement. One person knows the hash preimage, to do this they reeal
hash preimage which they do by creating a bitcoin script that says these
coins must not moe until someone reeals the hash preimage to take the
coins. At that point, the other person can read the preimage off of the
public blockchain and take their coins. And thus atomicity has been
achieed.

This isn't good: it links the two transactions becaus eyou see the same
hash preimage. And it also forces the erifiers to download the hash, the
preimage, and check them foreer. The erifiers don't really care about
that. Only the two transactors only care about the hash preimage, and
only then for a limited time. But somehow the data is still there
foreer?

I hae been working on a way to do scripts in an inisible off-chain way,
using witness data like a hash preimage is hidden in the signatures
themseles. Validators will always have to check signatures themselves.
Validators are already checking this. My program here is, how much extra
validation can you oerload these signatures with? So only the people
producing the signatures know this other stuff. But the validators won't
know.

I am taking digital signatures and adding some extra semantics to them.
It turns out that you can do a lot like this. This historically came
from the mimblewimble project that does not have support for scripts.
There was an ope nquestion when it appeared: how do we do any
contracting, atomic exchanges, lightning? One answer turned out to be
scriptless scripts, and that's where it came from, but it turns out tha
tit's applicable to bitcoin. Most of what I am doing requires that we
have support for schnorr signatures in bitcoin, which would require a
new opcode. It doesn't add any new semantics to the system, it just
creates an alternative to the ECDSA signature algorithm that people can
use if they want to.

Benedikt talked about schnorr signatures yesterday. There's one equation
here, but it's jus ta plus sign, so it's okay. In schnorr signatures,
you can create multisig with two parties or more, that look... same..
single signatures. They do this by interactively producing a signautre
on a joint signing key. Each one has a signing key. They add them
together and they are able to make a signature. A cool feature here is
that when they are doing this interaction they first agree on the first
half of the signature which is something like an ephemeral temporary
public key (a nonce). And then they produce the real signature. I am
going to stick a bunch of extra data into those steps. It never hits the
chian. Once it hits the chain, it's just a signature. But we can do some
cool things in this space between signers.

I am going to use an "adaptor signature" where the two components of
these schnorr signatures can be modified in such a way that we can add
some random number t such that you can get a real signature knowing the
secret value t and knowing the value secret t you can get a real
signature. Knowledge of this value t becomes a key to producing this
signature. The way this is done is using this thing called a valid
signature. Someone can verify the adaptor signature. There's a little t
value and a big T value.

Once Bob signs, Alice has some secret information that she can use to
get her coins.

Adaptor signatures can be used instead of hash preimages. These discrete
log challenges, have a bunch of extra structure. One is that you can
make them work across elliptic curves with some extra cryptography that
I will publish in the next few months. So even between Monero using
ed25519 and then bitcoin with secp256k1. The adaptor signatures are
undetectable and they are deniable. What winds up hitting the chian are
just these normal looking schnorr signatures, and this off-chain
interaction where Bob secretly re-blinded these and passed them to Alice
is undetectable and deniable. Anyone can take these two signatures after
the fact on any blockchain, add them and make them look like an adaptor
signature or whateer; there's no eidence that the original stuff
happened at all. This is good for privacy and fungibility. Coins that
are used for various protocols like this are indistinguishable from
coins used for normal p2p protocols.

An additional cool feature of scriptless scripts is that these adaptor
signatures are re-blindable. You can chain not just two transactions,
not just two transactions made atomic. You can make arbitrary chains of
transactions atomic. I can make all the hops happen atomically. This is
what lightning is based on. You can do this knowing that there's no risk
of coins being stolen because it's all happening atomically. There's a
privacy issue with using hash preimages ( a standard way to do this ),
where each party reeals a hash preimage and then the other parties can
observe this. But eahc hop could have a custom challenge, and translate
it into a new challenge. The participants in the hops are unable to tell
without complete collusion that they are part of the same path even
given the extra data. Nothing that hits the chain, no hashes that they
can look at, to identify it. Even given the secret data they are passing
amongst each other-- even then each hop in the path involves uniformly
random data that is uncorrelated with every other hop in that path or
any other path. So that's great thing for privacy on lightning.

## Q&A

A: You can do threshold signature tricks and that is completely
compatible with this. You can have each party revealing different things
to differnet people and they can have different weird structure ot their
adaptor signatures.

Q: How do you reeal the alue?

A: Alice ... alng with a alid signature with little t. Once you know s +
t, and if you learn t or s, you can get the other one.

Q: So the signature will...

A: Exactly. There's an additional thing gien. That's the adaptor
signature, is s + t.

Q: ... linkability between the two... like atomic swaps.

A: No. The reason is that, after the signature is on the chain, there's
little s hitting the chain. Anyone can make up some t value and T value.
Add little t to the signature and... and pretend it was some other r
value. You can do this with any t value, it's independent, there's
nothing tying this particular t value.

Q: Storage and transactions implication of this approach?

A: Using schnorr multisigs, there are multiple signatures being combined
into one. A signature on any transaction input could be reduced from 128
bytes to 64 byte signature. In addition to that, using something like an
atomic swap, and something like a hash preimage, thats another 64
bytes.. In this atomic swap example, rathe rthan having about 200 bytes
of data which is multisig plus a hash and a preimage, that's collapsed
into one signature which is only 64 bytes. And on top of that there's an
additional topic called aggregate signatures which can reduce from 64
bytes to 32 bytes. The rest of the transaction is the same, the shape of
the transaction is the same.

Q: This feels related to Tadge's work?

A: I would love if Tadge would describe his work as a scriptless script
so that I can claim to have a wide umbrella. I described adaptor
signatures-- you could do other stuff like pay to contract, sign to
contract, discreet log contracts, and another thing I hae been working
on that might be public in the next few months. I wanted to gie an
example of what hiding hte contracts offers to the chain. I wanted to
motiate the paradigm.
