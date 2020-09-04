Claiming Teak Rewards
=====================

Depending on your needs reward claiming can be done through your game client **or** your game server.

Client Integration
------------------

When the game loads, if there is a teak_reward_id present in the query parameters of the game URL, then the user should be granted a reward. You can start the reward flow by calling ``window.teak.claimReward(callback)``. This call must be made **after** calling ``window.teak.identify()``.

Our SDK will make a call into the Teak backend to confirm that user is eligible to receive this reward. If so, two things will happen simultaneously

* The callback given will be called
* Teak will make a POST request to your backend as described in our :doc:`rewarding/endpoint` documentation.

.. highlight:: json

The callback will be passed a single object parameter of the following form::

    {
      "metadata" : {
        "serverVersion": "",
        "serverId": "",
        "requestId": "",
        "requestTime": "",
        "code": 200
      },
      "response" : {
        "status" : "",
        "reward" : ""
      }
    }

``response.status`` will be one of the following:

:grant_reward: The user has been issued this reward by Teak
:self_click: The user has attempted to claim a reward from their own social post
:already_clicked: The user has already been issued this reward
:too_many_clicks: The reward has already been claimed its maximum number of times globally
:exceed_max_clicks_for_day: The user has already claimed their maximum number of rewards of this type for the day
:expired: This reward has expired and is no longer valid
:invalid_post: Teak does not recognize this reward id

If ``response.status`` is ``grant_reward`` then response.reward will be a JSON encoded object of the form ``{"internal_id" : quantity, "another_internal_id" : anotherQuantity}``. If ``response.status`` is any other value, the ``response.reward`` key will not be present.

Teak **does not** provide any in game UI. It is up to the game to inform users of the reward they were issued or why the reward was not granted.

Server integration
------------------

The server integration is identical to the client integration, except the reward flow is started by making a POST request to::

    https://rewards.gocarrot.com/<<teak_reward_id>>/clicks?clicking_user_id=<<your_user_id>>

This endpoint will respond with a status of 200 if the user has been granted the reward and a status of 403 if the user has not been granted the reward. The response body will be identical to the response passed to the callback in the client integration.