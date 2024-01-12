# Checkout Documentation
Version 0.1 <br /> 
First published on Jan 12, 2024 <br/>
Last updated on Jan 12, 2024

Cashtab Checkout for Self-Mint SLP Tokens.

## Installation
To use the Checkout, include the script
```html
<script type="text/javascript" src="https://dev.cert.cash:5000/checkout-v0.1.js"></script> 
```
in your HTML file. That enables you to call the checkout component and to render it.

## Configuration
1. Make a payment requests to obtain a `paymentUrl`
2. Save the corresponding `paymentId`
3. Render the checkout popup with the `paymentUrl` 
4. Upon successful checkout, receive the transaction id through the `onSuccess` callback and validate it

### Rendering the Checkout
Place the code below in an HTML file and load the checkout with the previously obtained `paymentUrl`.
```html
<script>
    document.querySelector('#paymentButton').addEventListener('click', function () {
        Checkout({ 
            paymentUrl: paymentUrl,
            onSuccess: function(txid, link) {
                console.log("success, broadcasted tx:", txid);
                console.log("see in explorer:", link);
                // optional tx validation for expected outputs...
            },
            onCancel: function(error) {
                console.log("payment has been canceled:", error);
            }
        }).render('');
    });
</script>
```
### Example
```js
document.querySelector('#paymentButton').addEventListener('click', function () {
    // define parameters to make payment request
    const params = {
        merchant_name: "Example Merchant",
        merchant_addr: "ecash:qq20ufx3rr00ej96xa2egwggrj7247stxq0awklt2w",
        amount: 4,   
        offer_name: "offer name",
        offer_description: "offer description",
        invoice: "#98dk3jkl",
        return_json: true       
    };
    
    fetch("https://sandbox.cmpct.org/v2?" + new URLSearchParams(params))
        .then(res => res.json())
        .then(function(data) {
            // save payment id 
            const paymentId = data.paymentId;

            // render the checkout popup
            Checkout({ 
                paymentUrl: data.paymentUrl,
                onSuccess: function(txid, link) {
                    console.log("success, broadcasted tx:", txid);
                    console.log("see in explorer:", link);
                },
                onCancel: function(error) {
                    console.log("payment has been canceled:", error);
                }
            }).render('');
    });
});
```
