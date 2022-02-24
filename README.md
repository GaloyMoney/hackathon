
# Using the Galoy API

 🎉 Welcome to the hackathon environment for the **Galoy API**: a public instance dedicated to exploring and testing the features of the Galoy API and infrastructure.

For this guide, we're using the Apollo Studio explorer, which enables you to discover and execute our GraphQL API operations in the browser. To access the [explorer tab]({{ graph.url.explorer }}), click the "play" icon on the left of the screen.

To help you understand the types of things you can do with the Galoy API, we have prepared wallets pre-loaded with some satoshis for all hackathon participants to use. You can use your Galoy-managed wallets to send money to any BTC wallet. You can also make payment requests and use them to receive money from any BTC wallet.

The satoshis in your test wallets are yours, and you can withdraw them to your own BTC wallet. Yes you heard that right, FREE CORN 🌽🌽🌽.

Let's get started.

## Getting and using a JWT token

Most of the GraphQL operations you will be making during the hackathon will require user authentication. We have pre-seeded the hackathon environment with accounts you can use. To get access to an account, we will issue JWT tokens to participants through our Slack channel [#hackathon](https://galoymoney-workspace.slack.com/archives/C0344LPRUMU). Please join the channel and message "token", and one of our team will send a JWT token to you via DM.

Once you have a valid JWT token, you'll need to define the `token` environment variable in the [explorer]({{ graph.url.explorer }}) settings (the cog icon):

```js
{
  "token": "JWT_TOKEN_HERE"
}
```

That makes all GraphQL operations you test in the explorer include the required `Authorization` header.

## Getting your account details

Let's start with a query to get some account details (which you'll need later on in this tutorial):

```gql
query Me {
  me {
    defaultAccount {
      wallets {
        id
        walletCurrency
        balance
      }
    }
  }
}
```

You should see that your account has 2 wallets available, each with a pre-funded balance:
- one BTC wallet, in **satoshis**
- one USD wallet, in **USD cents**

_Keep a note of your wallet IDs as you will need them for later operations._

The Galoy API supports sending and receiving payments that are denominated in BTC (satoshis) or in USD (cents). In this tutorial, we'll go over examples of how to do the latter.

You can send USD-denominated payments over lightning or onchain, you can also send direct payments to users of the Galoy API. These direct payments are settled immediately on the Galoy backend. We call them **intraledger** payments. Unlike lightning and onchain payments, intraledger payments have no fees. We'll see examples of intraledger transactions later in this tutorial but let's first start with an example using the lightning network.

## Sending USD over lightning

Let's say someone requested a payment from you over the lightning network. They'll give you a lightning **payment request**. If that payment request does not have a predefined amount, you can use the following mutation to send any USD amount for it:

```gql
mutation LnNoAmountUsdInvoicePaymentSend($input: LnNoAmountUsdInvoicePaymentInput!) {
  lnNoAmountUsdInvoicePaymentSend(input: $input) {
    errors {
      message
    }
    status
  }
}
```

To execute this mutation, you need to define the following variables (bottom pane in the explorer):

```js
{
  "input": {
    "walletId": "YOUR_USD_WALLET_ID_HERE",
    "paymentRequest": "LIGHTNING_INVOICE_PAYMENT_REQUEST_HERE",
    "amount": AMOUNT_TO_SEND_IN_USD_CENTS,
    "memo": "OPTIONAL_NOTE_FOR_THE_RECEIVER"
  }
}
```

Use the USD wallet ID from the previous `Me` query response. If you don't have a lightning payment request, you can generate one at `pay.bbw.sv/<username>`. For example, let's generate a payment request for the [New Story](https://newstorycharity.org/) charity organization. Under https://pay.bbw.sv/new_story, you'll see a QR code. Click on that code to copy the payment request string and use it in the `LnNoAmountUsdInvoicePaymentSend` mutation above to send any USD amount to New Story!

If the payment was successful, the mutation response will have a `SUCCESS` value in the status field. If the payment was not successful, the response will include one or more errors for you to understand what went wrong.

## Requesting USD over lightning

Using the following mutation, you can generate a USD lightning invoice and send it to someone as a payment request:

```gql
mutation LnUsdInvoiceCreate($input: LnUsdInvoiceCreateInput!) {
  lnUsdInvoiceCreate(input: $input) {
    errors {
      message
    }
    invoice {
      paymentRequest
      paymentHash
    }
  }
}
```

To execute this mutation, you need to define the following variables: 

```js
{
  "input": {
    "walletId": "YOUR_USD_WALLET_ID_HERE",
    "amount": AMOUNT_TO_RECEIVE_IN_USD_CENTS,
    "memo": "OPTIONAL_NOTE_FOR_THE_SENDER"
  }
}
```

This operation converts the USD cent amount you specify into the equivalent satoshis and creates a lightning invoice with that amount.

_**Note:** this lightning invoice expires after 2 minutes._

## Creating a USD invoice on behalf of a Galoy API user

You can generate a USD lightning invoice on behalf of any Galoy API user (including yourself). This is the process used by the pay.bbw.sv page that we used earlier.

To do that process through the API, you'll first need to use a query operation to get the wallet ID for the recipient. You can use the following query for that:

```gql
query UserDefaultWalletId($username: Username!) {
  userDefaultWalletId(username: $username)
}
```

You'll need the following variable:

```js
{
  "username": "RECIPIENT_USERNAME_HERE"
}
```

This responds with a wallet ID value that you'll need to use in the next operation.

<!-- TODO: Figure out how to get the USD wallet ID -->

To create the lightning invoice, you can use the following mutation:

```gql
mutation LnUsdInvoiceCreateOnBehalfOfRecipient($input: LnUsdInvoiceCreateOnBehalfOfRecipientInput!) {
  lnUsdInvoiceCreateOnBehalfOfRecipient(input: $input) {
    errors {
      message
    }
    invoice {
      paymentRequest
      paymentHash
    }
  }
}
```

To execute this mutation, you need to define the following variables: 

```js
{
  "input": {
    "walletId": "THE_USD_WALLET_ID_FOR_THE_RECIPIENT",
    "amount": AMOUNT_TO_RECEIVE_IN_USD_CENTS,
    "memo": "OPTIONAL_NOTE_FOR_THE_SENDER"
  }
}
```

This operation converts the USD cent amount you specify into the equivalent satoshis and creates a lightning invoice with that amount.

_**Note:** this lightning invoice expires after 2 minutes._

## Subscribing to updates

The Galoy API supports a few subscription operations where you can get immediate updates from the server when there is new data of interest. For example, you can use the following subscription operation to get BTC price updates and updates on any lightning transactions under your account:

```gql
subscription MyUpdates {
  myUpdates {
    errors {
      message
    }
    update {
      ... on Price {
        base
        offset
        currencyUnit
        formattedAmount
      }
      ... on LnUpdate {
        paymentHash
        status
        walletId
      }
    }
  }
}
```

Everytime there is a price update, this subscription operation will push a response containing the new price data. It'll also push a response everytime a lightning invoice under your account gets paid.

## More examples

WIP

### Setting a username

The username serves two purposes.

 1. Allows users to make payments to each other internally to this environment using just the username.
 2. Creates a lightning address which can be used to pay over the lightning network using the lnurl spec.

```gql
mutation UserUpdateUsername ($userUpdateUsernameInput: UserUpdateUsernameInput!) {
  userUpdateUsername(input: $userUpdateUsernameInput) {
    errors {
      message
    }
    user {
      id
      username
    }
  }
}
```

Remember to populate the variables pane at the bottom of the explorer window before you run the mutation or you will get errors.  As an example:

```js
{
  "userUpdateUsernameInput": {
    "username": "CHANGEME"
  }
}
```

### Creating a lightning invoice

```gql
mutation Mutation($input: LnInvoiceCreateInput!) {
  lnInvoiceCreate(input: $input) {
    invoice {
      paymentRequest
      paymentHash
      paymentSecret
      satoshis
    }
    errors {
      message
    }
  }
}
```

Remember to include the variables for the mutation input in the pane located at the bottom of the window.  As an example:

```js
{
  "input": {
    "walletId": null,
    "amount": null,
    "memo": null
  }
}
```

* `walletId`: use one of the wallet IDs that you took a note of from the initial query for account info.
* `amount`: choose an amount for the invoice.
* `memo`: Optionally include a memo (message string).

If you have a lightning wallet with available satoshis you can pay the invoice by using the returned `paymentRequest` if you want.  Otherwise you can paste the `paymentRequest` into [lndecode](https://lndecode.com/) to see the details of the invoice which has been generated for you.


### Sending BTC to external wallet

You can send any BTC amount to another wallet over lightning or onchain.  As an example let's say you want to send via lightning.

```gql
mutation LnInvoicePaymentSend($input: LnInvoicePaymentInput!) {
  lnInvoicePaymentSend(input: $input) {
    errors {
      message
    }
    status
  }
}
```

```js
{
  "input": {
    "walletId": null,
    "paymentRequest": null,
    "memo": null
  }
}
```

The API supports sending to a lightning invoice with a predefined amount using the `LnInvoicePaymentSend` or lightning invoice with no amount specificed using `lnNoAmountInvoicePaymentSend`.  Both of these mutations return a status field indicating the success or failure of the transaction.


### Getting help

For support working with this graph, contact the Galoy team at our Slack channel [#hackathon](https://galoymoney-workspace.slack.com/archives/C0344LPRUMU.)


## Links:

- Schema: {{ graph.url.sdl }}
- Playground: [api.freecorn.galoy.io/graphql](https://api.freecorn.galoy.io/graphql)
