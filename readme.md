# DAML Contracts

These contracts are used for the WaterLedger project. The primary systems are built around Ethereum Smart Contracts written in Solidity. However, these supplementary contracts are for the section of the workflow behind the scenes that is needed to track the liability of payments between parties within the WaterLedger system.

## DAML and Ethereum Hybrid

This secondary system was built as a proof of concept using DAML. There are a number of reasons for this.

- They are related to state workflow changes where DAML excels
- They relate to a created liability, again a strong point for DAML. Ethereum excels at tokenisation, but DAML has rich features for financial instruments 
- The details of the transaction are inherently private, and don’t require public visibility

## Workflow

The flow for this process is to create the `TradeProposal` on the matching of a buyer and seller. This contract is created by a watcher in the WaterLedger API that watches for the Add History event in the Solidity History Smart Contract. 

The implementation of a `TradeProposal` rather than just creating a `NewTrade` directly is due to DAML’s model that a Party cannot have an obligation placed on it without it approving it. Once the `TradeProposal` exists the buyer can exercise the `AcceptTradeIssuance` choice.

Note that the seller does not need to provide any approval as no liability is created on their part. They are listed as an `observer` only, as they have an interest, but they are a beneficiary rather than incurring a liability.

## Two Models

### Minimal implementation

WaterLedger by nature **requires** trustless data storage, there can be no possibility of any occluded or omitted data. This makes DAML a poor requirement fit for the main OrderBook, despite DAML’s **technical** capabilities.

The primary code used here then is `Trade.daml` which is the production code. It features a very “light touch” approach to DAML implementation. Key workflow decisions are made prior to deployment of the contract, and there is little interaction or pipeline. The sole choice that can be made is by the facilitator (WaterLedger’s API) to confirm that a payment was made. In principle this would be triggered by the post back from a payment gateway.

The sole choice, `MakePayment`, archives the liability contract, ending DAML’s involvement.

This code is feature-complete for WaterLedger’s true requirements.

### Fuller Implementation

There was a desire to more deeply investigate the capabilities of DAML from a technical point of view, and more fully implement the trade pipeline within DAML. 

This facilitated a `FullTrade` model, consisting of more parties involved, and more extensive pipeline with branching options. It also does not archive the final step, allowing them to be retrieved by the JSON API, replacing the Trades listing currently written in Solidity. 

**Note that this is explicitly not desired in production. It is a proof of concept.**

Further features will be added over time, including more robust testing, and potentially the entire OrderBook and matching process.


