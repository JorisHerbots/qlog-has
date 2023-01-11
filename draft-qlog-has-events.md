# Introduction

This document describes the values of the qlog name ("category" + "event") and
"data" fields and their semantics for HTTP Adaptive Streaming (HAS).

## Notational Conventions

{::boilerplate bcp14-tagged}

The event and data structure definitions in ths document are expressed
in the Concise Data Definition Language {{!CDDL=RFC8610}} and its
extensions described in {{QLOG-MAIN}}.

The following fields from {{QLOG-MAIN}} are imported and used: name, category,
type, data, group_id, protocol_type, importance, RawInfo, and time-related
fields.

# Overview

This document describes how HAS can be expressed in qlog using
the schema defined in {{QLOG-MAIN}}. HAS events are defined with a
category, a name (the concatenation of "category" and "event"), an "importance",
an optional "trigger", and "data" fields.

Some data fields use complex datastructures. These are represented as enums or
re-usable definitions, which are grouped together on the bottom of this document
for clarity.

When any event from this document is included in a qlog trace, the
"protocol_type" qlog array field MUST contain an entry with the value "HAS".

# Playback events

## stream_initialised
Importance: ??

Emitted when the stream is initialised.

Definition:

~~~ cddl
PlaybackStreamInitialised = {
    protocol: string
    url: string
    autoplay: boolean

    ? duration: uint16
    ; filepath to manifest
    ? manifest: string
}
~~~

## player_interaction
Importance: ??

Emitted when there is an interaction with the video player.

Definition:

~~~ cddl
PlaybackPlayerInteraction = {
    state: InteractionState
    playhead_ms: uint16
    ? playhead_frame: uint16
    ? playhead
}

InteractionState = 
    "play" /
    "pause" /
    "seek" /
    "speed" /
    "volume" /
    "resize" 
~~~

## rebuffer
Importance: ??

Emitted when the playback stalls

Definition:

~~~ cddl
PlaybackRebuffer = [
    playhead_ms: uint16
    ? playhead_frame: uint16
]
~~~

## stream_end
Importance: ??

Emitted when //

Definition:

~~~ cddl
PlaybackStreamEnd = {
    playhead_ms: uint16
    ? playhead_frame: uint16
}
~~~

## playhead_progress
Importance: ??

Emitted occasionally when the playhead progresses

Definition:

~~~ cddl
PlaybackPlayheadProgress = {
    playhead_ms: uint16
    ? playhead_frame: uint16
    ? playhead_bytes: uint16
}
~~~

## quality_changed
Importance: ??

Emitted when //

Definition:

~~~ cddl

~~~

# Adaptive BitRate events

## switch
Importance: ??

Emitted when the representation of the media changes

Definition:

~~~ cddl
ABRSwitch = {
    media_type: MediaType
    ? from_id: string
    ? from_bitrate: uint16
    to_id: string
    ? to_bitrate: uint16
}

MediaType = 
    "video" /
    "audio" /
    "subtitles" /
    "other"
~~~

## readystate_change
Importance: ??

Emitted when the readiness of the media potentially changed

Definition:

~~~ cddl
ABRReadyStateChange = {
    state: ReadyState
}

ReadyState =
    "have_nothing" /
    "have_metadata" /
    "have_current_data" /
    "have_future_data" /
    "have_enough_data"
~~~

## metrics_updated
Importance: ??

Logs the metrics of the ABR algorithm

Definition:

~~~ cddl
ABRStreamMetrics = {
    ? min_rtt: uint16
    ? smoothed_rtt: uint16
    ? latest_rtt: uint16
    ? rtt_variance: uint16

    ? bitrate: uint16

    ? dropped_frames: uint16
}
~~~

# Buffer events

## occupancy_update
Importance: ??

Emitted when the occupancy of a buffer changes

Definition:

~~~ cddl
BufferOccupancy = {
    media_type: mediaType
    playout_ms: uint16
    ? playout_bytes: uint16
    ? playout_frames: uint16
}
~~~

# Network events

## request
Importance: ??

Emitted when a network request is started

Definition:

~~~ cddl
NetworkRequest = {
    resource_url: string
    ? range: string
    media_type: MediaType
}
~~~

## request_update
Importance: ??

Emitted when a network request made progress

Definition:

~~~ cddl
NetworkRequestUpdate = {
    resource_url: string
    bytes_received: uint16
    ? rtt: uint16
}
~~~

## abort
Importance: ??

Emitted when a network request is aborted

Definition:

~~~ cddl
NetworkAbort = {
    resource_url: string
}
~~~