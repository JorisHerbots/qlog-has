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
    autoplay: boolean
}
~~~

## stream_end
Importance: ??

Emitted when the playback of the media has ended.

Definition:

~~~ cddl
PlaybackStreamEnd = {
    playhead: Playhead
}

Playhead = {
    ms: uint16
    ? frames: uint16
    ? bytes: uint16
}
~~~

## metadata_loaded
Importance: ??

Emitted when the manifest and other metadata is loaded.

Definition:

~~~ cddl
PlaybackMetadataLoaded = {
    protocol: Protocol
    url: string
    ? media_duration: uint16
    ; either a filepath to the manifest on the system
    ; or a stringified representation of the manifest
    ? manifest: string
}

Protocol = 
    "Progressive" /
    "DASH" /
    "HLS" /
    "MSS" /
    ...
~~~

## player_interaction
Importance: ??

Emitted when there is an interaction with the video player.

Definition:

~~~ cddl
PlaybackPlayerInteraction = {
    state: InteractionState
    playhead: Playhead
}

InteractionState = 
    "play" /
    "pause" /
    "seek" /
    "playback_rate" /
    "volume" /
    "resize" 
~~~

## player_state
Importance: ??

Emitted when there is an change of the playback state.

Definition:

~~~ cddl
PlaybackPlayerState = {
    state: PlaybackState
    playhead: Playhead
}

PlaybackState = 
    "playing" /
    "paused" /
    "stopped"
~~~

## stall
Importance: ??

Emitted when the playback stalls

Definition:

~~~ cddl
PlaybackStall = {
    playhead: Playhead
}
~~~

## playhead_progress
Importance: ??

Emitted occasionally when the playhead progresses

Definition:

~~~ cddl
PlaybackPlayheadProgress = {
    playhead: Playhead
}
~~~

## quality_changed
Importance: ??

Emitted when the representation being shown has changed

Definition:

~~~ cddl
PlaybackQualityChanged = {
    media_type: MediaType
    to: Representation
    ? from: Representation
}

Representation = {
    id: string
    ? bitrate: uint16
}

MediaType = 
    "video" /
    "audio" /
    "subtitles" /
    "manifest" /
    "other"
~~~

# Adaptive BitRate events

## RepresentationSwitch
Importance: ??

Emitted when the requested representation of the media changes

Definition:

~~~ cddl
ABRRepresentationSwitch = {
    media_type: MediaType
    to: Representation
    ? from: Representation
}
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
    buffer: Buffer
}

Buffer = {
    level_ms: uint16
    ? level_frames: uint16
    ? level_bytes: uint16
}
~~~

# Network events

## request
Importance: ??

Emitted when a network request is started

Definition:

~~~ cddl
NetworkRequest = {
    media_type: MediaType
    resource_url: string
    ? request_range: string
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
