---
layout: default
parent: Stanford 2017
grand_parent: Scalingbitcoin
title: How To Charge Lightning
nav_exclude: true
transcript_by: Bryan Bishop
---

{% if page.transcript_by %} <i>Transcript by:
{{ page.transcript_by }}</i> {% endif %} How to charge Lightning

Aviv Zohar

Our next talk is getting into layer 2. Leaving hwat needs to done at
th...

I am going to start with maybe a simple beginning talking about a single
channel between Alice and Bob. We heard about channels yesterday so I am
not going to go into details. Alice is not going to cheat Bob in my
scenario. Everything is going to run as if everyone honest and doing
their best effort, for my example. They open a channel and insert 10
coins. Every time Alice moves her coins to Bob or the other way around,
the liquidity within the channel shifts. So you can think of it as if we
are in a state space between 0 and 10. If we are just transacting in
single units of bitcoin we are just moving around there. I am going to
assume that just for the sake of modeling that Alice cannot predict
whether she will be paying Bob next or the other way or whatever.

Some basic fact about transaction channels... are that you can start to
do the math and for random walks this is well-known stuff. If Alice
starts to send money to Bob at a certain rate and Bob sends money back
at a certain rate, then we can look at the expected lifetime of a
channel. There's a formula and we can look at it in a graph format. And
it's slightly better. Over here is biased transfers on a channel if
we're slowly moving money then there's an optimal point as which we can
fund the channel. The x axis is balance that Alice has, we would like to
begin with most of the money on Alice's side. If we're doing, around
here, if the transfers are biased, so that, I think this is actually
Bob's side, so she is paying Alice slowly and money will flow as we slip
down this curve, right? We can even talk about balanced channels where
transfers go back and forth at equal probability. Benefits of lightning
show up here- if we put double the amount of money in the channel, then
the optimal way to start the channel is with equal fundings from both
participants, and the channel lifetime scales up with the square of
double. If you double the amount of money then you can quadruple the
channel lifetime. This makes lightning interesting, transfers get to
cancel each other out and the channel lives longer.

Once you start to look at this you must ask yourself-- we don't assume
that Alice and Bob are transacting a single point between them, they are
probably sending more between themselves. Sometimes we make very small
transfers, sometimes large ones. I often buy coffee or pay for a meal,
but only rarely do I buy a car. So there's a probability that I will be
doing a large transaction. And if you look at data for sizes of
transactions you see powe rlaws. The probability of a large payment
scales maybe inversely and quadratically with the size of the payment.
Right? So this is usually stuff we see in data. The exponent will be
different from 2, it could be larger, it could be smaller. This gives
rise to distributions of money transfers that are large.

And then you start to think about channels and you wonder, when do you
restart a channel? How much money would you put into it to get the
maximum utility from the channel? Beneficial to reset the channel even
if you didn't reach the border of the channel. Sometimes there's a large
hop that gets you in this yellow over here. But you might want to
restart the channel at this point. The next transaction that shows up
might be a large one, and it won't be usable in the channel because you
don't have the equilibrium in the channel funds.

If you simulate this stuff, you find that there's a-- what you're seeing
is the number of blockchain hits versus the radius at which I reset the
channel. There's an optimal point, other than waiting for the channel to
completely run out, you reset before it gets to the border.

We looked at basic channel behavior and we asked how much do channels
cost? There are two main costs to a channel. If you increase the amount
of funding in a chanenl, you increase its lifetime. Can I put in as much
money as I want, can I just pour a lt of money in there? The limitation
is that I can always get more money but I have to borrow it and pay
interest rates on it. So if I want to fund a lot of channels and lock up
funds in there, they are going to be moving back and forth over the
channel but never out of the channel. They are losing potential interest
if they had put the money elsewhere. There's another cost, which is
channel setup and settlement which has to touch the blockchain and it
has fees and fees might be high. So you are trying to mitigate between
these two costs. These are the things we will consider.

The next thing to ask is how are fees collected? What would a lightning
transaction pay and what would a bitcoin transaction pay? For regular
on-chain transactions you have fee rate volatility. There's a somewhat
fixed fee for bitcoin blockchain use that's how it's built. On
lightning, it's somewhat unknown. If you think about it, you will notice
that if you do a large transaction in a ightning channel, you shorten
the lifespan more than you do the small transaction. So you will incur a
cost if you reset the channel and estbalish it again. It makes a lot of
sense, at least in my view, to charge lightning transactions not a flat
fee, but a proportional fee to the amount of money being used. In some
instances, a large transaction should cost superlinearly in some sense,
since it hurts the channel much more than just its size.

This is also going into our model. And of course what we want t od ois
we want to give transactions a choice. I am assuming that people are
going to have this really nice wallet, and UX/UI wallet problems will go
away. And you are asking to send money from here to there, and it's
using the channel if it's cheaper, or use the blockchain if it's
cheaper, or you might see the fees and you say it's too expensive and
you say you're going to use something else like VISA or Western Union or
myabe you don't make a transaction at all. So this is our model.

Now I am ready to show you the approach going from this micromanagement
of channels to what happens to the economy. What we're going to do, in
general, is to think about a pattern of moving money. It's going to be
really simple to begin with. And then I am going to talk about channel
management and topology and so on and then the market of delivery for
these fees. And basically, right, the intuition that I want you to have
in your head before I show you some of the results is that we're going
to have these large transfers that really prefer to go on the blockchain
because on the blockchain they pay a flat fee, and on a lightning
channel network they pay proportional to their size. Small transactions
will prefer the lightning network. Together they share the cost of
establishing the channel and so on. Both of these sources of
transactions are going to touch the blockchain for different reasons,
like establishing channels and direct transactions. Do the prices
interact with eahc other? Are large transactions common enough to
squeeze out the lightning transactions? This is the exact question that
interests me.

Here are some made-up parameters that I picked. I am assuming every
person from now that I talk about has 10 tx/day. That's all they do in
expectation. We draw the amounts they use from a parallel distribution
like I defined before. Each of them are willing to pay up to 1% of what
they are transacting, in fees. There is a 4% yearly interest rate. I
know it's lower now, it could be higher, I don't know. It's made up. And
there are 288,000 on chain transactions that we can do every day. This
is baed on blocksize before segwit because we don't have exact numbers
on what segwit is going to do on transactions until it gets fully
adopted I don't have a good estimate I guess.

In terms of topology what am I going to assume about the world? What are
people trying? I am going to assume a nieve model which is just that
people are paired up together. And they are just Alice and Bob are
sending money back and forth. This is actually designed to be nice to
lightning. We know exactly which lightning channel to open, and no
multi-hop. And their partner there, they can send money back in forth.
In reality, it's much more complex. Another thing that you might
consider is some uniform topology where each person could be paying
another with equal probability and we don't know which 2 people are
going to be transacting. In this world, what is the topology of the
lightning network? One obvious solution is to model it with a central
hub, which simplifies the math and makes it easier for lightning to work
efficiently. This is the model that I am going to be talking about.
These things look different in how much money they end. But really they
are not too far apart. Because the model with the pairs has one
transaction channel for every 2 people. And the model hub has one
channel per person. So it's a factor of 2 on maintaining transaction
channels if we had exactly the same behavior and same funding in both. I
am going to talk about the pairs model and just as a walk-through model,
and we have done more in the paper that will come out. Let me walk you
through it.

Here's the behavior of individuals. Here's the funding of a channel.
We're assuming we have some blockchain fee, could be in BTC or
something. This is just an example. Given this fee, I can decide on how
much channel capacity to choose. I can make the channel large with a
bunch of BTC or small. I am funding it optimally where both sides have
equal funding and we are assuming transfers back and forth with equal
probability. As I grow the channel capacity, the red curve is cost, it's
one as the channel capacity grows, I have to touch the blockchain less
often (green line goes down), I do less resets onthe channel. The blue
line goes up the more money I put into the channel, I pay more in
interest opportunity cost. There's an optimal point where I want to be
around to fund the channel and therefore minimize my cost.

We can have a curve like this for every curve. So what happens to the
channel capacity when we change the fees? if I increase the fees slowly,
the optimal channel capacity that I am going to choose goes up like the
square root of the fee times some constant. This is the exact benefit of
lightning- I double the fees, I only need the channel to have a acpacity
square root of 2 times larger to manage it and have good behavior. And
from this we can derive demand for blockchain records. If I put some of
my funding into the channel, I can now figure out which transactions
find it expensive to go through the channel, and from my different
payments that come out of different sizes, and I have this demand curve,
and if you look at it, this is the exact curve on log-log scale which is
a straight line, it's a power law. So I have the demand scaling down as
one over the fee, and htis is for a world that has no lightning in it at
all. If I can't use lightning because of the way I chose my distribution
I would get this demand curve. In a world with lightning, the curve
looks a little bit like the same, but log-log, instead of 1/fee it's
more like 1/sqrt(fee) and that's a minor difference. What would it do to
us? We'll see.

The next question is what happesn when I scale up. This demand function
is for a single Alice-Bob hcannel and at differnet fees how many times
you will want to touch the blockchain. So now I am adding lots of Alices
and Bobs, maybe 100 million, each who want a channel with their partner.
Here are the fees, without lightning, when we scale up the number of
users, we go to 100s of millions, the fees go up. The fee for settlement
and blockchain space, goes up. Don't get hung up on the numbers, because
remember we made up the distributions. But look at it qualitatively. The
next thing of course is what happens with lightning. This gets
interesting. The demand curve had a different shape with it... You can
tell probably that the we're having very different fees. One thing that
is obvious to begin with is that we reah really high fees at the end.
Lightning tends to collect more money from people and they are willing
to pay it because it amortizes cost over the lifetime of the channel,
they are willing to pay more to estbalish the channel. Between 0 and 20
million people, we're actually making less money with lightning because
we added a tech that allows us to move some stuff off the chain and
there's less competition for the chain space. So there's lower fees. And
then it gets higher after 20 million people.

For miner's reveneu this is exactly proportional because there's a fixed
amount of spots on the blockchain. So if the fees are lower, miner
revenue is lower. If the fees are higher, then security goes up. So
we're seeing the effect of lightning here. I know this other thing is
controversial, but let's talk about the block size- here is the world
with lightning, what happens if we scale it up with segwit? I know this
is highly volatile and it's a hard topic to talk about, but with an
extra increase in block size, the fees drop. Instead of 25 here,
there's 10. It drops by more than a factor of 2. So there's more space
because of the increase. The total revenue of miners has gone down by
the block size increase. But we will getting more throughput, but the
security in this case, for this model, for these assumptions, has gone
down.

Here's the conclusion. I hope I hae interested you enough to come and
discuss this with me later. Lightning definitely helps. But now you have
to decide for yourself whether it's a lot or a little. I am not sure
what to think. I was expecting lightning to do a lot more in some sense.
I wanted a factor of 100x on transaction throughputs but we got
something less than half of what was already there. Maybe we can do
micropayments and suddenly it's different. A 2x block size increase
helps but not by very much. It's not that amazing. It's not surprising
though because I showed graphs that took user count from 10 million to
100 million, and I only increased block size by a factor of 2, so
obviously that's not going to help a lot. There's a lot that I didn't
show you, which will be in our paper. We are looking at heterogenous
population, but really, some people are moving more money than others,
some will push others out of the blockchain... the results will be
different. There are more complex patterns of flow. The pairs model is
super simplistic. There are other distributions of flows, like multi-hop
networks, circular where people don't just send money back and forth.
And usually people get money from their job. How would lightning
channels look like in that scenario? And in general I think I would like
to voice my concerns.. I don't know how fragile my models are, how much
these assumptions are bad? Would a change in interest rates do a huge
shift in the results? Would we see an economic effect? The fees are very
important there. Everything in the model was, you know, the average or
expected behavior-- what about fluctuations? We have those often in the
economy, right. So I'll leave you with more questions than I hope we
started. Thank you all.

## Q&A

Q: I am wondering how viable do you think it would be that when we have
to move to the chain because your channel becomes too big to be feasible
on lightning. Why not just purchase multiple services from the
counterparty? And what about resets? I send a little more money, and I
have my counterparty reset my channel. Wouldn't it help me a lot to
increase the apacity to use the on-chain for the reset and the payment?

A: Yes we have assumed something like that. If you do two things with
your ttransaction it's going t be larger, but then youre paying for it
more because you're paying per bytes and not per transaction. There are
some constants that need fine-tuning. Paying someone is a different
transaction size from setting up a channel. That would effect the model
at the edges. We used constants, because we didn't know what values to
choose.

Q: I'e been thinking about lightning and UX and transactions coming out
of our exchange. One thing I was wondering about... what are your
thoughts on what you think transaction olume would be effected by people
wh want to send on-chain and .. top-up their channel or do some sort of
channel operation like when you add a UTXO to a channel you're sending a
coin to the multisig on the blockchain... and adding an UTXO and saying
x amount is the other person's channel. Sending on the chain, but also
adding to the channel. Do you hae insights into what that would appear
like on the blockchain, and whether you can put that into your models at
all?

A: We thought about this a lot. There are a lot of interesting questions
exactly there. It depends on how cooperativ vv ve the othe rparty is...
if you're putting more money in the channle, that's not going to happen
often, you're locking more money in there. People might send their
counterparty directly, and then maybe someone send sit on the channel,
to reset, we're doing it in both layers. We thought about this. Our
reset transactions are 1 tx, because we thought people would be trying
to save money.