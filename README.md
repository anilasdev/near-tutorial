
# Create a Voting Contract on NEAR protocol!

## What is NEAR?

NEAR is an open-source and next-generation development platform built on a  [sharded](https://near.org/downloads/Nightshade.pdf),  [proof-of-stake](https://en.wikipedia.org/wiki/Proof_of_stake),  [layer-one](https://blockchain-comparison.com/blockchain-protocols/)  blockchain designed for usability. You can learn more about NEAR on their  [website](https://near.org/).

NEAR is  [fully permissionless, decentralized & community-governed](https://near.org/blog/near-mainnet-phase-2-unrestricted-decentralized/). This means it’s now possible for anyone to create accounts, transfer, and trade NEAR tokens, and build and launch applications on a performant blockchain that prioritizes ease of use for developers and end-users.

## Create an Account

The  [NEAR Wallet](https://wallet.testnet.near.org/)  is a non-custodial, web-based wallet for the NEAR blockchain.

To get started, go to  [https://wallet.testnet.near.org](https://wallet.testnet.near.org/)  and click “Create Account”.

You can find more info on this [here](https://near.org/blog/getting-started-with-the-near-wallet/)

## Prerequisite
[**Nodejs**](https://nodejs.org/en/download/) and [**Rust**](https://www.rust-lang.org/tools/install) must be installed in your machine.

## Project Structure
Find a detailed tutorial about folder structure of every smart contract build in Rust [here](https://docs.near.org/docs/develop/contracts/rust/intro)


## Contract State
Contract state actually holds the data in the blockchain. We need to define the structure of the data struct. This is an important step. Why??? Explained [here](https://www.near-sdk.io/upgrading/prototyping)

Generally whenever the state object is mutated, those transactions should be signed.

## The explorer

Each blockchain has their own explorer (public ledger). You can track all the transactions in the NEAR blockchain [here](https://explorer.testnet.near.org/)

## Let's dive into the core file
``lib.rs``

    use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
    use near_sdk::log;
    use near_sdk::serde::{Deserialize, Serialize};
    use near_sdk::{env, near_bindgen, AccountId};
    use std::collections::HashMap;

    #[derive(Serialize, Deserialize, Clone, BorshDeserialize,BorshSerialize)]
    pub struct Votes {
    pub votes: Vec<AccountId>,
    }

    #[derive(Serialize, Deserialize)]
    #[serde(crate = "near_sdk::serde")]
    pub struct JsonToken {
    pub votes: HashMap<String, Votes>,
    pub winner: String,
    }

    #[near_bindgen]
    #[derive(Serialize, Deserialize, BorshDeserialize, BorshSerialize)]
    pub struct Voting {
    candidate_votes: HashMap<String, Votes>,
    is_poll_open: bool,
    vote_started: u64,
    }
    impl Default for Voting {
    fn default() -> Self {
        let mapped_candidates = HashMap::new();
        Self {
            is_poll_open: true,
            candidate_votes: mapped_candidates,
            vote_started: env::block_timestamp(),
        }
    }
    }
    #[near_bindgen]
    impl Voting {
    pub fn add_candidates(&mut self, candidates: Vec<String>) {
        log!("{}", self.is_poll_open);
        if self.is_poll_open != true {
            env::panic(b"No active polling!");
        }
        for candidate in candidates.iter() {
            self.candidate_votes
                .insert(candidate.clone(), Votes { votes: vec![] });
        }
    }
    pub fn get_candidates(&self) -> Vec<String> {
        if self.is_poll_open != true {
            env::panic(b"No active polling!");
        }
        let candidates = self.candidate_votes.keys().cloned().collect();
        candidates
    }
    pub fn vote(&mut self, candidate: String) -> Votes {
        if !self.is_poll_open {
            env::panic(b"No active polling!");
        }
        let candidate_votes = self.candidate_votes.clone();
        let votes = candidate_votes.get(&candidate.clone());
        if votes.is_some() {
            log!("There is something");
            if votes
                .iter()
                .any(|&i| i.votes.contains(&env::signer_account_id()))
            {
                env::panic(b"You have already voted!");
            } else {
                self.candidate_votes.insert(
                    candidate.clone(),
                    Votes {
                        votes: vec![env::signer_account_id()],
                    },
                );
            }
        } else {
            log!("There is nothing");
        }
        self.candidate_votes.get(&candidate).unwrap().clone()
    }
    pub fn get_stats(&self) -> Option<JsonToken> {
        if !self.is_poll_open {
            env::panic(b"No active polling!");
        }
        let candidate_votes = self.candidate_votes.clone();
        let win = candidate_votes
            .iter()
            .max_by(|a, b| a.1.votes.len().cmp(&b.1.votes.len()))
            .map(|(k, _v)| k);
        log!("{:?}", win.unwrap());
        Some(JsonToken {
            votes: candidate_votes.clone(),
            winner: win.unwrap().clone(),
        })
    }
    pub fn get_results(&self) -> Option<JsonToken> {
        if self.is_poll_open {
            env::panic(b"Polling is active!");
        }
        let candidate_votes = self.candidate_votes.clone();
        let win = candidate_votes
            .iter()
            .max_by(|a, b| a.1.votes.len().cmp(&b.1.votes.len()))
            .map(|(k, _v)| k);
        log!("{:?}", win.unwrap());
        Some(JsonToken {
            votes: candidate_votes.clone(),
            winner: win.unwrap().clone(),
        })
    }
    pub fn end_poll(&mut self) {
        if !self.is_poll_open {
            env::panic(b"Polling is ended!");
        }
        self.is_poll_open = false;
    }
    pub fn start_poll(&mut self) {
        if self.is_poll_open {
            env::panic(b"Polling has already started!");
        }
        self.candidate_votes = HashMap::new();
        self.is_poll_open = true;
        self.vote_started = env::block_timestamp();
    }
    pub fn get_poll_status(&mut self) -> bool {
        self.is_poll_open
    }
    }

# How to build contract?
## Step 1 :
`yarn install` or `npm install`

## Step 2 :
`yarn build` or `npm run build`


## Deploy

The above command with generate a ``.wasm`` file which should be deployed to NEAR blockchain using the following command.
``near deploy --accountId example-contract.testnet --wasmFile out/example.wasm``

# Contract Calls

`near call $CONTRACT_ID  start_poll --accountId=$ACCOUNT_ID`

This method is to start the poll. Without calling this method, you can not call any other methods available.

`near call $CONTRACT_ID  add_candidates '{"candidates" :["Candidate 1", "Candidate 2"]}' --accountId=$ACCOUNT_ID`

This method is to add candidates to the current poll.

`near call $CONTRACT_ID  get_candidates --accountId=$ACCOUNT_ID`

This method returns all candidates. 

`near call $CONTRACT_ID  vote '{"candidate" :"Candidate 1"}' --accountId=$ACCOUNT_ID`

This method is to vote for a valid candidate. 

`near call $CONTRACT_ID  get_stats --accountId=$ACCOUNT_ID`

This method returns some stats and current leading candidate. 

`near call $CONTRACT_ID  end_poll --accountId=$ACCOUNT_ID`

This method ends the on going poll. 

`near call $CONTRACT_ID  get_results --accountId=$ACCOUNT_ID`

This method return the stats and winner of the previous poll.

Explore the [source code](https://gitlab.com/anilkumar1994/near-vote-contract/)

So let's build something on NEAR protocol.