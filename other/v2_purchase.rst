Purchase Reporting
==================

:Endpoint: https://api.gocarrot.com/v2/purchase
:Request Type: POST
:Content-Type: application/json or application/x-www-form-urlencoded

Description: The v2/purchase endpoint allows for server to server reporting of in game purchases or withdrawals. This endpoint should only be used if the game is not using standard platform in-app purchasing flows through Facebook Payments, the iOS App Store, Google Play, or the Amazon App Store.

Required Parameters
-------------------

:game_id: Your Teak App ID
:secret_key: Your Teak Server Secret
:user_id: The Game Assigned Player ID of the player who made the purchase or withdrawal. This must match the value provided by the game client's Teak SDK integration.
:amount: The amount of the purchase in USD pennies. For example if the player made a USD $4.99 purchase, amount should be 499. If the player withdrew money this value should be negative.

Optional Parameters
-------------------

:platform: One of "ios", "android", or "desktop". Amazon devices should be reported as "android". If absent, will default to "desktop". Must be correctly set for Teak to attribute purchases to player sessions.
:product_name: The name of the product the player has purchased. If absent Teak will generate a value for this based on the amount purchased or withdrawn.
:platform_id: A unique id for the purchase or withdraw. Teak will deduplicate purchase and withdrawal transactions by platform_id. If absent, Teak will generate a random value for this call.
:store_name: The name of the store front or payment provider the player purchased or withdrew from. If absent will default to "server_api"

Success Response
----------------

:Status Code: 200
:Response Body: JSON dictionary with 'code' key. 'code' will be '200'.
:Example: ``{"code":200}``

Error Responses
--------------

Not Found
^^^^^^^^^
:Status Code: 404
:Response Body: JSON dictionary with 'code' and 'error' keys. 'code' will be 404. 'error' will be a dictionary containing a 'message' key describing the error.
:Example: ``{"code":404,"error":{"message":"Invalid game id or user id"}}``
