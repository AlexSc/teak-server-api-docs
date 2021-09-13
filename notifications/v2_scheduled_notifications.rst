Notification Schedule Management
================================

These endpoints allow for the management of Schedules in Teak. Schedules are visible in the dashboard in the Notifications section, and direct Teak to send a Message to an Audience at a specific time, or based on a player entering or leaving the audience.

Creation
--------

:Endpoint: https://api.gocarrot.com/v2/scheduled_notifications
:Request Type: POST
:Content-Type: application/json or application/x-www-form-urlencoded
:Rate Limiting: 5 requests per second

The v2/scheduled_notifications endpoint allows for the creation of Schedules in Teak. Schedules will be displayed in the list of Schedules for the game in the Teak Dashboard and may be managed by Dashboard users.

This endpoint is generally preferred over the Direct Notification Scheduling endpoint.

This endpoint is appropriate for use when the users a notification should be sent to can be defined by an audience, or when a notification needs to be scheduled more than 30 days in advance.

Required Parameters
^^^^^^^^^^^^^^^^^^^

:game_id: Your Teak App ID
:secret_key: Your Teak Server Secret
:name: The name of the Schedule to create. Must be unique among all Schedules for the game. Limited to 255 characters.
:message_name: Specifies the Message the Schedule should send. The name of a Message in the dashboard. Must be present and unarchived. Limited to 255 characters.
:audience_name: Specifies the Audience the Schedule should send the Message to. The name of an Audience in the dashboard. Must be present and unarchived. Limited to 255 characters.
:send_type: Specifies how the Schedule determines when to send. Must be one of "now", "later", or "trigger".
  If "now", the Schedule will send the Message to the Audience immediately upon creation, and cannot be cancelled or paused.

  If "later", the Schedule will send the Message to the Audience at a time specified by the send_time parameter.

  If "trigger", the Schedule will send the message to players who either enter or leave the Audience at the time they do so, based on the trigger_type parameter.


Conditionally Required Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When send_type is "later"
:send_time: Specifies when the Schedule will send the Message to the Audience. Must be an ISO8601 UTC timestamp, e.g. 2021-10-01T17:00:00Z, for a time in the future.

Optional Parameters
^^^^^^^^^^^^^^^^^^^
:send_smear_duration_in_minutes: If specified, will distribute sends roughly evenly over the given number of minutes. Defaults to 0. Must be an integer value greater than or equal to 0 and less than or equal to 240.
:device_priority: Specifies if Teak will deliver the Message to the most recently used device for each player, or all devices used by each player. Must be one of "all", or "recent". Defaults to "recent".
:send_timezone: Specifies if Teak will send to all players at the same time, or to players depending on their last known timezone. Must be one of "UTC", or "local". Defaults to "UTC". Ignored if send_type is not "later".
:trigger_type: Specifies if a Schedule sends the Message when a player "enters" or "leaves" the audience. Must be one of "enters", or "leaves". Defaults to "enters". Ignored if send_type is not "trigger"
:start_time: Specifies when a Schedule with send_type "trigger" will start sending. Must be an ISO8601 UTC timestamp, e.g. 2021-10-01T17:00:00Z, for a time in the future, or null to immediately begin sending the schedule. Defaults to null. Ignored if send_type is not "trigger".
:end_time: Specifies when a Schedule with send_type "trigger" will stop sending. Must be an ISO8601 UTC timestamp, e.g. 2021-10-01T17:00:00Z, for a time in the future, or null to never stop sending the Schedule. Defaults to null. Ignored if send_type is not "trigger".
:enabled: Specifies whether a schedule starts as enabled. A schedule which is not enabled will be visible in the Dashboard by default (i.e. not archived), but will not make any sends. Accepted falsy values are 0 and "false". All other values will be interpreted as truthy. Defaults to true. It is an error to set this value to false when creating a schedule with send_type "now".

Success Response
^^^^^^^^^^^^^^^^

:Status Code: 200
:Response Body: JSON dictionary with "status" and "schedule" keys. "status" will be "ok". "schedule" will contain an object representing the created Schedule with the following keys: id, name, message_name, audience_name, send_type, send_smear_duration_in_minutes, device_priority, enabled, archived

  If send_type is "later", "schedule" will also contain the keys send_time and send_timezone.

  If send_type is "trigger", "schedule" will also contain the keys trigger_type, start_time, and end_time

Error Responses
^^^^^^^^^^^^^^^
Not Found
"""""""""
:Status Code: 404
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "error", "errors" will be a dictionary in which the keys are the parameter which could not be found, and the values will be an array of human readable messages indicating what could not be found.

Validation Error
""""""""""""""""
:Status Code: 422
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "error". "errors" will be a dictionary in which the keys are the parameter which failed to meet requirements, and the values will be an array of human readable messages for failed validations.

Rate Limit Response
"""""""""""""""""""
:Status Code: 429
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "rate_limt". "errors" will contain the key "rate_limit".

Updating
--------
:Endpoint: https://api.gocarrot.com/v2/scheduled_notifications/{id}
:Request Type: PUT
:Content-Type: application/json or application/x-www-form-urlencoded
:Rate Limiting: 5 requests per second

The v2/scheduled_notifications endpoint allows for the updating of Schedules in Teak. At this time the API may only cancel or archive Schedules. To change other parameters of a Schedule, cancel or archive the current Schedule and create a new Schedule.

It is an error to update the enabled state of a Schedule with send_type "now".

It is an error to update the enabled state of a Schedule with send_type "later" if the current time is beyond the Schedule's send_time.

It is an error to update the enabled state of a Schedule with send_type "trigger" if the current time is beyond the Schedule's end_time.

Required URL Fields
^^^^^^^^^^^^^^^^^^^
:id: Must be the id of a previously created Schedule, as specified by the "id" field in the API response.

Required Parameters
^^^^^^^^^^^^^^^^^^^
:game_id: Your Teak App ID
:secret_key: Your Teak Server Secret

Optional Parameters
^^^^^^^^^^^^^^^^^^^
:enabled: A boolean value indicating if the schedule is enabled or not. A disabled schedule will be visible in the Dashboard by default (i.e. not archived) but will not make any sends. Defaults to the current Schedule enabled state. Accepted falsy values are 0 and "false". All other values will be interpreted as truthy.
:archived: A boolean value indicating if the schedule is archived or not. An archived schedule will never be enabled, will not make any sends, and will not be visible in the Dashboard by default. Defaults to the current Schedule archived state. Accepted falsy values are 0 and "false". All other values will be interpreted as truthy.

Success Response
^^^^^^^^^^^^^^^^
:Status Code: 200
:Response Body: JSON dictionary with "status" and "schedule" keys. "status" will be "ok". "schedule" will contain an object representing the created Schedule with the following keys: id, name, message_name, audience_name, send_type, send_smear_duration_in_minutes, device_priority, enabled, archived

  If send_type is "later", "schedule" will also contain the keys send_time and send_timezone.

  If send_type is "trigger", "schedule" will also contain the keys trigger_type, start_time, and end_time

Error Responses
^^^^^^^^^^^^^^^
Not Found
"""""""""
:Status Code: 404
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "error", "errors" will be a dictionary in which the keys are the parameter which could not be found, and the values will be an array of human readable messages indicating what could not be found.

Validation Error
""""""""""""""""
:Status Code: 422
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "error". "errors" will be a dictionary in which the keys are the parameter which failed to meet requirements, and the values will be an array of human readable messages for failed validations.

Rate Limit Response
"""""""""""""""""""
:Status Code: 429
:Response Body: JSON dictionary with "status" and "errors" keys. "status" will be "rate_limt". "errors" will contain the key "rate_limit".
