# Tracking of analysis sources

To analyse from where users came to start an analysis, we store the `source` parameter of the query string of the iframe's URL with the analysis. We attach this parameter automatically for sources out of our own tools, e.g. the chat from StelleAssist will open the analysis with `https://<your identifier>.askstella.ai/?source=chat`. If you want to track sources from your website, you can add the `source` parameter to the URL of the iframe yourself with any parameter you like.

If you need even more control, we are also storing the full query string of the iframe's URL in the `query_string` property of the analysis, which is available to you in the CRM sync and data exports.
