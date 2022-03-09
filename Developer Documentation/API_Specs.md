# Notification Service

The notification service provides a mechanism to allow sending messages sent from server or client to another client.

## Overview
The notification service uses point-to-point messaging.

### Services
The Notification service defines three interfaces:

* bnet.protocol.notification.v2.client.NotificationService: The client portion of the notification service that supports sending notifications to a client.
* bnet.protocol.notification.v2.client.NotificationListener: A listener object for receiving notification events.
* bnet.protocol.notification.v2.server.NotificationService: The server portion of the notification service that supports sending notifications to a client.

### Components
The following components diagram outlines the actors for the Notification service.
![](/images/plantuml8155743692218351961.png)

### SDK Documentation
The BGS SDK package comes with a PDF of auto-generated API documentation. Each time you download the package, you'll find a current version of the PDF in the ```\doc``` folder.

## Use cases and examples
Example cases for Notification include:
* A player sending another player an invitation to spectate
* Notifying a user that their account inventory has a new item
* Notify a player in WOW that their subscription will expire in X days

## Integration Guide
The following integration guide documents details on each step of implementing notifications.

### SendNotification
The SendNotification API is designed for a client to deliver a notification to other clients. This allows a point-to-point message to be sent from a server or client to another client. Before calling this method, the client must be successfully created first.

Please note that the client doesn't need to be subscribed in order to receive messages, however the listener interface must be implemented to receive messages. Notification delivery is also not guaranteed, as the sender does not receive any feedback if the message was successfully delivered. Also, notifications are not persistent, so if the target client is not currently connected, delivery of the notification will fail.

#### Client to Client

The notification service provides a mechanism to allow sending messages sent from server or client to another client.

There are two types of delivery method which are point-to-point notification and subscription based server publication. In classic notification service, most message/delivery validation is done in the proxy level and the subscribers are stored in the Memcache, however the subscribers will be stored in the persistent storage in HA since the proxy will not have any state.

In V2 client protocol, client protocol is broken to two part, Notification and Publication service. Notification takes point-to-point messaging features and Publication allows a client to subscribe to one or more topics for a set of identity values.

## SendNotification

SendNotification API is designed for client to deliver a notification to other clients. This allows point-to-point message sent from server or client to another client. A client should have done the session creation and you will get the information through API in the Session service.

It doesn't need to subscribe in order to receive the messages. Notification delivery is not guaranteed. The sender does not receive any feedback as to weather of not the message was actually delivered. Notifications are not persistent. If the target client is not currently connected, delivery of the notification will fail.

![](/images/plantuml3642796119711329604.png)

## Client V2 Protocol

### V1 - OnNotificationReceived

```
message Notification
{
  optional EntityId sender_id = 1;
  required EntityId target_id = 2;
  required string type = 3;
  repeated Attribute attribute = 4;
  optional EntityId sender_account_id = 5;
  optional EntityId target_account_id = 6;
  optional string sender_battle_tag = 7 [(field_options).log = HIDDEN];
  optional string target_battle_tag = 8 [(field_options).log = HIDDEN];
  optional account.v1.Identity forwarding_identity = 10;
}
```

### V2 - NotificationReceivedNotification

```
message NotificationReceivedNotification
{
  repeated Notification notifications = 1;
  {
    optional string type = 1 [(valid).string.size = {min:1, max:512}];
    optional UserDescription sender = 2;
    {
      optional uint64 account_id = 1;
    }
    optional UserDescription target = 3;
    {
      optional uint64 account_id = 1;
    }
    repeated protocol.v2.Attribute attribute = 4;
    {
      optional string name = 1;
      optional Variant value = 2;
    }
    optional uint64 creation_time_ms = 5;
  }
}
```

### V2 - PublicationReceivedNotification

```
message PublicationReceivedNotification
{
  repeated Article articles = 1;
  {
    optional Target target = 1;
    {
      optional string topic_name = 1 [(valid).string.size = {min:1, max:512}];
      optional string identity = 2 [(valid).string.size = {min:1, max:512}];
    }
    repeated protocol.v2.Attribute attribute = 2;
    {
      optional string name = 1;
      optional Variant value = 2;
    }
    optional uint64 creation_time_ms = 3;
  }
}
```

*   There are two separated event messages in protocol V2.
    *   NotificationListener: A listener for receiving notification events sent from SendNotification API
    *   PublicationListener: A listener for receiving publication events sent from Publish API

## Configuration

Notification configuration used to define various limits on the Notification service. NotificationLimits limits number of the notification messages in the specific interval or payload or targets. NotificationBehavior provides specific behaviors for notification types. Each flag is used to check that the sender or receiver or message itself is appropriate for the delivery.

*   These are the notification types registered in Production:
    * WHISPER
    * WHISPER\_ECHO
    * AccountInventoryNewItem
    * TYPING
    * WTCG.SpectatorInvite
    * DST2.NotificationMessage
    * VIPR.Invitation
    * App.WhisperVoiceNotification
    * D3.NotificationMessage
    * WoW.Gamedata
    * WTCG.UtilNotificationMessage
    * BSAp.NotificationMessage
