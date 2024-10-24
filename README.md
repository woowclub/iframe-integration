# Integration of the askStella IFrame

For the best user experience, our IFrame should be included with our resize script so that the height of the IFrame adapts to its content needs.

Please note that you need to replace the `<YOUR-IDENTIFIER>` with your identifier, e.g. https://my-store.askstella.ai/ and paste it into your Shopify page.

```html
<script type="text/javascript">
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }
      let message = e.data;
      let iframe = document.querySelector("#askstella-iframe");
      iframe.style.height = message.height + "px";

      // If you have a static menu on the top, set the rough height of it here to makes sure the iframe is not hidden behind it
      let offset = 100;
      let y = iframe.getBoundingClientRect().top + window.pageYOffset - offset;
      window.scrollTo({ top: y, behavior: "smooth" });
    },
    false
  );
</script>

<iframe
  src="https://<YOUR-IDENTIFIER>.askstella.ai/"
  height="700px"
  width="100%"
  frameborder="0"
  id="askstella-iframe"
  allow="autoplay; camera;"
></iframe>
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

Here is an example of how to catch this message (in addition to the iframe resize) and put the product into a shopify cart. Please note that this will not trigger yet an update of the cart icon. This needs to be implemented according the options of your used shopify theme.

Code for some Shopify Themes can be found here:

- [Archetype Expanse](Shopify%20Themes/Archetype%20Expanse/README.md)
- [Archetype Fetch](Shopify%20Themes/Archetype%20Fetch/README.md)
- [Archetype Gem](Shopify%20Themes/Archetype%20Gem/README.md)
- [Archetype Impulse](Shopify%20Themes/Archetype%20Impulse/README.md)
- [Archetype Motion](Shopify%20Themes/Archetype%20Motion/README.md)
- [Maestrooo Impact](Shopify%20Themes/Maestrooo%20Impact/README.md)
- [Maestrooo Prestige](Shopify%20Themes/Maestrooo%20Prestige/README.md)

Generic example:

```html
<script type="text/javascript">
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }

      let message = e.data;
      if (message.type === "addToCart") {
        const data = { items: message.items };

        // add item to cart (according to theme)
        fetch("/cart/add.js", {
          method: "post",
          body: JSON.stringify(data),
          headers: {
            accept: "application/json",
            "Content-Type": "application/json",
            "Accept-Language": "en-US,en;q=0.8",
          },
        })
          .then((res) => res.json())
          .then((data) => {
            console.debug(data);
            // show cart drawer (according to theme)
          })
          .catch((e) => {
            console.error(e);
          });
      } else if (message.height) {
        // this seems to be an iframe resize message
        let iframe = document.querySelector("#askstella-iframe");
        iframe.style.height = message.height + "px";
        // If you have a static menu on the top, set the rough height of it here to makes sure the iframe is not hidden behind it
        let offset = 100;
        let y = iframe.getBoundingClientRect().top + window.pageYOffset - offset;
        window.scrollTo({ top: y, behavior: "smooth" });
      }
    },
    false
  );
</script>

<iframe
  src="https://<YOUR-IDENTIFIER>.askstella.ai/"
  height="700px"
  width="100%"
  frameborder="0"
  id="askstella-iframe"
  allow="autoplay; camera;"
></iframe>
```

# Integration of the customer's analysis result as IFrame

If you offer a login for your customers, you can integrate the user's analysis results as an IFrame in their profile after they have gone through the analysis process. In this way, the users can access their results and recommendations at any time. To integrate the IFrame, you need our "User ID" of the analysis that the user has performed. This ID is synchronized with your Klaviyo instance (or similar) after each analysis. Store the ID in your database next to the user's eMail address so you can retrieve it on the profile page.

The integration is the same as for the analytics IFrame, but you need to use the profile link and add the "user" parameter dynamically to the URL. Please note that you need to replace the `<YOUR-IDENTIFIER>` with your identifier and the `<USER-ID>` with the user ID, e.g. https://my-store.askstella.ai/profile/?user=123 and paste it into your Shopify page. If you are using our "Add to cart" functionality, check the explanation above and integrate the code accordingly here as well.

```html
<script type="text/javascript">
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }
      let message = e.data;
      let iframe = document.querySelector("#askstella-iframe");
      iframe.style.height = message.height + "px";
      // If you have a static menu on the top, set the rough height of it here to makes sure the iframe is not hidden behind it
      let offset = 100;
      let y = iframe.getBoundingClientRect().top + window.pageYOffset - offset;
      window.scrollTo({ top: y, behavior: "smooth" });
    },
    false
  );
</script>

<iframe
  src="https://<YOUR-IDENTIFIER>.askstella.ai//profile/?user=<USER-ID>"
  height="700px"
  width="100%"
  frameborder="0"
  id="askstella-iframe"
  allow="autoplay; camera;"
></iframe>
```
