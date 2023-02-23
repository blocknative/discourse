# Wallet-Boost
Wallet-Boost helps end-users take control of MEV outcomes and participate in the MEV market. The below design spec serves as an initial draft of our proposed Wallet-Boost solution. Please comment with any and all feedback and questions!


# Intro: Wallet-Boost

With MEV-Boost reaching ~95+% of adoption across the validator ecosystem, the MEV marketplace has reached a new era of MEV-aware design opportunities. The current marketplace for MEV primarily benefits validators and searchers but makes no consideration for the transaction originators (wallet and application end-users). Transaction originators are typically not aware of the background MEV supply chain and its impact on transaction execution.The current market assumes that transaction originators will not participate in MEV. Notably, transaction originators are in a suboptimal position in the MEV supply chain with no ability to control or participate in MEV rewards (or worse, their transaction execution is degraded at the transaction originators expense). To enable transaction originators to express MEV preferences, they need to be both aware and capable of controlling and impacting the MEV supply chain. 

### Problem Statement: the current state of affairs
MEV is an increasingly powerful force on the network – driving massive architectural changes like MEV-Boost and enshrined PBS. But, MEV originates from user transactions and the preferences of transaction originators are not contemplated by the current systems. Instead, they just ‘assume the transaction’ and hope for the best on the final transaction execution.

Wallet-Boost is an open-source project to empower transaction originators to participate in the MEV marketplace. Wallet-Boost aims to make transaction originators MEV-aware and give wallets (and ultimately their end-users) the ability to control MEV outcomes in favor of better transaction execution for the transaction originator. This is net constructive for the ecosystem because it makes MEV less opaque and ensures a level playing field for all network participants – rather than the current state of protocol-level rule sets that implicitly favor specialized/sophisticated actors who can extract value from end-users without their knowledge or consent. Wallet-Boost will be an open, neutral and highly composable standard that:
1. Any member of the current transaction supply chain can participate in equally
2. Is composable and enables wallets and dapps to remain in-control of the end user experience - implementing Wallet Boost will require minimal ux changes
3. Is straightforward to implement for transaction originators (in a similar fashion to how MEV-Boost is implemented by Validators) 
4. Unlocks new novel design opportunities for both wallets and auction designers

### Proposed Approach: high-level considerations

Wallet-Boost serves as a framework, specification and marketplace that brings together wallets, transaction originators, searchers and auction designers. This architecture provides the standard and rule set that wallets can leverage to enable their end-users (transaction originators) to be both MEV-aware and in control of MEV outcomes. Wallet-Boost builds upon the success of MEV-Boost utilizing the builder market place to enhance transaction execution for transaction originators. 

Wallets that choose to participate, run Wallet-Boost as a side-car, similar to that of a validator running MEV-Boost as a side-car. Wallet-Boost gives the wallet a high-degree of configurability to choose the auction (ultimately the tool set that end-users have to control MEV outcomes) as well as other parameters such as a list of included allowed builders and a list of blocked misbehaving searchers.

Searchers will participate in Wallet-Boost in a similar manner as the current builder auctions, except the auction will be to the end-user instead of the builder. Searchers are incentivized to participate honestly or risk being blocked by Wallet-Boost operators forever, thereby losing access to potentially highly valuable order flow. Searchers’ bids will be evaluated based on a transfer payment from the searcher to the user’s address, so searchers will need to incorporate this payment in their signed transaction(s) in the event they win an auction.

End-users will have the same experience they have today, except there will be an additional element of a rebate that an end-user should consider before making the transaction. End-user’s will care more about the swap execution price than the rebate, so the rebate should be thought of as an additional incentive or economic gain on a swap transaction. The rebate comes in the form of an ETH transfer payment from the searcher directly to the user’s address, so the wallet could show this as an extra field or as a gas rebate. There are future explorations to have the end-user receive any arbitrary ERC20 token instead of ETH.

Builders will continue to receive bundles in the same manner they receive bundles today, so there are no major considerations for the builder community. Wallets operating Wallet-Boost choose which builders they send their bundles to, so it is in the best interest of each builder not to censor or take advantage of Wallet-Boost order flow. Otherwise, builders risk being removed from the allow-list of builders and losing access to potentially highly valuable bundle flow.

While initially, Wallet-Boost is considering and serving the MEV landscape as it exists today, meaning the majority of MEV generated is from swaps across liquidity pools, Wallet-Boost will be highly configurable to allow Wallet-Boost operators to ‘plug-in’ any new novel auction mechanism that they see fit. This has the potential to unlock a new actor, auction designers. Auction designers can create any auction mechanism that meets the specification and standards set by Wallet-Boost.

### Design Objectives

Wallet-Boost’s framework and specification allows wallets to control how their users can become MEV-aware and meaningfully participate in the MEV marketplace. The design goals for Wallet-Boost are the following:
1. Highly modular and composable - Wallet-Boost is a neutral framework that has no preference on the auction design or wallet UI/UX. Wallet-Boost can be configured to meet the needs of each individual wallet and their community.
2. Open - Wallet-Boost will be built as an open-source project and be structured in a way that it is open to all wallets, builders, and searchers. However, Wallet-Boost does give wallets the configurability to block misbehaving searchers and the freedom to choose a set of builders to send their bundles.
3. MEV aware - Wallet-Boost enables wallets to empower end-users to understand what is happening in the background MEV supply chain in real-time.
4. Share MEV rewards - Wallet-Boost enables end-users to leverage their MEV-awareness by controlling MEV outcomes and participating in MEV rewards.
5. High execution speed and reliability - Users should not see any noticeable degradation in time required to get their transactions on-chain. 
6. Simple UI/UX - The end-user experience should be the same or better than the current UI/UX from wallets.
7. Censorship Resistant - Censorship resistance is a spectrum. While the protocol should aim for some minimum expected assurance that their transaction will be included on-chain, it should be each wallet-node operator's choice on where in this spectrum they sit to support a geographic diversity of nodes, as well as a mechanism for the community to monitor censorship.

### Mock Wallet-Boost Flow

Many wallets today simulate their user’s transactions before they sign and submit it to the network. The user will construct their desired transaction and the wallet will simulate the transaction against current block state, relaying the exact outcome (ie. net balance changes) of the desired transaction back to the user. The time between constructing the transaction and receiving the simulated result is known as the ‘intent layer’. Wallet-Boost intends to start the auction flow at the intent layer. The primary flow will consider five main events:
1. Triggering a new auction (boost_startAuction)
2. Broadcasting the new auction (boost_publishAuction)
3. Receiving bids for an existing auction (boost_handleAuctionBid)
4. Selecting a winner for an existing auction (boost_handleAuctionWinner)
5. Sending the winning bid and bundle to builders (boost_handleAuctionWinningBid)

The goal of our mock auction is to optimize for multiple variables, mainly:
* Execution speed
* Maximal MEV rebate to transaction originator
* Minimal trust in participating actors

Mock Flow:
1. Our User constructs a swap transaction and receives the simulated result back. Once the simulation is received by the user, Wallet-Boost will call boost_startAuction creating a new auction for Wallet-Boost to track.
2. When a new auction is added to Wallet-Boost, Wallet-Boost takes the unsigned transaction and calls boost_publishAuction which adds it to Wallet-Boost’s unsigned transaction mempool.
3. Searchers can subscribe to events on Wallet-Boost’s unsigned transaction mempool and receive real-time notifications on new auctions.
4. Searchers evaluate unsigned transaction events and bid on any profitable MEV opportunities.
5. Wallet-Boost receives bids and calls boost_handleAuctionBid, which adds the bid to the list of auction bids, and acknowledges receipt to the searcher.
6. When the user signs and submits the transaction through their wallet, Wallet-Boost calls boost_handleAuctionWinner, which stops the auction, chooses the highest bid and requests the signed transaction from the searcher.
7. Wallet-Boost collects the signed transaction from the searcher and calls boost_handleWinningBid, which creates a bundle with the user’s signed transaction and searcher’s signed transaction and sends the bundle to all allow-listed builders.

Mock Sequence Diagram

![](https://i.imgur.com/IJyhVGr.png)


Mock Wallet-Boost API
1. (w *WalletBoostNode) boost_startAuction(tx *unsignedTransaction) error
    * Takes in user’s unsigned transaction
    * Creates a new auction type and adds it to a map of auctions
    * Calls boost_publishAuction
2. (w *WalletBoostNode) boost_publishAuction(tx *unsignedTransaction) error
    * Adds unsignedTransaction to unsigned transaction mempool
    * Broadcasts unsigned transaction to subscribers
3. (w *WalletBoostNode) boost_handleBid(bid *Bid) error
    * Takes in a bid payload
    * Adds it to appropriate auction
    * Responds to searcher with confirmation of receipt
4. (w *WalletBoostNode) boost_handleAuctionWinner() error
    * Stops auction
    * Picks highest bid in auction
    * Requests a signed transaction from winning bidder
5. (w *WalletBoostNode) boost_handleWinningBundle(bid *Bid) error
    * Takes in a winning signed transaction
    * Builds bundle with user’s transaction and searcher’s transaction(s)
    * May need to simulate the winning bundle and take action based on the result of the simulation.
    * Sends the bundle to allow-listed builders

### Key Considerations and Known Issues

At this point there are two main outcomes:
1. Best case: Everything was successful and the winning bundle was included by one of the builders on chain, meaning the user gets their rebate directly in their wallet as an ETH transfer payment from the searcher
2. Worse case: The winning bundle was not included in the next block, which could trigger a number of different possibilities:
    * No action (the bundle can sit with the builders and attempt to be included again)
    * Start a new auction
    * Send user’s transaction through the mempool
    * Send user’s transaction privately
    * Ask user to take a new action

The main considerations and known issues are as follows:
1. Latency
    * The time between receiving the simulation and signing the transaction can vary greatly depending on how long a user takes to sign and submit the transaction. In the instance where this happens too quickly, Wallet-Boost should send the transaction privately or publicly through the mempool to guarantee the same or similar transaction execution speed as current user experience.
2. Trust
    * Current design requires the searcher to send their signed transaction payload to Wallet-Boost and trust that Wallet-Boost will not mishandle the signed payload. Searcher’s are more incentivized to game the system maliciously for profit than a wallet. Wallet’s build their reputation and trust over years with all parties and this would now include searchers. There is no direct economic gain for a Wallet to ‘attack’ a searcher by mishandling their signed transaction. In the future it may be possible to explore schemes which bond searchers to incentivize fair play and decrease the latency requirements of the system.
3. Transaction Execution
    * Even with 95+% market share on block creation, user’s risk worse execution speed through Wallet-Boost if a validator who is not connected to MEV-Boost is chosen for the current slot. Wallet-Boost could check whether a validator is connected to any of the builders on their allow-list and if they are not, send the transaction normally through the mempool.
4. Searcher Considerations
    * Must have low-latency architecture to submit bids and respond with winning signed transaction
    * Will likely receive some ‘spam’ unsigned events where the user gets the simulation, but never signs and submits the transaction.
    * If Wallet-Boost sends the MEV rebate bundle, the searcher would not be credited for the bundle under the current Flashbots’ reputation system.
    * Searchers can subscribe to the unsigned data stream and not participate in the auction, but instead attempt to front-run the user’s potential transactions. This is obviously a probabilistic game since there is no guarantee that the user actually makes that transaction. This is largely mitigated by sending transactions privately to builders as atomic bundles which are typically placed at the top of the block (ahead of any front-running attempt). Since these bundles are atomic, any builder algorithm would see they conflict with each other and could not include both the front-running transaction and the bundle (which would revert).
    * Searchers send signed transactions that fail as a bundle. Wallet-Boost may need to have a reputation system similar to the existing searcher reputation systems from builders to score bundles on profit and reputation.
5. Wallet UI/UX
    * The mock design does not make any consideration for showing the rebate in real-time (ie. as bids come in). There is opportunity to explore different UI/UX improvements under the current flow.

### Future Work

* Different auction mechanisms like a gas fee escalator dutch auction.
* Explore and research mechanisms to remove trust between searcher’s and Wallet-Boost
* Explore the potential to have the reward be in any ERC20 token and not just ETH
* Work with Builders to give ‘credit’ for Wallet-Boost bundles to the searcher under the builder reputation system instead of the wallet. The current flow would have Wallet-Boost sign and submit the bundle instead of the searcher, so the searcher would not receive credit in the searcher-to-builder reputation system.
* ERC-4337 ushers in a new paradigm for user wallet interaction. This new architecture also opens up the aperture of the MEV rebate design space.

### Attribution

* [MEV-Boost](https://github.com/flashbots/mev-boost)
* MEVBoost.org
* [Order flow, auctions and centralisation II: order flow auctions](https://collective.flashbots.net/t/order-flow-auctions-and-centralisation-ii-order-flow-auctions/284)
* [Kolibrio](https://docs.kolibr.io/a01c4d1609ff4143909b250f8badc8dc)
* [Rook](https://docs.rook.fi/reference/rook-protocol/understand-the-protocol)

### Appendix

MEV: History and the Need For Wallet-Boost 

MEV stands for Maximal Extractable Value and it is the value that is extracted by actors who control the ordering of transactions in a block. The value that you can extract from a block depends on how you order the transactions. When there was very little “defi” in 2017-2020, most of the MEV was done on-chain, meaning bots competed with each other in the mempool through a Priority Gas Auction (PGAs). These PGAs meant that bots were flooding the mempool with increasingly higher gas costs than their competitors. This led to high-fees and general network congestion. 

In 2020-2021, Flashbots launched as a research and development organization to mitigate the negative externalities of MEV. Flashbots created tools, namely bundles, allowing searchers to create a private ordered list of transactions with a bribe to incentivize miners (now validators) to include their bundle on-chain. So when a bot saw an opportunity, they no longer had to compete publicly in the mempool against their competitors. Instead, they could grab the mempool transaction,construct their transactions, and then send the ordered list privately to miners for inclusion. Flashbots new auction mechanism increased competition for MEV, moved the competition to a private auction vs. public auction, and incentivized highly gas-efficient smart contracts. Naturally, this relieved the network congestion from PGAs but the majority of MEV went to the miner and searchers saw their margins decrease from the increased competition.

Fast-forward to Ethereum post-merge and we have a brand new ecosystem and MEV supply chain that includes builders, relays, searchers and validators – and no longer miners. Additionally, Flashbots built a new system calledMEV-Boost that allows validators to outsource the construction of new blocks to specialized entities called builders. Builders compete against each other to create the most valuable blocks possible, typically by including as much MEV as they can to maximally incentivize the validator into choosing their block over competitors. See below for the new and current MEV-Boost transaction lifecycle.

![](https://i.imgur.com/RWxpKbl.png)


Today, builders are competing to accept as many bundles from searchers and/or have their own proprietary order flow to make the most profitable blocks possible. For the validator, participating in MEV-Boost is completely optional, however current estimates suggest you double your return as a validator by outsourcing block construction to builders vs. doing it yourself. Currently ~95% of validators are running MEV-Boost with 5 builders making up ~85% of blocks on-chain. 

Today’s MEV supply chain is the following: user’s construct their transaction, sign it through a wallet, broadcast it publicly into the mempool where a searcher picks it up, creates a bundle, sends the bundle to builders, who add it to their block and send it via an MEV-boost relay to the validator to include on-chain. 

![](https://i.imgur.com/FU4OuUH.png)


*Stephane Gosselin: [The MEV Supply Chain: a peak into the future of this industry](https://flashbots.mirror.xyz/bqCakwfQZkMsq63b50vib-nibo5eKai0QuK7m-Dsxpo)*

With the current supply chain, only two actors extract value: the searcher (typically less than 5%) and the validator (typically about 95%). We are starting to see builders extract some value for themselves, but so far it is minimal. Notably, the transaction originator (the one who ‘unlocks’ the MEV in the first place) does not extract any MEV from this supply chain. What if we created a system that recirculated some of that value that is going to the searcher and validator back to the tx originator/user? Enter Wallet-Boost. 

Wallet-Boost is an open-source project to facilitate MEV Value Recirculation to various parties, including transaction originators, wallets, protocols, validators and searchers.

