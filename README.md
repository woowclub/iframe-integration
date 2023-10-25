# Integration of the askStella IFrame
For the best user experience, our IFrame should be included with our resize script so that the height of the IFrame adapts to its content needs.
```html
<script type="text/javascript">
    window.addEventListener('message', function(e) {
        if (!e.origin.endsWith(".askstella.ai")) {
            console.debug("Message from other origin received: " + e.origin);
            return;
        }
        let message = e.data;
        let iframe = document.querySelector("#askstella-iframe");
        iframe.style.height = message.height + 'px';
        iframe.scrollIntoView({ behavior: "smooth" });
    } , false);
</script>

<iframe src="https://<YOUR-IDENTIFIER>.askstella.ai/" height="700px" width="100%" frameborder="0" id="askstella-iframe" allow="autoplay; camera;"></iframe>
```

## "Add to cart" functionality
If you opted in for the "Add to cart" functionality (we are showing cart icons on the user's profile to allow them to put products directly into their shopping cart), you need to catch those messages as well and act according to your shopping system to put the product into the user's basket.

A sample "add to cart" message looks like this:
```js
{
    type: "addToCart",
    items: 
        [
            {
                id: [VARIATION_ID_OF_PRODUCT],
                quantity: 1
            }
        ]
    }
```
Currently, we are only allowing single items to be added to the cart; therefore, the length of the items array is always one, as well as the quantity.

Here is an example of how to catch this message (in addition to the iframe resize):
```html
<script type="text/javascript">
    window.addEventListener('message', function(e) {
        if (!e.origin.endsWith(".askstella.ai")) {
            console.debug("Message from other origin received: " + e.origin);
            return;
        }

        let message = e.data;
        if ((message.type === "addToCart")) { 
            const data = { items: message.items }
            
            fetch('/cart/add.js', {
                method: 'post', body: JSON.stringify(data), headers: {
                    'accept': 'application/json',
                    'Content-Type': 'application/json',
                    'Accept-Language': 'en-US,en;q=0.8'
                },
            }).then(res => res.json())
                .then(data => {
                    console.debug(data);
                }).catch(e => {
                    console.error(e);
                })
        } else if (message.height) {
            // this seems to be an iframe resize message
            let iframe = document.querySelector("#askstella-iframe");
            iframe.style.height = message.height + 'px';
            iframe.scrollIntoView({ behavior: "smooth" });
        }
    } , false);
</script>

<iframe src="https://<YOUR-IDENTIFIER>.askstella.ai/" height="700px" width="100%" frameborder="0" id="askstella-iframe" allow="autoplay; camera;"></iframe>
```