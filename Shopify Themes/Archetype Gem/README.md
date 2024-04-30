# Archetype Gem Shopify Theme

With the following code you can integrate our IFrame, enabling automatic resizing and the "Add to cart" function.
Only replace the "<YOUR-IDENTIFIER>" with your identifier, e.g. https://my-store.askstella.ai and paste it into your Shopify page.

```html
<script type="text/javascript">
  window.addEventListener(
    "message",
    function (e) {
      if (!e.origin.endsWith(".askstella.ai")) {
        console.debug("Message from other origin received: " + e.origin);
        return;
      }

      let message = e.data;
      if (message.type === "addToCart") {
        const data = { items: message.items };

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
            document.dispatchEvent(new CustomEvent("cart:build"));
            document.dispatchEvent(new CustomEvent("cart:open"));
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
