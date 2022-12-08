# CMPCT Developer Guide
A solution for trustless Multi-Recipient Payments using the [BUX.digital Payment Protocol Merchant Server](https://github.com/bux-digital/documentation/blob/main/merchant-server-api.md).

### Notice for the use of this service
The transactions created over the CMPCT-API use the Badger Universal Token (BUX) which is an eToken on the eCash blockchain. During the initial acquisation of BUX, an authorization code must be bought by the customer - this process is irreversible regardless of whether BUX were actually minted or used for a purchase! It is highly recommended that Agents inform their Customers about the nature of this payment method. Agents who receive payments should take measures to validate the IPN before approving a purchase (see chapter 5).
## Process of the Payment Method
1. Customer chooses payment option BUX or uses direct payment button
2. Agent server sends GET request containing the payment details to CMPCT API
3. CMPCT API returns `paymentLink` that leads the Customer to a non-custodial web wallet with prefilled transaction data
4. Customer pays inside the wallet and, on success, is led to `success_url`
5. Agent receives IPN and validates it before approving the purchase

### 1. Customer chooses payment option BUX

### 2. Agent server sends GET request
A GET request is made to `https://pay.cmpct.org/v1/`. The query string must follow the exact syntax as specified in the [documentation of the BUX.digital Merchant Server](https://github.com/bux-digital/documentation/blob/main/merchant-server-api.md). Agents have to gather ecash addresses from possible recipients with the prefix `ecash` or `etoken` to which the payment of BUX is sent to. 

### 3. CMPCT API returns paymentLink
Agent redirects Customer to `paymentLink` which will be of the form `https://pay.badger.cash/i/{paymentId}`.

### 4. Customer pays inside the wallet
The Customer will be redirected to a non-custodial wallet which will be newly opened if the Customer didn't create one using this browser before. Because the `paymentLink` has been used, the related transaction data will be prefilled into the wallet. The Customer, having no sufficient amount of BUX inside their wallet, will be asked to buy an authorization code to MINT the token inside the wallet. This purchase is done by conventional payment methods. After succesful MINT by the Customer, the token can be used in order to settle the payment request. It is possible that the Customer uses previously minted BUX token. For more information about the non-custodial web wallet, please see [this documentation](https://docs.cashtab.com/docs/).

### 5. Agent receives IPN and validates it
Following the fullfillment of a payment request, an IPN is sent to the URL specified in the `ipn_url` of the initial GET request. This request must be validated by the Agent. Several stages of validation are possible, dependent on the involved risk of the Agent. The recommended stages of validation are as following:
1. validate IP address, existence of transaction and order key
2. validate the transaction outputs

#### 5.1 validate IP address, existence of transaction and invoice
Make sure that the origin of the IPN matches the origin of  `paymentLink` (`https://pay.badger.cash/i/{paymentId}`). The IPN contains a transaction id, `txn_id`, of the broadcasted transaction. This id allows anyone to verify the existence and contents of this transaction by asking a node (see below). Agents should also check whether the IPN truly matches an open order. Due to the public nature of the blockchain it would be possible to attack the Agent's IPN server with previously used or old, but still valid, transactions. See the following sketch down below.

```javascript
const axios = require('axios');

async function postIpn(req, res) {
    const ipn = req.body;
    
    
    // validate ip address
    const ipAddress = req.socket.remoteAddress;
    // compare... 


    // validate existence of transaction
    const url = `https://ecash.badger.cash:8332/tx/${ipn.txn_id}?slp=true`;
    const result = await axios.get(url);
    const txData = result.data;


    // validate that transaction settles new order
    const orderKey = ipn.custom; 
    // compare payment status of order in Agent database...

}
```
If the transaction is existent on the blockchain, `txdata` as defined above will look like this: 
```
{
  hash: 'c403e286094c5f236cf27db31ca26a8e038d708ec71f4aa803931cefcdcfa7e4',
  fee: 444,
  rate: 1047,
  mtime: 1670507577,
  height: 769485,
  block: '000000000000000005a5fff31193b30dcfcffd2125e0d380d57c911f439d56c4',
  time: 1670505744,
  index: 18,
  version: 1,
  inputs: [
    {
      prevout: [Object],
      script: '419033c3cc6ad8a3d90d80f012d11a78c197c458c7d95a52cdc8965fb0de119e0f53f105334ce3771e8b8c998525369e572aae3924f5bfd11873280b32efe24aba412102033df0bcf94dea5bb838eb1187e3b748755a5b87b3feeb16e2eaa6a5b3bbd43c',
      sequence: 4294967295,
      coin: [Object]
    },
    {
      prevout: [Object],
      script: '4184366b1e79947d6d3b253407fc2b28ab2137fdce2add73e2d4db8cb634332c62ee55577bccec31e232ae0d2ef0a9b7702c000e9576c098f7d7f210392d4620c9412102033df0bcf94dea5bb838eb1187e3b748755a5b87b3feeb16e2eaa6a5b3bbd43c',
      sequence: 4294967295,
      coin: [Object]
    }
  ],
  outputs: [
    {
      value: 0,
      script: '6a04534c500001010453454e44207e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e508000000000000c350',
      address: null
    },
    {
      value: 546,
      script: '76a9142c7467c77f5904d6bbbac74e3f81fc13deee5f0d88ac',
      address: 'ecash:qqk8ge780avsf44mhtr5u0uplsfaamjlp5l96rstgd',
      slp: [Object]
    },
    {
      value: 7254,
      script: '76a91414fe24d118defcc8ba37559439081cbcaafa0b3088ac',
      address: 'ecash:qq20ufx3rr00ej96xa2egwggrj7247stxq0awklt2w'
    }
  ],
  locktime: 0,
  hex: '01000000027f85726d8ef49ca0afa9c3acc41de7b17dae4aea2c5fd328941e07fb233900650100000064419033c3cc6ad8a3d90d80f012d11a78c197c458c7d95a52cdc8965fb0de119e0f53f105334ce3771e8b8c998525369e572aae3924f5bfd11873280b32efe24aba412102033df0bcf94dea5bb838eb1187e3b748755a5b87b3feeb16e2eaa6a5b3bbd43cffffffff5a1aea7995a7c322f3e143a30ca094198f06445344fc9008878c8d6512c4269602000000644184366b1e79947d6d3b253407fc2b28ab2137fdce2add73e2d4db8cb634332c62ee55577bccec31e232ae0d2ef0a9b7702c000e9576c098f7d7f210392d4620c9412102033df0bcf94dea5bb838eb1187e3b748755a5b87b3feeb16e2eaa6a5b3bbd43cffffffff030000000000000000376a04534c500001010453454e44207e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e508000000000000c35022020000000000001976a9142c7467c77f5904d6bbbac74e3f81fc13deee5f0d88ac561c0000000000001976a91414fe24d118defcc8ba37559439081cbcaafa0b3088ac00000000',
  slpToken: {
    tokenId: '7e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e5',
    ticker: 'BUX',
    name: 'Badger Universal Token',
    uri: 'https://bux.digital',
    hash: '',
    decimals: 4
  },
  confirmations: 2
}
```


#### 5.2 validate the transaction outputs
If Agents have already validated the IP address, concerns that another token than BUX would have been used are minor. Still, this remains an optional validation which is included in the code sample below. Inside `txData`, there is the possibility to validate the exact amounts for each recipient specified by the GET request in chapter 2. Based on  `order_key` or `offer_name`, Agents can call their database to see which query strings they've used and therefore expect to be existent in the transaction outputs. Since addresses in the outputs might not be in the format that Agents use in their GET request (varying prefixes `ecash`/`etoken`), down below will be an example of converting ecash addresses into the desired format. If this additional dependency is undesired, Agents should use addresses with the `ecash` prefix to make the GET request of chapter 2 as they will surely be provided by every node. 

```javascript 
const ecashaddr = require('ecashaddrjs');

const outputs = txData.outputs;
const buxTokenId = "7e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e5";
const buxDecimals = 4;
const isBuxTransaction = txData.slpToken.tokenId === buxTokenId;

let recipientArray = [];  
if (isBuxTransaction) {
    for (let i = 1; i < outputs.length; i++) {
        const isSlpOutput = outputs[i].slp;
        if (isSlpOutput) {  
            const buxAmount = +(outputs[i].slp.value) / 10**buxDecimals;
            recipientArray.push({
                address: convertAddress(outputs[i].address, "etoken"),
                buxAmount: buxAmount
            });
        }
    }
}


// function returns address with desired prefix
function convertAddress(address, targetPrefix) {
    const { prefix, type, hash } = ecashaddr.decode(address);
    if (prefix === targetPrefix) {
        return address;
    } else {
        const convertedAddress = ecashaddr.encode(targetPrefix, type, hash);
        return convertedAddress;
    }
};
```
The resulting `recipientArray` will contain all addresses of BUX recipients accompanied by their respective amounts (see below). These should now be compared to the expected results. Please be aware that the additional fees by your relay provider will be included in this array as well as possible change back to the sender. Therefore, it should only be tested if expected addresses/amounts are contained in the transaction ouputs, allowing additional recipients to exist as well. If addresses and amounts are as expected, the purchase can be approved.
```
[
  {
    address: 'etoken:qqk8ge780avsf44mhtr5u0uplsfaamjlp53mnpxvv6',
    buxAmount: 5
  }
]
```
