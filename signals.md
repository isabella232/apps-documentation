# App Signal Overview

Signals are the way we can tell an app when something that it cares about happens in a company it is connected to. This section will cover using signals in a Lessonly App, see the [Demux readme](https://github.com/lessonly/demux/blob/master/README.md#signals) for more detailed information on creating signals and the demux system the Lessonly Apps is built on.

## General Flow

When a signal in triggered in a company, Lessonly will look for any Lessonly Apps that are connected to that company and that are listening for that signal. For all apps that are found to have a connection listening to the company listening for that signal, Lessonly will transmit the signal to the app. The Lessonly app should receive the signal promptly and return a 2xx response to indicate success (202 accepted is a good choice). All processing of the signal should be done in the background to avoid blocking a response.

## Timeout

A signal will timeout if it spends more than 10 seconds attempting to connect to the app or waiting on a response from the app. This period is intentionally very short because any app should not process the signal inline, but should receive the signal and process it without blocking the response back to Lessonly. We don't want to spend a lot of our background job processor time waiting for a response from an app.

## De-duplication

To avoid a flood of transmission backing up, signal transmissions that are identical will be sent as only one. We guarantee that you will receive at least one transmission for a unique action and that the transmission sent will contain the most recent state of the object in question.

Here is a concrete example. Let's say that a lesson is archived/unarchived 5 times very quickly so that it all happens before the first transmission can be sent.

First of all, notice that the signal does not have separate actions based on state; so there is not a "archive" and "unarchived" action, but just a single "archive" action for both. The partner app needs to know the final state of the object, and if an action represents state then the order you receive the signals in becomes important. The single "archive" action represents any change in the archive status of the lesson.

Because all those events happened before the transmission had time to send, the partner app will only end up receiving a single transmission. This transmission will contain the state of the lesson after the final time the lesson's archive states was changed (the most current state).

# Signals Reference

## Common Headers

Every signal will be sent with a common set of headers related to the transmission:

```
"user-agent"=>"Demux"
"content-type"=>"application/json"
"content-length"=>"96"
"x-demux-signal"=>"lesson_editing"
"x-demux-signature"=>"0ef61eb9e64ab8f6f379540abedf1999a94b977c8555db4ed77e9f3c924962db"
```
*user-agent* - will always be "Demux"

*x-demux-signal* - contains the name of the signal that's being sent

*x-demux-signature* - contains the signal body signed using the app's secret. You should verify the body of the signal using this upon receipt.

## Signature

The signature provided in the x-demux-signature header should be used to verify the contents of the signal before using it.

The signature is create by taking the signal body and hashing it using HMAC and the "SHA256" algorithm.

Here is an example in ruby for checking the signature:

```Ruby
received_signature == OpenSSL::HMAC.hexdigest("SHA256", app_secret, request_body)
```

## lesson_editing

### Actions

*archive* - The archive action represents when the archive status of a lesson changes.

### Payload Example

```JSON
{
  "action": "archive"
  "company_id": Integer,
  "lesson": {
    "id": Integer,
    "archived": Boolean,
    "archived_at": String
  }
}
```
