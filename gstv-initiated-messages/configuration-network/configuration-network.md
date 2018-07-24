# GSTV Initiated Message - Network Configuration Updates

Message sent with the network wide configuration values including shutdown hours and evergreen video settings

## Key/Value Glossary
### Fields from GSTV Messages
- **guid** - _required_ - A GUID, generated by GSTV, that uniquely identifies this update.  This will be expected in notifications that refer to this update.
- **updatedOn** - _required_ - A UTC date and time that indicates when this update was created. This can be used to ensure updates are applied in the appropriate order. The format will be `YYYY-MM-DD HH:MM:SS.MS` using 24 hour time.
- **configuration** - _required_ - A complete network configuration that should be used to replace the existing configuration.
    - **shutdownHours** - _required_ - Number of hours a site should continue to play without connectivity. If site goes beyond this number of hours without connectivity, playback will halt at the site.
    - **evergreen** - _required_ - Configuration to control swapping video files when updates to specific files are not received.
      - **filename** - _required_ - Filename - including file extension - of video to control.
      - **expiryHours** - _required_ - The number of hours a video can play before expiring and being swapped out or removed from playback. The timing starts from the time the site receives the video file.
      - **swapFilename** - Filename - including file extension - of video file to play if the expiryHours value has been exceeded. If blank, no video should play in the loop during the spot.
  - **volume** - Array of volume level settings.
    - **startTime** - _if volume array exists this is required_ - The local time represented in HH:MM using 24 hour time that the volume setting will take effect.
    - **level** - _if volume array exists this is required_ - The volume level to be set at the site represented as a number between 0 (no audio) and 100 (highest volume setting).

### Additional Fields in Dover Responses
- **notificationType** - A string indicating the step in the update process to which this notification refers.
  - Values
    - CONFIGURATION_VALIDATION
    - MEDIA_CHECK
    - CONFIGURATION_PULLED_BY_TERMINAL
    - CONFIGURATION_ACCEPTED_BY_TERMINAL
- **notificationTimestamp** - A UTC date and time that indicates when this update was created. This can be used to ensure updates are applied in the appropriate order. The format will be `year-month-day hours:mins:secs.milliseconds` using 24 hour time.
- **status** - A String that will indicate if the step in the update process for which this notification was generated succeeded or failed.
  - Values
    - success
    - failure
- **id** - A unique identifier for this notification, generated by Dover, that will be used to refer to the notification in human communication.
- **additionalInformation** - A free-form String that can be a message or other details about the notification.  This should be human-readable.
- **terminalId** - The id of the terminal at the site.

## Ideal Path
### What GSTV Sends to Dover
```javascript
{
  "guid": String,
  "updatedOn": String,
  "configuration": [
    {
      "shutdownHours": Number,
      "volume": [
        {
          "startTime": String,
          "level": Number
        }
      ],

      "evergreen":[
        {
          "filename": String,
          "expiryHours": Number,
          "swapFilename": String
        },
        {
          "filename": String,
          "expiryHours": Number,
          "swapFilename": String
        },
        {
          "filename": String,
          "expiryHours": Number,
          "swapFilename": String
        },
        {
          "filename": String,
          "expiryHours": Number,
          "swapFilename": String
        },
        {
          "filename": String,
          "expiryHours": Number,
          "swapFilename": String
        }
      ]
    }
  ]
}
```

[What GSTV Sends to Dover - Example Data](samples/json/sample1.json)

### What Dover Sends to GSTV once Network Configuration Updates have been Validated
```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_VALIDATION",
  "status": "success"
}
```

[What Dover Sends to GSTV once Network Configuration Updates have been Validated - Example Data](samples/json/sample1_notification1.json)

### What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified in the Dover Cloud
```javascript
{
  "guid": String,
  "id": String,
  "filename": String,
  "notificationTimestamp": String
  "notificationType": "MEDIA_CHECK",
  "status": "success"
}
```

[What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified in the Dover Cloud - Example Data](samples/json/sample1_notification2.json)

### What Dover Sends to GSTV once Network Configuration Updates have been Pulled by the Terminal
```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_PULLED_BY_TERMINAL",
  "siteId": String,
  "status": "success",
  "terminalId": String
}
```

[What Dover Sends to GSTV once Network Configuration Updates have been Pulled by the Terminal - Example Data](samples/json/sample1_notification3.json)

### What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified at the Terminal
```javascript
{
  "guid": String,
  "id": String,
  "filename": String,
  "notificationTimestamp": String
  "notificationType": "MEDIA_CHECK",
  "siteId": String,
  "status": "success"
  "terminalId": String
}
```

[What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified at the Terminal - Example Data](samples/json/sample1_notification4.json)

### What Dover Sends to GSTV once Network Configuration Updates have been Accepted by the Terminal
```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_ACCEPTED_BY_TERMINAL",
  "siteId": String,
  "status": "success",
  "terminalId": String
}
```

[What Dover Sends to GSTV once Network Configuration Updates have been Accepted by the Terminal - Example Data](samples/json/sample1_notification5.json)

## Error Path
If any part of a message causes an error the entire update will be rejected.

### What Dover Sends if the Network Configuration Update Notification has Validation Errors
Follows [Default Configuration Error Handling](#default-configuration-error-handling).

```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_VALIDATION",
  "status": "failure",
  "additionalInformation": ""
}
```

[What Dover Sends if the Network Configuration Update Notification has Validation Errors - Example Data](samples/json/sample1_notificationfailure1.json)

#### Specific Examples
##### What Dover Sends if the shutdownHours or expiryHours for any File in the Network Configuration Update Notification is not Formatted Correctly
If the shutdownHours or expiryHours is not a number with two or fewer decimal points follow [Default Configuration Error Handling](#default-configuration-error-handling). Additionally, decimals may only be .25, .5 and .75.

Return above structure with this specific value for additionalInformation.

```javascript
  "additionalInformation": "Shutdown Hours is Incorrectly Formatted"
```

```javascript
  "additionalInformation": "Expiry Hours is Incorrectly Formatted for {filename}"
```

##### What Dover Sends if the filename or swapFilename in the Network Configuration Update Notification does not Contain a File Extension
Follow [Default Configuration Error Handling](#default-configuration-error-handling).

Return above structure with this specific value for additionalInformation.

```javascript
  "additionalInformation": "Filename is Missing Extension - {filename}"
```

```javascript
  "additionalInformation": "Swap Filename is Missing Extension - {filename}"
```

##### What Dover Sends if the level for a Volume Object in the Site Configuration Update Notification is not Formatted Correctly
If the volume level is not a number between 0 and 100 follow [Default Configuration Error Handling](#default-configuration-error-handling).

Return above structure with this specific value for additionalInformation.

```javascript
  "additionalInformation": "Volume Level is Incorrectly Formatted"
```

##### What Dover Sends if the startTime for a Volume Level in the Site Configuration Update Notification is not Formatted Correctly
If the startTime is not formatted at HH:MM follow [Default Configuration Error Handling](#default-configuration-error-handling).

Return above structure with this specific value for additionalInformation.

```javascript
  "additionalInformation": "Volume Start Time is Incorrectly Formatted"
```

##### What Dover Sends if the GUID or updatedOn in the Network Configuration Update Notification Does Not Exist
Follow [Default Error Handling](#default-error-handling).

Return above structure with this specific value for additionalInformation.

```javascript
  "additionalInformation": "GUID is Missing"
```

```javascript
  "additionalInformation": "Updated On is Missing"
```

### What Dover Sends if they are Unable to Pull Network Configuration Update Notification to Terminal
Follow [Default Configuration Error Handling](#default-configuration-error-handling).

```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_PULLED_BY_TERMINAL",
  "siteId": String,
  "status": "failure",
  "terminalId": String,
  "additionalInformation": ""
}
```

[What Dover Sends if they are Unable to Pull Network Configuration Update Notification to Terminal - Example Data](samples/json/sample1_notificationfailure3.json)

### What Dover Sends if a Network Configuration Update Notification is Rejected by the Terminal
Follow [Default Configuration Error Handling](#default-configuration-error-handling).

```javascript
{
  "guid": String,
  "id": String,
  "notificationTimestamp": String
  "notificationType": "CONFIGURATION_ACCEPTED_BY_TERMINAL",
  "siteId": String,
  "status": "failure",
  "terminalId": String,
  "additionalInformation": ""
}
```

[What Dover Sends if a Network Configuration Update Notification is Rejected by the Terminal - Example Data](samples/json/sample1_notificationfailure4.json)

## Standard Operating Procedure
### Applying Configuration Updates
1. Poll for network configuration update notification.
1. Validate network configuration update notification.
  - **If success**
    - Follow [What Dover Sends to GSTV once Network Configuration Updates have been Validated](#what-dover-sends-to-gstv-once-network-configuration-updates-have-been-validated).
  - **If failure**
      - Follow [What Dover Sends if the Network Configuration Update Notification has Validation Errors](#what-dover-sends-if-the-network-configuration-update-notification-has-validation-errors).
1. Verify media files associated with network configuration are present Dover Cloud.
  - **If success**
    - Follow [What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified in the Dover Cloud](#what-dover-sends-to-gstv-once-media-files-associated-with-the-network-configuration-updates-have-been-verified-in-the-dover-cloud).
  - **If failure**
    - Follow [What Dover Sends if Media File is not in Dover Cloud](../../README.md#what-dover-sends-if-media-file-is-not-in-dover-cloud).
1. Pull network configuration to terminal.
  - **If success**
    - Follow [What Dover Sends to GSTV once Network Configuration Updates have been Pulled to the Terminal](#what-dover-sends-to-gstv-once-network-configuration-updates-have-been-pulled-to-the-terminal).
  - **If failure**
    - Follow [What Dover Sends if they are Unable to Pull Network Configuration Update Notification to Terminal](#what-dover-sends-if-they-are-unable-to-pull-network-configuration-update-notification-to-terminal).
1. Verify media files associated with network configuration are present at terminal.
  - **If success**
    - Follow [What Dover Sends to GSTV once Media Files Associated with the Network Configuration have been Verified in the Dover Terminal](#what-dover-sends-to-gstv-once-media-files-associated-with-the-network-configuration-have-been-verified-in-the-dover-terminal).
  - **If failure**
    - Follow [File Missing at Terminal](../../README.md#file-missing-at-terminal)
1. Apply network configurations to terminal.
  - **If success**
    - Follow [What Dover Sends to GSTV once Network Configuration Updates have been Accepted by the Terminal](#what-dover-sends-to-gstv-once-network-configuration-updates-have-been-accepted-by-the-terminal).
  - **If failure**
    - Follow [What Dover Sends if a Network Configuration Update Notification is Rejected by the Terminal](#what-dover-sends-if-a-network-configuration-update-notification-is-rejected-by-the-terminal).

### Error Handling
#### Default Error Handling
1. Send a notification message to GSTV alerting that an error has occurred.
1. GSTV sends another notification update message.

#### Default Configuration Error Handling
1. Send a notification message to GSTV alerting that an error has occurred.
1. If a previous configuration exists
  1. Continue to operate using previous configuration until next update
1. If no previous configuration exists
  1. Use default initialization configuration until next update
1. GSTV sends another notification update message.