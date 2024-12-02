# e-commerce integration

If you are using Shopify and our ShopifyApp, then you don't need to worry about the integration, as it is already set up
for you. If you are using a different e-commerce platform, you will find here the information you need to integrate our
system to gain the most out of it.

## Analysis integration

Our StellaMatch analysis can be integrated as an iframe on any page of your website. The iframe URL is
`https://<your identifier>.askstella.ai/`. This can be on a dedicated page or for exmaple in a popup on a product page.

```html
<iframe
  src="https://<YOUR-IDENTIFIER>.askstella.ai/"
  height="700px"
  width="100%"
  frameborder="0"
  id="askstella-iframe"
  allow="autoplay; camera;"
></iframe>
```

### Automatic height adjustment

Each step in the analysis funnel can have a different height. Either you choose a height for the iframe that is high
enough for all steps or you will get a scroll bar within the iframe. Alternatively you can adjust the height of the
iframe automatically, because our iframe always reports the needed height to the outer page in a postMessage.
Our PostMessage has the following payload:

```json
{
  "type": "height_changed",
  "height": 1000
}
```

You can use the following code snippet:

```html
<script type="text/javascript">
  // catch post messages from StellaMatch
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }
      let message = e.data;
      if (message && message.type === "height_changed") {
        const iframe = document.querySelector("#askstella-iframe");
        if (!iframe) {
          console.warn("StellaMatch iframe not found");
          return;
        }

        // Set the height of the iframe
        iframe.style.height = message.height + "px";
      }
    },
    false
  );
</script>
```

If the previous step was very long and the user is scolled down, especially on mobile phones, it can happen, that the
next step is not visible. To avoid this, you can scroll the page to the top of the iframe with the following code
extension. If you have a static menu on the top, you need to set the offset to the height of the menu.

```js
// Scroll the page to the top of the iframe if it is not in the viewport
const iframeTop = iframe.getBoundingClientRect().top;
const viewportHeight = window.innerHeight || document.documentElement.clientHeight;
if (iframeTop < 0 || iframeTop > viewportHeight) {
  // If you have a static menu on the top, set the rough height of it here to makes sure the iframe is not hidden behind it
  let offset = 100;
  let y = iframe.getBoundingClientRect().top + window.pageYOffset - offset;
  window.scrollTo({ top: y, behavior: "smooth" });
}
```

### Displaying the user's analysis results

If the user already did an analysis, it is recommended to display the result in the iframe the second time he/she opens
the page. That way the user can always get back to her/his result page. At the bottom of the result page is a button to
restart the analysis.

Do make this possible, we send a post message to the outer page whenever the user finishes an analysis. You can listen
to this message and save the URL, e.g. in the localStorage of the user's browser. If the user comes back to the page,
you can then set the URL as the source of the iframe.

Our PostMessage has the following payload:

```json
{
  "type": "analyse_finished",
  "analysis_url": "https://<your identifier>.askstella.ai/...",
  "products": [...], // not relevant here
  "name": "FirstName of user", // not relevant here
  "colorType": "Analysis Result" // not relevant here
}
```

Store the URL and load it when the page is loaded:

```html
<script type="text/javascript">
  // catch post messages from StellaMatch
  window.addEventListener("message", function (e) {
    if (!e.origin.endsWith(".askstella.ai")) {
      // Message not from askStella
      return;
    }
    let message = e.data;
    if (message && message.type === "analyse_finished") {
      // store the URL in the localStorage
      localStorage.setItem("askstella_analysis_url", message.analysis_url);
    }
  });

  // load the result page if analysis was done
  document.addEventListener("DOMContentLoaded", () => {
    // Retrieve the URL from local storage
    const analysisUrl = localStorage.getItem("askstella_analysis_url");
    if (analysisUrl) {
      const iframe = document.querySelector("#askstella-iframe");
      if (!iframe) {
        console.warn("StellaMatch iframe not found");
        return;
      }
      // Change the iframe's src attribute
      iframe.src = analysisUrl;
    }
  });
</script>
```

In case your customers have an account on your website, you can also store the analysis URL in your database and show
the result page in a dedicated section of the user's profile.

### Add to cart functionality

On the result page the user has the option to add an item directly to the cart. To make this possible, you need to
listen to the post message from the iframe and add the product to the cart.
If the store also offers sets/bundles of multiple products and we recommend them, we will add a `setId` to the message
to indicate that the items are part of a set. For example, the setId might refer to a product bundle that offers
purchasing foundation and a concealer together for a reduced price.
of a foundation and a concealer, then the setId would be the ID of the bundle and the items array would contain the IDs
of the variation of the foundation and variation of the concealer that were recommended.

The post message has the following payload:

```json
{
  "type": "add_to_cart",
  "source": "...", // (optional) for advanced tracking the value of the "&source=..." query parameter is forwarded here
  "setId": "234", // if this is a set, the ID of the set otherwise undefined
  "items": [
    {
      "id": "123",
      "quantity": 1,
    },
    ...
  ],
}
```

You can use the following code snippet to add the product to the cart:

```html
<script type="text/javascript">
  // catch post messages from StellaMatch
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }
      let message = e.data;
      if (message && message.type === "addToCart") {
        const items = { items: message.items };

        // add items to cart (according to your e-commerce system)
        if (message.setId) {
          // add set with provided item list to cart
        } else {
          // add single items to cart
        }
        // if successfull, show cart drawer and/or update cart icon
      }
    },
    false
  );
</script>
```

## Pre-fill sets with recommended products

If the shop is also offering sets/bundles of multiple products and we are also recommenging those, then you can pre-fill
the set with the recommended products when the user clicks on our recommendation. We will open the bundle's product page
and attach the recommended variations of the included products to the URL as query parameters.
E.g. `https://yourshop.com/bundle/123?variations=234,345`.

Use this information to pre-fill the set with the recommended items.

## Tracking purchases

When a user is making an analysis, we recommend products to her/him. To track revenue resulting from these
recommendations, each purchase must be analysed if it contains a product that was recommended by us.

Wheneber a user finishes an analysis (`analyse_finished`) and in addition whenever a user opens their result page
(`result_page_opened`), we send a post message to the outer page. This message contains the products that were
recommended to the user. You can store this information in the local storage of the user's browser and use it to track
purchases.

We recommend to also take the `result_page_opened` message into account, because each time the user opens the result we
will re-evaluate the recommendations based on the newest shop inventory.

Our PostMessage has the following payload:

```json
{
  "type": "analyse_finished", // or result_page_opened
  "analysis_url": "https://<your identifier>.askstella.ai/...",
  "products": [...], // the recommended products
  "name": "FirstName of user", // not relevant here
  "colorType": "Analysis Result" // not relevant here
}
```

The `products` array contains the following information for each product:

```json
{
  "id": "123", // the id of the product, more precisely the variant's id of the recommended variant of a product
  "link": "url of product"
}
```

Storing and loading the products:

```html
<script type="text/javascript">
  // catch post messages from StellaMatch
  window.addEventListener("message", function (e) {
    if (!e.origin.endsWith(".askstella.ai")) {
      // Message not from askStella
      return;
    }
    let message = e.data;
    if (message && (message.type === "analyse_finished" || message.type === "result_page_opened")) {
      // store the products in the localStorage
      localStorage.setItem("askstella_recommended_products", JSON.stringify(message.products));
    }
  });

  // this step now heavily depends on your e-commerce system
  // either evaluate the purchase on the client side and mark the purchase
  // or transfer the recommended products to your server and mark the purchase there
  document.addEventListener("PurchaseDone", (cart) => {
    // Retrieve the recommended products from local storage
    let recommendedProducts = localStorage.getItem("askstella_recommended_products");
    if (!recommendedProducts) {
      return;
    }
    recommendedProducts = JSON.parse(recommendedProducts);
    // check each product in the purchase if it was recommended
    for (let product of cart.products) {
      for (let recommendedProduct of recommendedProducts) {
        if (product.id === recommendedProduct.id) {
          // mark the product as "recommended by Stella"
        }
      }
    }
  });
</script>
```

### Allow us to analyse the data in our dashboard (coming soon)

Having the information about the purchases, we can evaluate the success of our recommendations and provide our client
with valuable insights on our dashboard.

POST all purchases made in your show to our API endpoint `https://api.askstella.ai/v1/purchase` with the following JSON payload:

```json
{
  "products": [
    {
      "id": "123",
      "price": 12.34,
      "quantity": 1,
      "was_recommended": true
    },
    ...
  ]
}
```

Use your API key in the header `Authorization: Bearer <API_KEY>` to authenticate the request.

## Show recommendations on the product page or listing

In the previous sections, we explained how to store the recommended products in the local storage of the user's browser.
This information can be used to show the user the recommended products on the product page or in the listing.

```js
// assuming that #recommended-products is the container where the recommended products should be shown,
// e.g. a floating div on the right side of the page
const widget = document.querySelector("#recommended-products");
if (!widget) {
  console.warn("Widget not found");
  return;
}

// Retrieve the recommended products from local storage
let recommendedProducts = localStorage.getItem("askstella_recommended_products");
if (!recommendedProducts) {
  // user has not done an analysis yet, recommend it to the user
  widget.innerHTML = `
    <p>Do you need help choosing the right colour for you?<br/>
      <a href="#" onClick="openAnalysisInAPopop(); return false;">
        Find your colour
      </a>
    </p>
  `;
  return;
}
recommendedProducts = JSON.parse(recommendedProducts);

widget.innerHTML += `
<div>
    <p>Your recommended colours:</p>
    <ul id="recommended-list"></ul>
</div>
`;
let recommendationFound = false;
// assuming that variations[{id: 1234, name: "nude",...},...] holds the information of each variant of the product on the current page
for (let recommendedProduct of recommendedProducts) {
  for (let variation of variations) {
    if (variation.id === recommendedProduct.id) {
      // show the recommended product on the page
      const ul = document.querySelector("#recommended-list");
      const li = document.createElement("li");
      li.textContent = variation.name;
      ul.appendChild(li);
      recommendationFound = true;
    }
  }
}
if (recommendationFound) {
  widget.style.display = "block";
} else {
  widget.style.display = "none";
}
```

## Integration with StellaAssist

(Coming soon)

### Update the chat with analysis results

(Coming soon)

### Open the analysis funnel or the user's result page on request from the chat

(Coming soon)

## Full code example/skeleton

### Code for page(s) where the analysis iframe is integrated

```html
<iframe
  src="https://<YOUR-IDENTIFIER>.askstella.ai/"
  height="700px"
  width="100%"
  frameborder="0"
  id="askstella-iframe"
  allow="autoplay; camera;"
></iframe>

<script type="text/javascript">
  // catch post messages from StellaMatch
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        // Message not from askStella
        return;
      }
      const message = e.data;
      if (!message || !message.type) {
        return;
      }
      const iframe = document.querySelector("#askstella-iframe");
      if (!iframe) {
        console.warn("StellaMatch iframe not found");
        return;
      }

      if (message && message.type === "height_changed") {
        // Set the height of the iframe
        iframe.style.height = message.height + "px";

        // Scroll the page to the top of the iframe if it is not in the viewport
        const iframeTop = iframe.getBoundingClientRect().top;
        const viewportHeight = window.innerHeight || document.documentElement.clientHeight;
        if (iframeTop < 0 || iframeTop > viewportHeight) {
          // If you have a static menu on the top, set the rough height of it here to makes sure the iframe is not hidden behind it
          let offset = 100;
          let y = iframe.getBoundingClientRect().top + window.pageYOffset - offset;
          window.scrollTo({ top: y, behavior: "smooth" });
        }
      } else if (message.type === "analyse_finished" || message.type === "result_page_opened") {
        localStorage.setItem("askstella_analysis_url", message.analysis_url);
        localStorage.setItem("askstella_recommended_products", JSON.stringify(message.products));
      } else if (message.type === "addToCart") {
        const items = { items: message.items };

        // add items to cart (according to your e-commerce system)
        if (message.setId) {
          // add set with provided item list to cart
        } else {
          // add single items to cart
        }
        // if successfull, show cart drawer and/or update cart icon
      }
    },
    false
  );

  // load the result page if analysis was done
  document.addEventListener("DOMContentLoaded", () => {
    // Retrieve the URL from local storage
    const analysisUrl = localStorage.getItem("askstella_analysis_url");
    if (analysisUrl) {
      const iframe = document.querySelector("#askstella-iframe");
      if (!iframe) {
        console.warn("StellaMatch iframe not found");
        return;
      }
      // Change the iframe's src attribute
      iframe.src = analysisUrl;
    }
  });

  // this step now heavily depends on your e-commerce system
  // either evaluate the purchase on the client side and mark the purchase
  // or transfer the recommended products to your server and mark the purchase there
  document.addEventListener("PurchaseDone", (cart) => {
    // Retrieve the recommended products from local storage
    let recommendedProducts = localStorage.getItem("askstella_recommended_products");
    if (!recommendedProducts) {
      return;
    }
    recommendedProducts = JSON.parse(recommendedProducts);
    // check each product in the purchase if it was recommended
    for (let product of cart.products) {
      for (let recommendedProduct of recommendedProducts) {
        if (product.id === recommendedProduct.id) {
          // mark the product as "recommended by Stella"
        }
      }
    }
  });
</script>
```

### Code for the product page or listing

```html
<script type="text/javascript">
  // assuming that #recommended-products is the container where the recommended products should be shown,
  // e.g. a floating div on the right side of the page
  const widget = document.querySelector("#recommended-products");
  if (!widget) {
    console.warn("Widget not found");
    return;
  }

  // Retrieve the recommended products from local storage
  let recommendedProducts = localStorage.getItem("askstella_recommended_products");
  if (!recommendedProducts) {
    // user has not done an analysis yet, recommend it to the user
    widget.innerHTML = `
    <p>Do you need help choosing the right colour for you?<br/>
      <a href="#" onClick="openAnalysisInAPopop(); return false;">
        Find your colour
      </a>
    </p>
  `;
    return;
  }
  recommendedProducts = JSON.parse(recommendedProducts);

  widget.innerHTML += `
<div>
    <p>Your recommended colours:</p>
    <ul id="recommended-list"></ul>
</div>
`;
  let recommendationFound = false;
  // assuming that variations[{id: 1234, name: "nude",...},...] holds the information of each variant of the product on the current page
  for (let recommendedProduct of recommendedProducts) {
    for (let variation of variations) {
      if (variation.id === recommendedProduct.id) {
        // show the recommended product on the page
        const ul = document.querySelector("#recommended-list");
        const li = document.createElement("li");
        li.textContent = variation.name;
        ul.appendChild(li);
        recommendationFound = true;
      }
    }
  }
  if (recommendationFound) {
    widget.style.display = "block";
  } else {
    widget.style.display = "none";
  }
</script>
```