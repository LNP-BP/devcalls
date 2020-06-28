## Transcript/notes of RGB-Beta Demo Q&A session 
[dev call of 24th June, 2020]

### 1
G:> Final acceptance. When the receiver accepted he could check the validity of the internal consignment, but he could not check that the UTXO was actually spent..
We were omitting the part where you get the witness back to the wallet and sing and broadcast it in testnet. So, the receiver now can check the validity but not the confirmation.
Is that correct? 

M:> Right now we are not connecting to btc blockchain or LN... 
You need to set up the bp node and have bitcoin core connected. The interface to bp node will be finalised over next week. You will need to run the bp node indexing 
of the whole blockchain (which would take several hours) and then you will be talking to bp node from RGB one.


### 2
G:> Here the security model is like in bitcoin: validity and confirmation, in a sense that the daemon can already check that the consignment contains well formed transaction 
and then you check for double spends. So you can’t say that the tnx is confirmed, but you can say that it’s valid, right?

M:> Validation consists of 3 parts and the validation against btc thxn graph either from blockchain or from LN channel is one of those. You can do 2 others 
(of the internal data consistency, against Schema and Schema scripts, but you can’t do the V against btc txn graph, meaning that you wouldn’t know that the witness commitments are
done because you will need the info about btc txn fees, which can’t be extracted from offchain data and you can’t trust that data even if you will save that information; 
you will need to make sure that the outpoints are not spent and you can’t check that the actual commitments are done in txns that are part of a state channel in LChannel 
or a part of btc chain.

### 3 
M:> RGB uses btc txns in 2 ways: 1st - to put the commitments to what happens with RGB offchain (commit to each transfer which is called a state transition) into the btc txn. 
[Witness - the txn that contains the commitment, it’s a part of SUS witness.]
2nd - you use unspent tnx output to allocate some state to, in particular to allocate some assets to this tnx output. And when this output is spent, it means that you actually 
did a transfer of that asset and the txn that spends this output, must contain the commitment (meaning it should be a witness txn). Basically you are using 2 txns: one you need to
assign assets to, other - witness txn. Key idea of A. and G. from the early days was that basically you can combine these 2 tnxs into 1, meaning that the W txn may contain 
the output which contains the asset that you are transferring to smbdy, meaning that you assign the asset not to the existing txn output, but to the output that will be created 
by the payer, by the person who pays you. 
The question is if we need this opportunity.

G:> So one option is that I give you the outpoint directly, the second - I give you a normal btc address and you create the outpoint within the same txn where you actually burn 
the UTXO.

M:> yes, but it has 2 problems. I dont just provide you with the valid outpoint, I need to provide a signed input to that txn, because you are paying me USDT (assets), 
you don't pay me btcs, and if I provide you with just a btc address, you will need to allocate some of your btcs to that address. But I’m not paying you btcs, I’m paying you 
assets. So, basically you need to give me additionaly to the btc address in form of input of partially signed txn, which I will merge into my btc txn. 

G:> Scenario where the receiver creates/gives another outpoint is more complex in theory, but it’s actually better from many points, because it’s also the scenario where a user 
can use another btc txn in order to further move the asset, so it also allows for less RGB specific onchain footprint. I cannot see any case in which we might use a simple address
based txn. It could be what we started from, but then we somehow abstracted it in this UTXO-specific model and I cannot see the situation that we might need it with in the future.

A:> The use case can be when smbd creates a new wallet and doesn’t have any utxo and wants to receive an asset. In this model he needs to first receive some btc and then attach 
the asset to this txn. He probably needs btcs anyway because you will need to pay fees.

M:> In this case. If I need to pay to you (you have a newly created wallet) 100 USDT, it is my obligation to provide you not just with USDT, but also with btcs, which I wouldnt do.

G:> basically you cannot receive assets unless you have btcs/utxos in your wallet.

M:> no matter if we include this ability or not, you still need to get outputs with btcs, but where will you get it?

A:> if we use the address-based set, you can send to an address instead of existing utxo. 

M:> but you can’t send to an address. doing that means that as a payer I need to create a txn output using the address you provided to me and this output must contain some btcs.

A:> yes, but maybe there is a case where smbd says “I want to receve an asset+some btcs” so you can just receive them and keep them there. Or if you want to withdraw 
from an exchange, you want to withdraw both the asset(s) and btcs in one go.

J:> Can i still receive an asset for free without btc to an empty output, still own it and then receive btc and spend btc separately to be able to transmit the asset?

A:> no, because there is no concept of an empty output, so you can receive to a 0 value output, but you can’t spend it because it’s below the dust limit.

M:> and it would be obvious to chainanalysis tools that these outputs are used for RGB. 

G:> one guy that sends me an asset and one that sends be some btc in order to enable me to receive asset.

M:> there are both options present now in RGB (not in the command line demo version, but in the protocol and in the code). The decision needs to be made in order to reduce 
the complexity that we have now. I think that exchanges will not do the withdrawal of 2 different assets at the same time from security and other reasons. 
<....>
M:> if both sender’s and receiver’s wallets support the ability (which is complex) to send and receive btc+assets and have an interactive protocol.
<....>
M:> if you send me some assets and I don’t have any btcs, we need to negotiate that additionally to the assets, you will send me your btcs and I will own them. 
Why would you need it?
<...>
M:> technically you can do it in many ways. if I dont have any btcs, there are always many ways for you to send me both the a and a btc niot withstanding which decision 
we will make on a protocol level, it’s only a question of how wallets need to implement this functionality (creating 2 different txns or anything else). 
So I don’t think that this decision affects UX, it affects confidentiality and adds complexity.

### 4
G:> when you are using the 32bit blinding factor, are we sure that taking into account that the utxo set is very small, I cannot use the utxo set in order to try all 
the 32bit combinations and brute force? Is 32bit enough to avoid the possibility of brute forcing for, say, GPUs?

A:> Computer can do 23GH/sec of SHA256

M:> we can move to 48bits and have 4bytes/outpoint

----
M:> the thing about the demo, it works with btc mainchain and LN in same way, the difference is which backend you connect (bp node or lnp node). for LN though, it will not be 
a user who would initiate the change, it will be a wallet on the user behalf (it changes the way it should be integrated into the wallet, but it doesn't change the way 
how RGB node internals work).
