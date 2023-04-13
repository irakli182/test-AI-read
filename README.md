# Running SKD tests

1. Clone the sdk repository:

```sh
git clone https://github.com/metaplex-foundation/js.git metaplex-sdk
```

2. Checkout the latest release tag

```sh
cd metaplex-sdk
# checkout the latest release tag
tag=$(git describe --tags `git rev-list --tags --max-count=1`)
echo $tag
@metaplex-foundation/js-plugin-aws@0.18.0
git checkout $tag -b latest
```

2. Install the following:

```sh
npm i -g pnpm turbo
```

3. Install project dependencies using `pnpm`:

```sh
pnpm install
```

4. Run all the tests as follows:

```sh
npx turbo run test
```

5. Run single test as follows:

```sh
pnpm run test:filter <TEST_NAME>
```

NB: Release tags can be found here: <https://github.com/metaplex-foundation/js/releases> Currently we are using the this release as of the time of writing `@metaplex-foundation/js@0.18.3`


&nbsp;

---

&nbsp;


# Auction House

Auction House is a program that allows users to exchange assets within the Solana blockchain. The ethos of this program is to allow anyone to create and configure their own marketplace and even provide their own custom logic on how assets should be exchanged. The motivation behind the Auction House protocol is to create a healthy ecosystem of marketplaces that focus on different use-cases, and more importantly, each bringing their own flavor into the way they allow users to trade assets.


&nbsp;

---

&nbsp;


# Managing Auction House

## Create

```js
const auctionHouseSettings = await metaplex
    .auctionHouse()
    .create({
        sellerFeeBasisPoints: 500 // 5% fee
        authority: metaplex.identity(),
        requireSignOff: true,
        canChangeSalePrice: true,
        hasAuctioneer: true, // to enable auctioneer
        auctioneerAuthority: metaplex.identity(),
    });
```

small code example showcasing some of the Auction House attributes.

```js
const { auctionHouse } = await metaplex.auctionHouse().create({...});

auctionHouse.address;                   // The public key of the Auction House account              
auctionHouse.auctionHouseFeeAccount;    // The public key of the Auction House Fee account
auctionHouse.feeWithdrawalDestination;  // The public key of the account to withdraw funds from Auction House fee account
auctionHouse.treasuryMint;              // The mint address of the token to be used as the Auction House currency
auctionHouse.authority;                 // The public key of the Auction House authority
auctionHouse.creator;                   // The public key of the account used to create the Auction House instance
auctionHouse.bump;                      // The `Bump` of the Auction House instance
auctionHouse.feePayerBump;              // The `Bump` of the fee account
auctionHouse.treasuryBump;              // The `Bump` of the treasury account
auctionHouse.auctioneerAddress;         // he public key of the `Auctioneer` account
```

* Auction house bump refers to a practice in online auctions where a seller artificially inflates the bidding price of their item by using a second account to bid on their own auction. This creates the impression of a bidding war and can encourage other buyers to bid higher in order to win the item.

SDK test:
https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/createAuctionHouse.test.ts

smart contract test:
https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/create_auction_house.rs

---

## Fetch Auction House

Once created, the Auction House instance can be fetched. An Auction House can be uniquely identified by its PDA account address or a combination of its creator address and the treasury mint address.

a PDA account address is a unique address associated with a smart contract that governs the auction process for an NFT, and it is used to ensure the integrity and transparency of the auction process. When an NFT is put up for auction on the Metaplex auction house, it is transferred to a PDA account address, which is associated with the auction smart contract. The smart contract then manages the auction process, including verifying bids, managing the auction timeline, and transferring the NFT to the winning bidder at the end of the auction.

An Auction House can be fetched using two ways:

* ```By address```: using the Auction House address
* ```By creator and mint```: using the combination of the creator address and the treasury mint. Note that when the Auction House has Auctioneer enabled, the auctioneerAuthority is also required in addition to the creator and the mint.

```js
// by address
const auctionHouse = await metaplex
    .auctionHouse()
    .findByAddress({ address: new PublicKey("Gjwc...thJS") });

// by creator and mint
// in this example, we assume that the Auction House
// does not have Auctioneer enabled
const auctionHouse = await metaplex
    .auctionHouse()
    .findByCreatorAndMint({
        creator: new PublicKey("Gjwc...thJS"),
        treasuryMint: new PublicKey("DUST...23df")
    });
```

### acutioneer

An auctioneer is a program or smart contract that manages the auction process for a particular NFT. The auctioneer is responsible for verifying bids, managing the auction timeline, and transferring the NFT to the winning bidder at the end of the auction.

The auctioneer is typically implemented as a Program Derived Address (PDA), which is a unique address associated with the auction smart contract code. When an NFT is put up for auction, it is transferred to the PDA, which then manages the auction process using the smart contract code.

The Metaplex auctioneer is designed to be transparent, secure, and decentralized, and it allows for automated and trustless management of the auction process. This reduces the need for intermediaries and improves efficiency, while also ensuring that the auction process is fair and verifiable.

The auctioneer uses a number of different mechanisms to ensure the integrity of the auction process, including:

* Bid Verification: The auctioneer verifies each bid to ensure that it meets the minimum bid requirements and is valid.
* Time Management: The auctioneer manages the auction timeline, including the start and end times for the auction, and ensures that bids are processed in a timely       manner.
* Payment and Delivery: The auctioneer manages the payment and delivery of the NFT to the winning bidder, ensuring that the transaction is secure and verifiable.

---

## Update Settings

once an Auction House instance is created, you can update most of its settings later on as long as you are the authority of the Auction House instance.
The following settings can be updated:

* ```authority```  
    - This feature allows the owner of an NFT to delegate authority to another address or account to manage the auction process for                                          their NFT.
    
* ```sellerFeeBasisPoints```  
    - This feature allows the owner of an NFT (non-fungible token) to set the percentage of the sale price of the NFT that will be charged as a fee by the auction           house. This fee is deducted from the final sale price of the NFT and is paid to the auction house.
    
* ```requiresSignOff```
    - This feature allows the owner of an NFT to control whether or not they need to manually sign off on a transfer of their NFT after an auction has ended.
    
      When requiresSignOff is enabled for an NFT, the winning bidder of the auction will not automatically receive the NFT after the auction has ended. Instead, the         auction house will send a transfer request to the owner of the NFT, and the owner must manually sign off on the transfer before the NFT can be transferred to the       winning bidder.
      
* ```canChangeSalePrice```
    - This feature allows the owner of an NFT to control whether or not the sale price of their NFT can be changed after an auction has started.
    
* ```feeWithdrawalDestination```
    - This feature allows the owner of an auction to control where the fees collected during the auction will be sent..
    
* ```treasuryWithdrawalDestination```
    - This feature allows the owner of the auction house to control where the proceeds from an auction will be sent after the auction has ended.
    
* ```auctioneerScopes```
    - This feature allows the owner of the auction house to control the permissions and access levels of third-party applications that integrate with the auction             house.

To update the settings, we need the full model in order to compare the current data with the provided data. For instance, if you only want to update the feeWithdrawalDestination, you need to send an instruction that updates the data whilst keeping all other properties the same.

Also, by default, feeWithdrawalDestination and the treasuryWithdrawalDestination are set to metaplex.identity(), ie., the same wallet which is set as the authority and the creator by default.

```js
import { Keypair } from "@solana/web3.js";

const currentAuthority = Keypair.generate();
const newAuthority = Keypair.generate();
const newFeeWithdrawalDestination = Keypair.generate();
const newTreasuryWithdrawalDestination = Keypair.generate();
const auctionHouse = await metaplex
    .auctionHouse()
    .findByAddress({...});

const updatedAuctionHouse = await metaplex
    .auctionHouse()
    .update({
        auctionHouse,
        authority: currentAuthority,
        newAuthority: newAuthority.address,
        sellerFeeBasisPoints: 100,
        requiresSignOff: true,
        canChangeSalePrice: true,
        feeWithdrawalDestination: newFeeWithdrawalDestination,
        treasuryWithdrawalDestination: newTreasuryWithdrawalDestination
    });
```

SDK test:
https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/updateAuctionHouse.test.ts

Smart contract Test:
https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/update_auction_house.rs



---

## Withdraw Funds

Funds from accounts can be transferred back to "destination" wallets. These withdrawal destination accounts can be set by the Auction House authority.

```js
// withdraw funds from fee account
await metaplex
    .auctionHouse()
    .withdrawFromFeeAccount({
        auctionHouse,
        amount: 5
    });

// withdraw funds from treasury account
await metaplex
    .auctionHouse()
    .withdrawFromTreasuryAccount({
        auctionHouse,
        amount: 10
    });
```

1. Auction House Fee Wallet to the Fee Withdrawal Destination Wallet.
2. Transfers funds from Auction House Treasury Wallet to the Treasury Withdrawal Destination Wallet.

In both the cases, The Auction House from which the funds are being transferred and the amount of funds to withdrawn need to be specified. This amount can either be in SOL or in the SPL token used by the Auction House as a currency.

* Withdraw funds from fee account:
  - SDK test:
    https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/withdrawFromFeeAccount.test.ts
    
  - smart contract test: 
    https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/withdraw_from_fee.rs


* Withdraw funds from terasury account:
  - SDK test: https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/withdrawFromTreasuryAccount.test.ts
    
  - smart contract test:
    https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/withdraw_from_treasury.rs

&nbsp;

---

&nbsp;


# Trading Assets on Auction House

We talked about Auction Houses and how to create & manage them. Once an Auction House is created, assets can be traded on it. A simple trade on a marketplace usually comprises of three actions:

1. The seller lists an asset
2. The buyer makes a bid on the asset
3. The buyer makes a bid on the asset

We will talk about these three actions and see code examples to easily execute these actions. We will also see trade scenarios that are different from the above simple trade scenario, and go through a code example to execute each scenario. We'll also explore how listings and bids can be cancelled once they are created.

---

## Listing assets

This action is also referred to as creating a Sell Order. When a sell order is created using Auction House, the asset being listed remains in the wallet of the seller. This is a very important feature of Auction House as it allows users to list assets in an ```escrow-less``` fashion and thus users still maintain custody of assets while the assets are listed.

The asset seller can create two types of listings depending on the price at which they list the asset:

1. ```Listing at price greater than 0```: when a user lists an asset at a price which is greater than 0 SOL (or any other SPL-token). In this case, the seller's wallet needs to be the signer and they need to ensure that their wallet has sufficient funds to cover any transaction fees associated with the sale. This may require the user to purchase additional SOL or SPL-tokens in order to cover these fees.

2. ```Listing at price of 0```: when a user lists an asset for 0 SOL (or any other SPL-token). In this case, the authority can sign on behalf of the seller if canChangeSalePrice option is set to ```true```. When this happens, the Auction House finds a non-0 matching bid on behalf of the seller. The asset can only be listed and sold for a price of 0 if the seller acts as the signer. There must be one and only one signer; authority or seller must sign.

Depending on the type of token being listed, the number of tokens to be listed can also be specified when creating a sell order:

1. ```In case of Non-Fungible Tokens (NFTs)```: due to the non-fungibility and uniqueness of every token, only 1 token can be listed.

2. ```In case of Fungible Assets```: the seller can list more than 1 tokens per listing. For example: If Alice owns 5 DUST tokens, they can list 1 or more (but less than or equal to 5) of these DUST tokens in the same sell order.

In the following code snippet we are making a sell order for 3 DUST tokens (fungible tokens) for a total price of 5 SOL. Important to note here is that if we were creating a sell order for an NFT, we do not have to specify the number of tokens to be listed as it will default to 1 token. Specifying any other amount will result in an error.

```js
await metaplex
    .auctionHouse()
    .createListing({
        auctionHouse,                              // A model of the Auction House related to this listing
        seller: Keypair.generate(),                // Creator of a listing
        authority: Keypair.generate(),             // The Auction House authority
        mintAccount: new PublicKey("DUST...23df"), // The mint account to create a listing for, used to find the metadata
        tokenAccount: new PublicKey("soC...87g4"), // The token account address that's associated to the asset a listing created is for 
        price: 5,                                  // The listing price
        tokens: 3                                  // The number of tokens to list, for an NFT listing it must be 1 token
    });
```

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/createListing.test.ts
  

---

## Bidding on assets

A user looking to buy an asset can make bids, or Buy Orders for that asset.

There can be two types of buy orders depending on that whether the asset is listed or not:

1. ```Private bids```: This is the most common type of bid. A user, interested in a listed asset on an Auction House, creates a private bid on the given asset. This bid is tied to the specific auction and not the asset itself. This means that when the auction is closed (either the bid is rejected and the listing is cancelled, or the bid is accepted and the asset is sold), the bid is also closed.

2. ```Public bids```: A user can post a public bid on a non-listed NFT by skipping seller and tokenAccount properties. Public bids are specific to the token itself and not to any specific auction. This means that a bid can stay active beyond the end of an auction and be resolved if it meets the criteria for subsequent auctions of that token.

Like in the case of sell orders, buy orders can also specifiy the number of tokens to be bid upon depending on the type of asset listed:

1. ```Partial Buy Order```: We discussed the case of listing more than 1 token when listing a fungible asset. When such a sell order exists, a user can make a bid to buy only a portion of the listed tokens, or make a partial buy order. For example: if Alice listed 3 DUST tokens for 5 SOL, Alice can make a bid to buy 2 DUST tokens for 2 SOL. In other words, a user can create a buy order of said assets that is less than the token_size of the sell order.

2. ```Complete Buy Order```: This is the case where the buyer creates a bid to buy all the tokens listed in the sell order. In case of non-fungible assets (NFTs) where only 1 token can be listed per sell order, a complete buy order is created. Complete buy orders can also be created in case of fungible tokens.

In the following code snippet we are making a buy order for 3 DUST tokens (fungible tokens) for a total price of 5 SOL. Important to note here is that if we were creating a sell order for an NFT, we do not have to specify the number of tokens to be listed as it will default to 1 token. Specifying any other amount will result in an error.

This is an example of a private bid as we are specifying the seller account and the token account. If either one of them is not specified while creating the bid, the bid will be public.

```js
await metaplex
    .auctionHouse()
    .createBid({
        auctionHouse,                              // A model of the Auction House related to this listing
        buyer: Keypair.generate(),                 // Creator of a bid
        seller: Keypair,generate(),                // The account address that holds the asset a bid created is for, if this or tokenAccount isn't provided, then the bid will be public.
        authority: Keypair.generate(),             // The Auction House authority
        mintAccount: new PublicKey("DUST...23df"), // The mint account to create a bid for
        tokenAccount: new PublicKey("soC...87g4"), // The token account address that's associated to the asset a bid created is for, if this or seller isn't provided, then the bid will be public.
        price: 5,                                  // The buyer's price
        tokens: 3                                  // The number of tokens to bid on, for an NFT bid it must be 1 token
    });
```

SDK test:
https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/createBid.test.ts

---

## Executing sale of assets

Now that we know how to create a listing (sell order) and a bid (buy order), we can learn how to execute sales of assets. When the sale of an asset is executed:

1. The Auction House transfers the bid amount from the buyer escrow account to the seller's wallet.

2. The Auction House transfers the asset from the seller's wallet to the buyer's wallet.


### Trade scenarios.

There are different trade scenarios in which assets can be sold using Auction House:

* ```Direct Buy```
* ```Direct Sell```
* ```Independant Sale Execution```


1. ```Direct Buy```, or "Buying" at list price: This is the case when the execution of the sale happens when a user bids on a listed asset. In other words, a direct buy operation creates a bid on a given asset and then executes a sale on the created bid and listing.

```js
const listing = await metaplex
    .auctionHouse()
    .findListingByReceipt({...}) // we will see how to fetch listings in the coming pages
    
const directBuyResponse = await metaplex
    .auctionHouse()
    .buy({
        auctionHouse,                   // The Auction House in which to create a Bid and execute a Sale
        buyer: Keypair.generate(),      // Creator of a bid, should not be the same as seller who creates a Listing
        authority: Keypair.generate(),  // The Auction House authority, if this is the Signer the
                                        // transaction fee will be paid from the Auction House Fee Account
        listing: listing,               // The Listing that is used in the sale, we only need a
                                        // subset of the `Listing` model but we need enough information
                                        // regarding its settings to know how to execute the sale.
        price: 5,                       // The buyer's price
    });
```

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/directBuy.test.ts
  
* smart contract test: 
  https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/buy.rs
  

2. ```Direct Sell```, or "Selling" at bid price: Counterpart to the case of direct buy, this is the case when a user, interested in an unlisted asset, places a bid on it. If the asset owner now lists the asset for the bid amount, the execution of the sale can take place, thus direct selling the asset.

```js
const bid = await metaplex
    .auctionHouse()
    .findBidByReceipt({...}) // we will see how to fetch bids in the coming pages
    
const directSellResponse = await metaplex
    .auctionHouse()
    .sell({
        auctionHouse,                              // The Auction House in which to create a listing and execute a Sale
        seller: Keypair.generate(),                // Creator of a listing, there must be one and only one signer; Authority or Seller must sign.
        authority: Keypair.generate(),             // The Auction House authority, if this is the Signer the
                                                   // transaction fee will be paid from the Auction House Fee Account
        bid: bid,                                  // The Public Bid that is used in the sale, we only need a
                                                   // subset of the `Bid` model but we need enough information
                                                   // regarding its settings to know how to execute the sale.
        sellerToken: new PublicKey("DUST...23df")  // The Token Account of an asset to sell, public Bid doesn't
                                                   // contain a token, so it must be provided externally via this parameter
    });
```

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/directSell.test.ts
  
* smart contract test:
  https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/sell.rs
  

3. ```Independant Sale Execution```, or Lister agreeing to a bid: This is the case when the execution of the sale takes place independantly, after a ```Buy Order``` (bid) and a ```Sell Order``` (listing) exist for a given asset.

```js
const listing = await metaplex
    .auctionHouse()
    .findListingByReceipt({...}) // we will see how to fetch listings in the coming pages
    
const bid = await metaplex
    .auctionHouse()
    .findBidByReceipt({...})     // we will see how to fetch bids in the coming pages
    
const executeSaleResponse = await metaplex
    .auctionHouse()
    .executeSale({
        auctionHouse,                   // The Auction House in which to create a Bid and execute a Sale
        authority: Keypair.generate(),  // The Auction House authority, if this is the Signer the
                                        // transaction fee will be paid from the Auction House Fee Account
        listing: listing,               // The Listing that is used in the sale, we only need a
                                        // subset of the `Listing` model but we need enough information
                                        // regarding its settings to know how to execute the sale.
        bid: bid,                       // The Public Bid that is used in the sale, we only need a
                                        // subset of the `Bid` model but we need enough information
                                        // regarding its settings to know how to execute the sale.
    });
```

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/executeSale.test.ts
  
* smart contract test: 
  https://github.com/metaplex-foundation/metaplex-program-library/blob/34fe71d267353c6edb48a399a529369c1c1b237a/auction-house/program/tests/execute_sale.rs
  
---

## Cancel Listings and Bids

Once listings and bids are created in an Auction House, they can be cancelled via the authority.

```js
const listing = await metaplex
    .auctionHouse()
    .findListingByReceipt({...}) 
    
const bid = await metaplex
    .auctionHouse()
    .findBidByReceipt({...})    
    
// Cancel a bid
const cancelBidResponse = await metaplex               
    .auctionHouse()
    .cancelBid({
        auctionHouse,            // The Auction House in which to cancel Bid
        bid: bid,                // The Bid to cancel
    });

// Cancel a listing
const cancelListingResponse = await metaplex
    .auctionHouse()
    .cancelListing({
        auctionHouse,            // The Auction House in which to cancel listing
        listing: listing,        // The listing to cancel
    });
```

### Cancel a bidding

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/cancelBid.test.ts


### Cancel a listing

* SDK test:
  https://github.com/metaplex-foundation/js/blob/278b93334d73ca995abd98517c9082752209fd02/packages/js/test/plugins/auctionHouseModule/cancelListing.test.ts
  

&nbsp;

---

&nbsp;


# Managing Buyer Escrow Account

we discussed how to make bids and listings, and execute sales of assets. When we talked about execution of sales, we briefly mentioned the Buyer Escrow Account.

This account is a program derived address (PDA) that acts as an escrow, by temporarily holds the bidder's funds (SOL or SPL-tokens). These funds are equal to the bidding price and are stored in this PDA until the sale goes through. When the sale is executed, the Auction House transfers these funds from the buyer escrow account PDA to the seller's wallet. These funds are managed by the Auction House authority. Let us see how we the authority manages this account.

---

## Getting Balance

Auction House has to make sure that there are enough funds in the buyer escrow account, for the sale to go through.

### snippet that fetches the balance of the buyer escrow account for a given Auction House.

```js
import { Keypair } from "@solana/web3.js";

const buyerBalance = await metaplex
    .auctionHouse()
    .getBuyerBalance({
        auctionHouse,
        buyerAddress: Keypair.generate() // The buyer's address
    });
```

---

At this point, the Auction House knows how much funds are currently there in the buyer escrow account corresponding to a user.
Now if this user makes a bid on an asset, Auction House can decide to transfer funds from the user's wallet to the buyer escrow account in case of insufficient funds.

### snippet of how funds can be transferred from the buyer's wallet to the buyer escrow account for an Auction House.

```js
import { Keypair } from "@solana/web3.js";

const depositResponse = await metaplex
    .auctionHouse()
    .depositToBuyerAccount({
        auctionHouse,              // The Auction House in which escrow
                                   // buyer deposits funds. We only need a subset of
                                   // the `AuctionHouse` model but we need
                                   // enough information regarding its
                                   // settings to know how to deposit funds.
        buyer: metaplex.identity() // The buyer who deposits funds. This expects a Signer
        amount: 10                 // Amount of funds to deposit. This can either
                                   // be in SOL or in the SPL token used by
                                   // the Auction House as a currency.
    });
```

---

## Withdraw Funds

The Auction House should also be able to withdraw funds back from the buyer escrow wallet to the buyer's wallet, in case the user wants their funds back and / or have cancelled their bid.

### snippet of how funds can be withdrawn from the buyer escrow wallet to the buyer's wallet for the given Auction House.

```js
import { Keypair } from "@solana/web3.js";

const withdrawResponse = await metaplex
    .auctionHouse()
    .withdrawFromBuyerAccount({
        auctionHouse,              // The Auction House from which escrow buyer withdraws funds
        buyer: metaplex.identity() // The buyer who withdraws funds
        amount: 10                 // Amount of funds to withdraw. This can either
                                   // be in SOL or in the SPL token used by
                                   // the Auction House as a currency.
    });
```


&nbsp;

---

&nbsp;


# Auction House Receipts

To aid transaction / activity tracking on marketplaces, the Auction House program supports the generation of receipts for listings, bids and sales.
Auction House cancels receipts when the corresponding instruction (bid, listing or sale) is cancelled.

---

## Printing Receipts

To generate receipts, the receipt printing function should be called immediately after the corresponding transaction (```PrintListingReceipt```, ```PrintBidReceipt```, and ```PrintPurchaseReceipt```).

Additionally, the ```CancelListingReceipt``` and ```CancelBidReceipt``` instructions should be called in the case of canceled listings and bids.

There are two fields that can be introduced to each function above to print the corresponding receipt:

1. ```printReceipt```: This is a boolean field that defaults to true. When this field is set to true, a receipt is printed for the corresponding function.

2. ```bookkeeper```: The address of the bookkeeper wallet responsible for the receipt. In other words, the bookeeper is the wallet that paid for the receipt. It's only responsibility at this time is tracking the payer of the receipt so that in the future if the account is allowed to be closed the program knows who should be refunded for the rent. This field defaults to ```metaplex.identity()```.

```js
// printing the ListReceipt
await metaplex
    .auctionHouse()
    .createListing({
        printReceipt: true,
        bookkeeper: metaplex.identity()
    })

// printing the BidReceipt
await metaplex
    .auctionHouse()
    .createBid({
        printReceipt: true,
        bookkeeper: metaplex.identity()
    })

// printing the PurchaseReceipt
await metaplex
    .auctionHouse()
    .executeSale({
        printReceipt: true,
        bookkeeper: metaplex.identity()
    })
```


&nbsp;

---

&nbsp;


# Finding bids, listings and sales

We saw how to make receipts for bids, listings and sales. These receipts make it easier for the marketplace operators to keep track of these actions.
We will see how to fetch these bids, listings and sales.

---

## Find All in an Auction House

There are multiple criteria to find all bids, listings and sales (or purchases) in an Auction House.

### snippet for finding bids by multiple criteria. You can use any combination of keys.

```js
// Find all bids in an Auction House.
const bids = await metaplex
  .auctionHouse()
  .findBids({ auctionHouse });

// Find bids by buyer and mint.
const bids = await metaplex
  .auctionHouse()
  .findBids({ auctionHouse, buyer, mint });

// Find bids by metadata.
const bids = await metaplex
  .auctionHouse()
  .findBids({ auctionHouse, metadata });
```


### snippet for finding listings by multiple criteria. You can use any combination of keys.

```js
// Find all listings in an Auction House.
const listings = await metaplex
  .auctionHouse()
  .findListings({ auctionHouse });

// Find listings by seller and mint.
const listings = await metaplex
  .auctionHouse()
  .findListings({ auctionHouse, seller, mint });
```


###  snippet for finding purchases by multiple criteria. It supports only 3 criteria at the same time including the required auctionHouse attribute.

```js
// Find all purchases in an Auction House.
const purchases = await metaplex
  .auctionHouse()
  .findPurchases({ auctionHouse });

// Find purchases by buyer and mint.
const purchases = await metaplex
  .auctionHouse()
  .findPurchases({ auctionHouse, buyer, mint });

// Find purchases by metadata.
const purchases = await metaplex
  .auctionHouse()
  .findPurchases({ auctionHouse, metadata });

// Find purchases by seller and buyer.
const purchases = await metaplex
  .auctionHouse()
  .findPurchases({ auctionHouse, seller, buyer });
```

---

## Find by Receipt

###  the snippet for finding bids, listings and sales by corresponding receipt acccount address.

```js
// Find a bid by receipt
const nft = await metaplex
  .auctionHouse()
  .findBidByReceipt({ receiptAddress, auctionHouse };

// Find a listing by receipt
const nft = await metaplex
  .auctionHouse()
  .findListingByReceipt({ receiptAddress, auctionHouse };

// Find a sale / purchase by receipt
const nft = await metaplex
  .auctionHouse()
  .findPurchaseByReceipt({ receiptAddress, auctionHouse };
```

---

## Find by Trade State

### snippet for finding bids, listings and sales by corresponding trade state PDA accounts.

```js
// Find a bid by trade state
const nft = await metaplex
  .auctionHouse()
  .findBidByTradeState({ tradeStateAddress, auctionHouse };

// Find a listing by trade state
const nft = await metaplex
  .auctionHouse()
  .findListingByTradeState({ tradeStateAddress, auctionHouse };

// Find a sale / purchase by trade state
const nft = await metaplex
  .auctionHouse()
  .findPurchaseByTradeState({ sellerTradeState, buyerTradeState, auctionHouse };
```
 
 
 &nbsp;
 
 ---
 
 &nbsp;
 
 
 # Certified Collections
 
 ## Introduction 
 
 Certified Collections enables NFTs – and tokens in general — to be grouped together and for that information to be verified on-chain. Additionally, it makes it easier to manage these collections by allocating data for them on-chain.
 
This feature provides the following advantages:

* Easy to identify which collection any given NFT belongs to without making additional on-chain calls.
* Possible to find all NFTs that belong to a given collection
* Easy to manage the collection metadata such as its name, description and image.
 
---

## Collection NFTs

In order to group NFTs — or any token — together, we must first create a Collection NFT whose purpose is to store any metadata related to that collection. It has the same data layout on-chain as any other NFT

The difference between a Collection NFT and a Regular NFT is that the information provided by the former will be used to define the group of NFTs it contains whereas the latter will be used to define the NFT itself.

---

## Linking Regular NFTs to Collection NFTs.

Collection NFTs and Regular NFTs are linked together using a "Belong To" relationship on the Metadata account. The optional Collection field on the Metadata account has been created for that purpose.

* If the Collection field is set to None, it means the NFT is not part of a collection.
* If the Collection field is set, it means the NFT is part of the collection specified within that field.

As such, the Collection field contains two nested fields:

* Key: This field points to the Collection NFT the NFT belongs to. More precisely, it points to the public key of the Mint Account of the Collection NFT. This Mint Account must be owned by the SPL Token program.
* Verified: This boolean is very important as it is used to verify that the NFT is truly part of the collection it points to. More on that below.

---

## Differentiating Regular NFTs from Collection NFTs.

The ```Collection``` field alone allows us to link NFTs and Collections together but it doesn't help us identify if a given NFT is a Regular NFT or a Collection NFT. That's why the CollectionDetails field was created. It provides additional context on Collection NFTs and differentiate them from Regular NFTs.

* If the ```CollectionDetails``` field is set to ```None```, it means the NFT is a Regular NFT.
* If the ```CollectionDetails``` field is ```set```, it means the NFT is a Collection NFT and additional attributes can be found inside this field.

 
The ```CollectionDetails``` is an optional enum which currently contains only one option V1. This option is a struct that contains the following field:

* ```Size```: The size of the collection, i.e. the number of NFTs that are directly linked to this Collection NFT. This number is automatically computed by the Token Metadata program but can also be set manually to facilitate the migration process.

---

## Nested Collections

Because ```Collections``` and NFTs are linked together via a "Belong To" relationship, it is possible by design to define nested collections. In this scenario, the ```Collection``` and ```CollectionDetails``` fields can be used together to differentiate Root and Nested Collection NFTs.

---

## Verifying NFTs in Collections

As mentioned above, the ```Collection``` field contains a ```Verified``` boolean which is used to determine if the NFT is truly part of the collection it points to. Without this field, anyone could pretend their NFT to be part of any collection.

In order to flip that ```Verified``` boolean to ```True```, the Authority of the Collection NFT must sign the NFT to prove that it is allowed to be part of the collection.


### The following instructions are available to set, verify or unverify an NFT as part of a sized collection:

* [Verify a sized collection item](https://docs.metaplex.com/programs/token-metadata/instructions#verify-a-sized-collection-item) 
* [Unverify a sized collection item](https://docs.metaplex.com/programs/token-metadata/instructions#unverify-a-sized-collection-item)
* [Set and verify a sized collection item](https://docs.metaplex.com/programs/token-metadata/instructions#set-and-verify-a-sized-collection-item)

If your Collection NFT does not yet have its ```CollectionDetails``` field setup, you must use the following instructions instead:

* [Verify a collection item](https://docs.metaplex.com/programs/token-metadata/instructions#verify-a-collection-item)
* [Unverify a collection item](https://docs.metaplex.com/programs/token-metadata/instructions#unverify-a-collection-item)
* [Set and verify a collection item](https://docs.metaplex.com/programs/token-metadata/instructions#set-and-verify-a-collection-item)

---

## Delegating the Collection Authority

By default, only the Update Authority of the Collection NFT can verify that an NFT is part of that collection.

However, the Update Authority can also delegate this responsibility to other authorities. This allows us to delegate the ability to add NFTs to our collection to one or several trusted parties. These delegated Collection Authorities can then set, verify and/or unverify NFTs from this collection using the instructions listed in the previous section.

The following instructions enable us to approve and reject a Collection Authority:

* [Approve a new Collection Authority](https://docs.metaplex.com/programs/token-metadata/instructions#approve-a-new-collection-authority)
* [Revoke an existing Collection Authority](https://docs.metaplex.com/programs/token-metadata/instructions#revoke-an-existing-collection-authority)

---

## Set and verify a collection using collections.metaplex.com

Metaplex provides a helpful [web tool](https://collections.metaplex.com/) that allows us to create Collection NFTs and add verified NFTs to them.

You may use the following step to get started with that tool:

1. Visit collections.metaplex.com
2. Connect your wallet, but first, verify this wallet is the Update Authority.
3. Select the cluster you want to work on — e.g. devnet, mainnet, etc.
4. Click on "Create a Collection".
5. Enter the name, symbol, logo, and description of your Collection NFT.
6. Choose between these three options:
   - Individual NFTs: Insert the mint address of your NFTs.
   - First verified creator: Insert the public key of the first creator defined in your NFTs. This can help with Candy Machines as the first creator address is derived      from their public key.
   - CSV file: Upload a CSV file that contains the list of mint addresses. The CSV file should contain all the public keys, separated by commas with no spaces.
7. Click "Create Collection" and two transaction approvals will be required
   - The first transaction approval will allow Metaplex to be the delegate to make the migration.
   - The second transaction will create the parent Collection NFT.
8. After the parent Collection NFT is created, the migration will start in the background. You may close the tab and come back to it later with the same wallet to see    the status.


video tutorial on how Verified Collections work and how to use the web tool mentioned above. You can watch it [here](https://drive.google.com/file/d/1VU4xL_yF6LCe0UogVn4As5PMAzUV__8C/view):





