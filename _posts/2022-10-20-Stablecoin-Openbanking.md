---
layout: post
title: 'OpenBanking+ERC-20=❤️: An automated stablecoin'
author: stelios
tags: [payments, openbanking, pisp, blockchain, hands-on]
categories: [Blockchain, Ethereum, Payments]
featured: true
description: "."
image: /assets/images/ethereum-stablecoin/coinwire-japan-iNOavZh6dQ8-unsplash.jpg
---

In 

# Stablecoins and why we need them

Stablecoins are central to the crypto ecosystem.
First let's look at the **What** stablecoins are and then examine the **Why** we need them.

## Types of stablecoins

As their name suggests, stablecoins offer *stability* of exchange value against another asset or currency. Say 1 
stablecoin = 1 USD. 

This stability can be achieved in a number of ways, which give us the different [categories][1].

* **"True" Stablecoins**    
These are non-interest bearing coins. Stability is achieved by the issuer through 2 "pillars" 
  * The issuer mints and buys back coins at par 
  * They hold assets to back their obligation to redeem the outstanding stablecoins. These assets need to be denominated 
    in the currency of the reference asset (to avoid currency risk), be highly liquid, and be "safe" (i.e. low risk of 
    losses in negative market conditions).  
Effectively, true stablecoins are a form of "narrow banks". They should hold their "coin holder reserves" separate and 
  highly liquid, so that in a bankruptcy coin holders can be made whole.
* **Deposit Stablecoins**    
These are deposit claims, provable on blockchain, held in insured commercial banks. Holders are protected by the legal 
framework governing bank deposits (e.g. capital requirements, deposit insurance,...). In this regard, deposit coins 
preserve the current system of banking and payments, removing legacy friction. In theory (since there is no clear legal 
framework atm), deposit coins issued by one institution, might not be recognised by others. 
* **Decentralised Stablecoins Backed By Crypto**    
These stablecoins do not have a central operator, but are governed by a consensus of the users who take part in the network.
Coins in circulation are backed by crypto-assets (e.g Dai LINK is backed by a crypto-basket). Users lock up a certain 
amount of cryptocurrencies as collateral for borrowing DAI. Dai maintains its peg to the US Dollar, through a mechanism 
of forced collateral liquidations, 
* **Decentralised Algorithmic Stablecoins**  
These coins don’t have any collateral backing their system, and they rely on algorithms and market incentives to have 
their remain stable against a fiat currency. Though they appear elegant in theory, it is questionable how their underlying 
primitives will work in extreme conditions. This was highlighted in a spectacular way by the Terra UST collapse LINK.  

## Use of stablecoins

If you have read my series on how global payments work
https://sgerogia.github.io/Payments-Intro-Part1/
https://sgerogia.github.io/Payments-Intro-Part2/
https://sgerogia.github.io/Payments-Intro-Part3/

the world of international fiat payments is still underpinned by a network for nostro-vostro accounts with slow, 
multi-day settlement cycles

In the 24/7/365 world of near real-time blockchain settlement, this will just not cut it

Stablecoins are an IOU 
real, in the case of true & backed s/c
conceptual, in the case of crypto-backed and algorithmic s/c
Digital unit of account
to massively accelerate the velocity of fiat money to match that of native digital assets  

# Intro to OpenBanking

Have gone into great-length in my 2-part series
https://sgerogia.github.io/OpenBanking-Part1/
https://sgerogia.github.io/OpenBanking-Part2/

<insert diagram here>

# Marrying OpenBanking & web3

![Marriage](../assets/images/ethereum-stablecoin/foto-pettine-IfjHaIoAoqE-unsplash.jpg)
> Photo by Foto Pettine on Unsplash

## Problem 
USDT and to some extent USDC belong to the true stablecoin category  
* On-ramp only for large investors ($100k deposit minimum)
* Off-ramp for accredited people (purely discretionary)
https://cryptoslate.com/you-can-redeem-tether-usdt-11-on-tether-to-but-theres-a-catch/
* Most important Asset audit and verification is manual and constant source of FUD w.r.t quality of assets 

An ideal solution would automate away the need for manual processing and auditing 
note: In true narrow bank form part of USDT/C's revenue stream is arbitraging between offering zero yield (for depositors)
and receiving interest from converting USD balances into interest-bearing instruments (e.g. bonds) 
This would make the process of validating balances self-evident on-chain


## Challenges

OpenBanking is built on top of OAuth2, which has a fundamental assumption: end-to-end secure channel
This is crucial as the OB messages carry sensitive information
note: One example is bank account details which is PII and could be used in banking fraud
Another example is customer consent authorisation links (link needed) and consent auth. tokens 
Though not explicitly documented, in most banks' cases these are single use. A malicious actor intercepting them could 
just make a failed attempt just to "consume" them and invalidate an otherwise valid flow

This comes at a stark contrast with blockchains making all information visible to everyone by default

We will need to consider how to use available primitives to address these shortfalls 

## Solution 

Note: this is public key authenticated encryption first introduced in NaCl https://nacl.cr.yp.to/valid.html
Already supported by Metamask on the browser-side LINK and by a wide array of languages (Python, Golang,...) on the 
server-side  

![OpenBanking Stablecoin](../assets/images/ethereum-stablecoin/ob-stablecoin.png)
> The minting flow of our OpenBanking dApp

<Describe the steps and outline>

TPP derives public encryption key and puts it in published contract
This could derive from the TPP's chain private key or, even better, have a separate encryption key pair (e.g. located 
in an HSM)


mintRequest: Send public key in the payload
{
institutionId: institutionId,
sortCode: sortCode,
accountNumber: accountNumber,
name: name,
publicKey: myPubKey
}

Receive MintRequest: Use private key to decrypt, store user's public key for all subsequent user-bound encrypted 
payloads (e.g. authRequest) 




# Let's get coding

![Some coding-y image](../assets/images/ethereum-stablecoin/james-harrison-UVMPVIRCF5w-unsplash.jpg)
> Photo by James Harrison on Unsplash

> The code for this blog post is in repository [hello-stablecoin][].
> Each section below has a corresponding code branch (`v1`, `v2` etc) with the progress of the project until that point. 
> You can switch to that branch and follow along at your pace.

## v1 - Setup and smart contract minting

Going to have everything in a monorepo
Everything related to smart contract will be in sub-folder `/chain`

First installed right Node version.
Then initial project skeleton was created using Hardhat (link)
In your case you can jusat download all dependencies with 
```
nvm install 18.7.0
nvm use 18.7.0
npm install
```

Created an initial config file (https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/hardhat.config.ts) by scavenging from our [previous project] https://github.com/sgerogia/hello-chainlink/blob/main/hardhat.config.js

created the outline of our smart contract using OpenZeppelin's wizard https://docs.openzeppelin.com/contracts/4.x/wizard
Note: Our contract is ERC-20, mintable & burnable (the whole point of this exercise!), pausable (in case of emergency) and ownable (as only the TPP should have full control)

Our additions to the contract are 
* Disable the public mint method (https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L77)
* Expose the TPP's public key for clients to use for encryption https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L25
* Create a map of mint commitments (https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L35)
* Define the payload of the different events that the contract will emit (https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L42-L60)
 Notice that we have made some fields `indexed` to aid with event retrieval
* Define the logic and controls of the 4 smart contract methods (mintRequest https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L95)
 authRequest https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L113
 authGranted https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L132
 paymentComplete https://github.com/sgerogia/hello-stablecoin-2/blob/v1/chain/contracts/ProvableGBP.sol#L147

We can now compile out contract to check for errors `make compile-contract`

## v2 - Contract unit tests 

Unit tests at this stage provide some basic coverage 
are split in 3 groups
* basic functionality, like contract name and pausing (https://github.com/sgerogia/hello-stablecoin-2/blob/v2/chain/test/ProvableGBP_base_test.ts)
* end-to-end logical flow (method and event payload) (https://github.com/sgerogia/hello-stablecoin-2/blob/v2/chain/test/ProvableGBP_e2e_flow_test.ts)
* encryption/decryption, simulating the behaviour of the user's wallet and TPP's process (https://github.com/sgerogia/hello-stablecoin-2/blob/v2/chain/test/ProvableGBP_mintRequest_test.ts)

For encryption/decryption we need to work with what is supported by the Metamask wallet. This is what our end users 
will use once we have our UI in-place. 
We will use the [eth-sig-util library] https://metamask.github.io/eth-sig-util/index.html  
Under the hood it supports `Curve25519, XSalsa20 and Poly1305` https://github.com/MetaMask/eth-sig-util/blob/v5.0.0/src/encryption.ts#L39 as defined in NaCl


Reader can extend the unit test suite as an exercise to cover edge cases etc 

## v3 - Manual testing 

### Hardhat tasks 

To facilitate smoke testing in a remote chain, we can utilize Hardhat's task plugin mechanism (https://hardhat.org/hardhat-runner/docs/advanced/create-task)

We have created a number of tasks to allow us to emulate the steps of the cycle (plus a few more generic ones)
For convenience they have a numeric prefix to their name so that they appear first in the Hardhat list as well as indicate
the correct order of calling

They are created in pairs
one to call the smart contract method (e.g. `mintRequest`) 
one to receive the event on the other side and decrypt its payload (e.g. `getMintRequest`)

You may notice that not all actions in the cycle have been implemented. 
This could be a nice exercise for the reader

### Ganache deployment 

We will use Ganache as a convenient local Ethereum chain 

We create a new workspace (i.e. chain instance), making sure the settings match our Harhdat config file.

![Ganache settings](../assets/images/ethereum-stablecoin/ganache-settings.png)

We pick 2 of the auto-created accounts and take note of their private keys and account ids.
One as the smart contract owner (TPP) and one as the user (bank account holder)

![Ganache account keys](../assets/images/ethereum-stablecoin/ganache-keys.png)

Take a copy of the account private keys, we will use them below prefixed with `0x`. 
I.e. if the Ganache private key is `123abc`, we will be using `0x123abc`.

With Ganache running, we deploy the smart contract  
```bash
PRIVATE_KEY=0x<TPP_PRIVATE_KEY> \
npx hardhat deploy --network ganache --tags gbp
```

The `tags` switch helps us select the deployment script(s) from the `deploy` folder (link in our case only one)

We can verify the contract deployment in Ganache.  
![Ganache contract deployment](../assets/images/ethereum-stablecoin/ganache-deploy-trx.png)

Take note of the contract address, as we will need it for our...

### End-to-end testing

We can now use the tasks we have created to emulate the flow between the 2 accounts: user and TPP.

Start by having the user make a `mintRequest`    
```bash
PRIVATE_KEY=0x<USER_PRIVATE_KEY> \
npx hardhat 1-mint-request \
    --contract <CONTRACT_ADDRESS> \
    --network ganache \
    --amount 100 \
    --institution-id test-bank \
    --sort-code 405060 \
    --account-number 1234567 \
    --name "John Doe"
```

The TPP's Ethereum client process needs to receive the message. 
Since we have not developed it yet, we need to do it manually. The `getMintRequest` action prints and decrypts the message.  
To demonstrate the decrypted values, the command prints an example consent creation call for Natwest's sandbox bank.
note: For more details, on how to set up and use Natwest's sandbox take a look at the blog post LINK 
```bash
PRIVATE_KEY=0x<TPP_PRIVATE_KEY> \
npx hardhat 2-get-mint-request \
    --contract <CONTRACT_ADDRESS> \
    --network ganache \
    --account <USER_ACCOUNT>
```

The command prints out the message contents, after decrypting the payload with the `TPP_PRIVATE_KEY`.  
```text
Listening for MintRequest for PGBP (contract 0x... ) on network ganache
MintRequest received
        RequestId 0xbff95a362c6b6c2577c5a0dd76ac3f645b2fdda2bcacd5c3fc351916a4d44954
...
        Encr. data {"version":"x25519-xsalsa20-poly1305","nonce":"...","ephemPublicKey":"...","ciphertext":"..."}
--------------------------------------------------
MintRequest encr. payload
...
        Public key CgTZFRFIToxlhoxMkexzbhNiTgSkIkqeI2egzp0kHVA=
--------------------------------------------------
```
It is important here to note how the encrypted payload is decrypted using the receiver's (TPP) private key.
Inside it contains the user's encryption public key. 
The TPP must use this for all subsequent user-bound messages.

You can continue the flow with the other 2 implemented tasks.

Sending the authorisation request from the TPP to the user.   
```bash
PRIVATE_KEY=0x<TPP_PRIVATE_KEY> \
npx hardhat 3-auth-request \
    --contract <CONTRACT_ADDRESS> \
    --network ganache \
    --request-id <ORIGINAL_REQUEST_ID> \
    --public-key <PUBLIC_KEY_FROM_MINT_REQUEST> \
    --url "https://bank.com/some-url/to/authorise/consent/"
```

...and receiving it on the other side.  
```bash
PRIVATE_KEY=0x<USER_PRIVATE_KEY> \
npx hardhat 4-get-auth-request \
    --contract <CONTRACT_ADDRESS> \
    --network ganache \
    --account <USER_ACCOUNT>
```

It will be an excellent exercise for the reader to continue implementing the remaining missing tasks
5-auth-granted
6-get-auth-granted
7-payment-complete 
and verify the increase in total supply (i.e. tokens being minted) with `npx hardhat total-supply`


## Brief pause - Sandbox bank accounts

Will use the Natwest sandbox portal 
to have better control and overview of the money flows 
A detailed overview of the sandbox and its basic concepts can be found here https://sgerogia.github.io/OpenBanking-Part2/
If you are not familiar it would be very helpful to take some time to read the basic concepts

To keep things simple, we will use basic security (username & password) in our application
![Create sandbox app](../assets/images/ethereum-stablecoin/create-sandbox-app.png)

Scroll down to find the `Payment Initiation 3.1.7` API and configure it to have *<reduced> security* and *programmatic 
approval*. 
Both of these will make testing easier (but, obviously, are not available in a production environment).

![Reduced security](../assets/images/ethereum-stablecoin/reduced-security.png)

You can use the auto-provisioned data or you can wipe them out and upload the attached test data file. 
It contains only 2 users with accounts: *John Doe* (payer) and *Provable GBP Limited* (receiver)

At this point you can manually verify the end-to-end setup of the sandbox account or skip to the next section. 

The following expanding box contains the cURL commands to test money movements.

<details markdown="1">
  <summary>Click to expand!</summary>

Replace `CLIENT_ID` & `CLIENT_SECRET` in the commands below from the newly created sandbox application.

**Client access token**  
```bash
curl -k \
-X POST \
https://ob.sandbox.natwest.com/token \
-H 'Content-Type: application/x-www-form-urlencoded' \
-d 'grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&scope=payments' \
| jq '.access_token'
```

**Create a consent**  
Replace the `CLIENT_ACCESS_TOKEN` below with the value.  
```bash
curl -k \
  -X POST \
  https://ob.sandbox.natwest.com/open-banking/v3.1/pisp/domestic-payment-consents \
  -H 'Authorization: Bearer CLIENT_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-jws-signature: IGNORED_DUE_TO_REDUCED_SECURITY' \
  -H 'x-idempotency-key: SOME_UNIQUE_IDEMPOTENCY_KEY' \
  -d '{
      "Data": {
        "Initiation": {
          "InstructionIdentification": "instr-identification",
          "EndToEndIdentification": "e2e-identification",
          "DebtorAccount": {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "50000012345601",
            "Name": "John Doe"
          },
          "InstructedAmount": {
            "Amount": "1000",
            "Currency": "GBP"
          },
          "CreditorAccount": {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "50000087654301",
            "Name": "ProvableGBP Limited"
          },
          "RemittanceInformation": {
            "Unstructured": "Provable GBP mint",
            "Reference": "Provable GBP mint"
          }
        }
      },
      "Risk": {
        "PaymentContextCode": "Services",
        "MerchantCategoryCode": null,
        "MerchantCustomerIdentification": null,
        "DeliveryAddress": null
      }
    }' \
    | jq '.Data.ConsentId'
```

**Generate authorisation URL**  
Paste the value in the `CONSENT_ID` below.  
`REDIRECT_URI` is as you have defined it in the sandbox application definition.  
```bash
curl -v -G -k \
  "https://api.sandbox.natwest.com/authorize" \
   --data-urlencode "client_id=CLIENT_ID" \
   --data-urlencode "response_type=code id_token" \
   --data-urlencode "scope=openid payments" \
   --data-urlencode "redirect_uri=REDIRECT_URI" \
   --data-urlencode "request=CONSENT_ID"
```

**Complete authorisation**  
Copy the `location` and paste in a browser.  
Complete the consent authorisation, using the login information of *John Doe* (`customerNumber` etc)

![Ebanking login](../assets/images/ethereum-stablecoin/ebanking-login.png)

![Ebanking 4P](../assets/images/ethereum-stablecoin/ebanking-4p.png)

![Approve consent](../assets/images/ethereum-stablecoin/approve-consent.png)

Once complete, copy the `code` URL parameter.  

![Code URL param](../assets/images/ethereum-stablecoin/code-url-param.png)

**Exchange code for access token**  
Replace `CLIENT_ID`, `REDIRECT_URI` and `AUTH_CODE_FROM_URL` as necessary.  
```bash
curl -G -k \
  -X POST \
  https://ob.sandbox.natwest.com/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'client_id=CLIENT_ID&client_id=CLIENT_SECRET&redirect_uri=REDIRECT_URI&grant_type=authorization_code&code=AUTH_CODE_FROM_URL' \
  | jq '.access_token'
```

**Make the payment**  
Replace the `ACCESS_TOKEN` and `CONSENT_ID` from above.  
```bash
curl -k \
  -X POST \
  https://api.sandbox.natwest.com/open-banking/v3.1/pisp/domestic-payments \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-jws-signature: IGNORED_DUE_TO_REDUCED_SECURITY' \
  -H 'x-idempotency-key: SOME_UNIQUE_IDEMPOTENCY_KEY' \
  -d '{
      "Data": {
        "ConsentId": "CONSENT_ID",
        "Initiation": {
          "InstructionIdentification": "instr-identification",
          "EndToEndIdentification": "e2e-identification",
          "DebtorAccount": {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "50000012345601",
            "Name": "John Doe"
          },
          "InstructedAmount": {
            "Amount": "1000",
            "Currency": "GBP"
          },
          "CreditorAccount": {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "50000087654301",
            "Name": "ProvableGBP Limited"
          },
          "RemittanceInformation": {
            "Unstructured": "Provable GBP mint",
            "Reference": "Provable GBP mint"
          }
        }
      },
      "Risk": {
        "PaymentContextCode": "Services",
        "MerchantCategoryCode": null,
        "MerchantCustomerIdentification": null,
        "DeliveryAddress": null
      }
    }' \
    | jq '.Data.DomesticPaymentId'
```

**Verify payment settlement (optional)**  
Replace the `PAYMENT_ID` and `ACCESS_TOKEN` from above.  
```bash
curl -G -k \
"https://api.sandbox.natwest.com/open-banking/v3.1/pisp/domestic-payments/PAYMENT_ID" \
-H 'Authorization: Bearer ACCESS_TOKEN' \
| jq '.Data.Status'
```

You can also login to the sandbox E-Banking as "ProvableGBP Limited" (Bankline customer id `1234567890`) and verify that 
the transaction took place and the business account balance has increased (i.e. we can mint new tokens).

![Access e-banking](../assets/images/ethereum-stablecoin/access-ebanking.png)

![Bankline credentials](../assets/images/ethereum-stablecoin/bankline-creds.png)

![Bankline transactions](../assets/images/ethereum-stablecoin/bankline-trx.png)

</details>

## v4 - TPP client

Create a new folder to hold our Go code  
```bash
mkdir tpp-client && cd tpp-client
go mod init github.com/sgerogia/hello-stablecoin/tpp-client
```

We will use the go-resty library 
`go get github.com/go-resty/resty/v2`

and create an abstract TPP client interface and a concrete Natwest client implementation.

End-to-end integration test verifies correct functionality of the client
The integration test has hard-coded account numbers from our test data set
Here we take advantage of the programmatic approval consent offered by the Natwest sandbox

## v5 - ETH client 

`go get github.com/ethereum/go-ethereum`

Then generate our Go code bindings from the compiled contract's ABI.
```bash
cat ../chain/artifacts/contracts/ProvableGBP.sol/ProvableGBP.json \
   | jq '.abi' \
   | abigen --abi - \
      --type ProvableGBP \
      --pkg contract \
      --out contract/provable_gbp.go \
      --bin ../chain/artifacts/contracts/ProvableGBP.bin
```


We are finally ready for an...

## End-to-end test

Let's build the binaries and run the local server.  
```bash
make build-cli
make run-local-http
```

Goa has generated a full-fledged API client out-of-the-box. Each API service (`token`, `math`) has become a separate 
CLI sub-command. You can see its usage with  
```bash
./server-cli token --help
./server-cli math --help
./server-cli math mul --help
...
```

Let's get a token with the CLI.    
> In this section we also show the equivalent cURL side-by-side.

```bash
./server-cli token auth --body '{ "password": "password", "username": "user" }'

curl -X POST http://localhost:8080/auth \
-H 'Content-Type: application/json' \
--data-binary '{ "password": "password", "username": "user" }'   
```

Let's try to multiply some numbers.   
```bash
./server-cli math mul \
  --numbers '["4", "3.543", "-2"]' \
  --token "TOKEN_FROM_THE_PREVIOUS_STEP"
  
curl http://localhost:8080/mul/4,3.543,-2 \
-H 'Authorization: Bearer TOKEN_FROM_THE_PREVIOUS_STEP'  
```

Finally, let's submit a JSON document for addition of its numeric elements.  
```bash
./server-cli math sum \
  --token "TOKEN_FROM_THE_PREVIOUS_STEP" \
  --body '{"a":{"b":4},"c":2, "d":[1, 3]}' 

curl -X POST http://localhost:8080/sum \
-H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJ1c2VyIiwiZXhwIjoxNjY0ODgwNTY4LCJqdGkiOiJmZDQ3Zjc2MC0xN2Q3LTRlNDYtYjdjZi0yMWM1MzBjNzc0NzgiLCJpYXQiOjE2NjQ4NzY5NjgsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCIsInN1YiI6InVzZXIifQ.efisAM2XQdbmuNPRoYffYewH636CNgCLNh4tecnkXSGygfPvNrQhXpkeM3zA731j2VIIJqss8NeDcXPxQFbwHDdcxqmt5w0b-onNuGCFf2u7W55rWANNOCMjke5B4QSCop9waVV-eXSF70yIXPT5iNKD7SIlOv4FrNzkvNye5w4VCg7g-oZovjsZdmaLN2SfLdzyxXTBLw4TCst5SFiVzzcyhPMOevnX5mSv6p0uI_iPCO0GhdLyW20-HghazSRDI6xoj5vepuoP5_fCPdpwZUhsO75o_pl66IENhBmiovEsvpbEV5qYc0sXVKH4yk6Jcis6OCbHL3gQSVB97Sc4xTSmGkEkGWvbdyf_j4uBKRE8SVMZd93EqDJkze5Os7umKP5Nw8ws4JLuoyWab-kM1wNwTJmGNFWuhC_Tfql4blDKFZPIZOF-Yqqj67QP5f1nQ9pW1GNerbiPhTQ4noUNRtyokruyFjBhWql_0ebYf94xesvJMIa93GnVoIlSuuX9' \
--data-binary '"{\"a\":{\"b\":4},\"c\":2, \"d\":[1, 3]}"'
```

This last example allows us to see how the CLI client might "sweep under the rug" some Goa internals.  
In this case the data-type of the [`/sum` payload][26] (`String`) uses Goa's default [codec mechanism][27]. Namely, a `String` 
representation must be surrounded by double quotes. A nice improvement would be to implement a custom codec to support 
our unstructured document as JSON, without the need for quote escaping.

## Next steps

Not the optimal way
Could have shorten the interaction by 1 round-trip
Have the redirect to our server and pick up the consent authorisation code immediately 

<DIAGRAM HERE>

The response from the bank gives us the original consent id which we can use as a correlation identifier

A shorter interaction cycle Reduces attack & error surface area and inevitably leads to a better user experience 

A great exercise for the reader to take this system to v2 


# Discussion & Parting thought

![Discussion](../assets/images/goa-service/kris-n9u9ZEoH2yM-unsplash.jpg)
> Photo by kris on Unsplash

Not the optimal 


![Kaleido architecture](../assets/images/ethereum-stablecoin/kaleido-architecture.png)


# Footnotes

1. <a name="footnote_1"></a>This section is using UML notation only for ease of expression and understanding. The actual 
Goa implementation uses different naming and relations internally. 
2. <a name="footnote_2"></a>E.g. `[1,2,3,4]` and `{"a":6,"b":4}` both have a sum of 10, `[[[2]]]` and `{"a":{"b":4},"c":-2}`
  both have a sum of 2, etc
3. <a name="footnote_3"></a>This is a one-off command. It generates files which you will be editing in the following sections. 



  [1]: https://www.globalstablecoins.co.uk/