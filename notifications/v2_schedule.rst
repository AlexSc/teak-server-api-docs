Server Initiated Notification Sending
=====================================

:Endpoint: https://api.gocarrot.com/v2/schedule
:Request Type: POST
:Content-Type: application/json
:Rate Limiting: 30 requests per second

Description: The v2/schedule endpoint allows for bulk scheduling of notifications to be delivered within 30 days. For simplicity this endpoint takes an identifier and message for the notification message to do initial Dashboard setup automatically. If the notification message already exists in the Dashboard this endpoint will not update the content of the notification message.

Notification Schedules created by this endpoint will be Local send types, and will deliver to the user's most recently used device at the time of sending.

Required Parameters
-------------------

:game_id: Your Teak App ID
:secret_key: Your Teak Server Secret
:notification_identifier: The identifier of the notification in the dashboard. Will be created if not present. Limited to 255 characters.
:notification_message: The default message of the notification on all platforms. If the notification already exists, its message will not be updated. Limited to 160 characters.
:user_ids: Array of Game Assigned Player IDs to send the notification to. Maximum of 100 per call. If you need to schedule the notification for more users, make additional calls.

Optional Parameters
-------------------

:send_time: The time in UTC at which to deliver the notification. Must be in ISO8601 format, e.g. "2018-08-23 17:00:00". Must be within 30 days from the time the call is made. If ommitted, or if set to a past time, notifications will be scheduled to deliver immediately.

Success Response
----------------

:Status Code: 200
:Response Body: JSON dictionary with 'status' and 'ids' keys. 'status' will be 'ok'. 'ids' will contain an array of strings which are opaque ids for the deliveries scheduled. Every delivery will have a unique id.
:Example: ``{"status":"ok", "ids":["12904919075210912"]}``

Error Response
--------------
:Status Code: 424
:Response Body: JSON dictionary with 'status' and 'errors' keys. 'status' will be 'error'. 'errors' will be a dictionary in which the keys are the parameter which failed to meet requirements, and the values will be an array of human readable messages for failed validations.
:Example: ``{"status":"error","errors":{"notification_identifier":["must be present"]}}``

Rate Limit Response
-------------------
:Status Code: 429
:Response Body: JSON dictionary with 'status' and 'errors' keys. 'status' will be 'rate_limit'. 'errors' will contain the key 'rate_limit'
:Example: ``{"status":"rate_limit","errors":{"rate_limit":["/v2/schedule may only be called 30 times per second. Please wait a second and try again"]}}``
