# App Signal Overview

Signals are the way we can tell an app when something that it cares about happens in a company it is connected to. This section will cover writing a signal in Lessonly, see the [Demux readme](https://github.com/lessonly/demux/blob/master/README.md#signals) for more detailed information on how to use signals in general.

## General Flow

The general flow of sending a signal works like this. Let's say, for example, that a lesson is archived. When it's archived, the `LessonEditingSignal` is send the message `archive` to indicate that the archive status of the lesson has changed. At this point, a background job is triggered that will look for connections between an app and this company that are listening for the "lesson_editing" signal. For each of those connections, we will create a `Demux::Transmission` record and queue another background job to transmit the transmission using a post request to the app's `signal_url`. The result of the transmission will be saved back to the transmission record for later inspection. Any 2xx response will be considered successful, but its typical to return a 202 accepted.

## Timeout

A signal will timeout if it spends more than 10 seconds attempting to connect to the app or waiting on a response from the app. This period is intentionally very short because any app should not process the signal inline, but should receive the signal and process it without blocking the response back to Lessonly. We don't want to spend a lot of our background job processor time waiting for a response from an app.

## De-duplication

To avoid a flood of transmission backing up, signal transmissions that are identical will be sent as only one. We guarantee that you will receive at least one transmission for a unique action and that the transmission sent will contain the most recent state of the object in question.

Here is a concrete example. Let's say that a lesson is archived/unarchived 5 times very quickly so that it all happens before the first transmission can be sent.

First of all, notice that the signal does not have separate actions based on state; so there is not a "archive" and "unarchived" action, but just a single "archive" action for both. The partner app needs to know the final state of the object, and if an action represents state then the order you receive the signals in becomes important. The single "archive" action represents any change in the archive status of the lesson.

The `LessonEditingSignal#archive` action will be called 5 times with the same arguments (same lesson ID and account ID) so they are considered identical. Because all 5 calls happened before the first transmission was sent, there will not be 5 transmission records queued for sending but only 1 because of de-duplication.

When that 1 transmission is processed, it will retrieve the lesson record in question and build a payload based on it's current status and then send that. So, although only one transmission is sent, it represents the latest state of the lesson. No signals will be sent for all the toggling in between the starting state and ending state.

One exception to this is context. Context, when passed to an action, is a part of what makes a transmission unique. If two signals are triggered with identical arguments but different context, they will not be de duplicated. If the context matches exactly along with all the arguments, they will be eligible for de de-duplication. Where there is unique context, we want to preserve that and not have it lost.

## Signal Creation

Signals are defined in the `app/signals/` directory. A signal contains one or more actions that are grouped around an object and use case. Apps can choose to subscribe to signals that fit their use case. Keep this in mind when writing a signal; any given signal should attempt to represent a grouping of actions that we would imagine would be useful together given common use cases.

For example, if an app is concerned with when a lesson is edited, it may not also be concerned with when a user completes a lesson or when a user is added to that company, so we wouldn't want to have those actions contained in the same signal most likely.

All signal names will end in `Signal`, for example `LessonEditingSignal`.

In general it's best to process a record lazily when possible. What this means is that we should use `context` sparingly when creating our signals and allow as much as possible to be retrieved when building the payload during a transmission.

# Signals Reference
