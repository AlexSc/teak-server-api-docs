Teak Reward Endpoint
====================

What is the Teak Reward Endpoint?
---------------------------------

To issue virtual currency or virtual items in response to a user clicking on a notification, social post, or Teak Reward Link you must **set up a server endpoint that can receive a form POST request** from us. This server side endpoint must:

* Validate the request
* Issue the specified reqrds to the customer
* Respond to the request and indicate success or failure

The Request
-----------

Teak will issue a POST request to a specified endpoint. There is only one endpoint per app, and it is set as the "Server Enpoint" under "Server to Server Communication" in your app's settings.

The request will have the following fields set in the POST body:
    :app_id: The Teak App ID of the application (This is typically your Facebook App ID)
    :clicking_user_id: The internal UID of the user who has clicked on the post and should receive the reward.
    :event_id: A random string unique to issuing this reward to this user
    :post_id: A 64 bit integer which identifies the post, request, notification, or CRM gift that the user clicked on
    :post_type: A string identifying the post or CRM gift in the Teak CMS
    :posting_user_id: The ID of the user who made the post or request, or 0 if this is a notification or Teak reward link
    :reward: A JSON blob with what the user should be rewarded with. The format is {"internal\_id" : quantity, "another\_internal\_id" : quantity}. For example, {"coins" : 25, "energy" :10 }
    :timestamp: The UNIX time at which Teak first attempted to grant this reward

    :signature: A SHA256 HMAC signature of the request built as follows

::

    POST\n
    {{teak_reward_endpoint_url}}\n
    {{key}={{value}}&{{key2}}={{value}} // etc. No terminating newline

.. highlight:: ruby

All keys are sorted alphabetically, and then the entire string is signed using the Server to Server Teak App Secret (available under "Server to Server Communication in your app's settings). The signature is base64 encoded and CGI escaped. In ruby this looks like::

      string_to_sign = "POST\n#{teak_reward_endpoint_url}\n"
      sig_keys = callback_parameters.keys.sort
      string_to_sign += sig_keys.map { |k| "#{k}=#{callback_parameters[k]}" }.join('&')
      signature = CGI.escape(Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest::Digest.new("sha256"), teak_server_secret, string_to_sign)).strip)


Validating the request
----------------------

To validate the request we recommend computing the signature using the above method and comparing it to the signature passed in by Teak. If the signatures do not match, then the request is invalid and should be denied.

Alternatively you can check that the request originated from one of the following IP addresses

* 54.174.233.10
* 54.173.237.66
* 54.86.179.227
* 54.172.184.138
* 54.152.126.152

This list may be updated at any time without notice. As such, we **strongly recommend** that you perform signature validation and not IP address based validation.

Furthermore you should confirm that the user has not already been granted a reward with the given event_id. In the event of a network failure it is possible that Teak will attempt to retry granting the reward. When doing so all parameters, including event_id, will be identical. If the user has already been granted a reward with the given event_id simply return a successful response and do no further processing

Issuing the Reward
------------------

Once the request has been validated you must JSON decode the 'reward' parameter. This will be a dictionary of the internal id of the reward and the quantity. For example, many games internally refer to their earned and purchased currencies as "softCash" and "hardCash", while the user facing names may be "Coins" and "Gold". In the reward dictionary Teak will use the internal "softCash" and "hardCash" names. An example reward parameter would be

``{"softCash" : 50, "hardCash" : 10}``

The reward parameter may include any number of rewards and quantities. It is important that your reward endpoint properly grants the user **all** specified rewards.

Responding to the Request
-------------------------

We expect the endpoint to respond with the HTTP status code 200 and the string TEAKOK anywhere in the response body. If the status code is not 200 or the string TEAKOK does not appear in the response body, Teak will retry the request 16 times with an exponential back off over a period of ~3 days. Each retry will be completely identical. No parameters will ever change between retries.

Testing the endpoint
--------------------

You can test that your endpoint is working from the 'Bundles' tab in the Rewards page for your app in Teak. Create a bundle to test with (we recommend one that gives out a variety of currencies and items, if applicable) and then click 'Test'

In the modal that pops up, enter a user id in User Id: and 0 in Posting User ID, then click Preview. Teak will attempt to grant the reward by POST-ing to your server, and will show the POST request made, the string which Teak signed with your Server to Server key and how the server responded. If your server responds with 200 and 'TEAKOK' in the body, and you confirm that the user was granted the items, you're ready to ship!