# Klaviyo Events

To enable our Klaviyo sync, go to your funnel's "General" settings and enable the "Sync analyses with CRM" option by choosing "Klaviyo" from the dropdown. Afterwards you will need to enter your Klaviyo API key and optionally a list ID if you want to add the profile on a dedicated list in addition to the event.

If Klaviyo events are enabled in your account, we will send an event to Klaviyo when a user has completed the analysis. The event is sent with the following properties:

| Property                    | Description                                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `action`                    | 'AskStella Analysis Completed' (Name of event)                                                                                                   |
| `askstella_funnel_name`     | Short name of the funnel in which this analysis was performed                                                                                    |
| `askstella_analysis_id`     | The analysis ID in our system                                                                                                                    |
| `askstella_analysis_url`    | URL of the analysis results page for this user                                                                                                   |
| `askstella_analysis_result` | One of our color types (`LIGHT`, `LIGHT_WARM`, `WARM`, `DEEP`, `VERY_DEEP`, `SOFT_LIGHT`, `SOFT_MEDIUM`, `SOFT_DEEP`, `SOFT_VERY_DEEP`, `CLEAR`) |
| `askstella_hair_color`      | The hair color the user selected (`VERY_LIGHT`, `MEDIUM_BLONDE`, `LIGHT`, `RED_BLONDE`, `RED`, `MID_BROWN`, `BROWN`, `BLACK`)                    |
| `askstella_eye_color`       | The eye color the user selected (`BLUE`, `GREY`, `GREEN`, `GREEN_BROWN`, `AMBER`, `BROWN`, `DARK_BROWN`)                                         |
| `askstella_skin_color`      | The skin color the user selected (`FAIR`, `LIGHT`, `MEDIUM`, `MEDIUM_DEEP`, `DEEP`)                                                              |
| `askstella_query_string`    | The query string of the URL that led the user to the analysis (e.g. utm parameters you used)                                                     |
| `askstella_products_all`    | List of products (see below)                                                                                                                     |

In addition, if you have generic "single selection" steps in your funnel, we will also send the selected value of these steps as properties. The property name will be the step's key name and the key of the value:
`askstella_[Step Key]: [Options Key]`
e.g. `askstella_skin_dryness: VERY_DRY`

Each product in the product list (askstella_products_all) will have the following properties:

| Property            | Description                                                      |
| ------------------- | ---------------------------------------------------------------- |
| `title`             | Name of the product                                              |
| `image`             | Link to the product image                                        |
| `short_description` | Short description of the product                                 |
| `color`             | HEX Color of the product                                         |
| `link`              | Link to the product page (including the variant ID if necessary) |

If no user (with this email) exists in Klaviyo, we will create a new user and add the event to this new profile. If the user already exists, the event will be added to the user's profile.

The same properties of the event will also be set in the user's profile. This way the profile will always show the most recent analysis results of the user, while the event history will show all the analyses the user has performed over time.
