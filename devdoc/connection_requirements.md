#connection requirements
 
##Overview

connection is module that implements the connection layer in the AMQP ISO.

##Exposed API

```C
	typedef void* CONNECTION_HANDLE;

	typedef enum CONNECTION_STATE_TAG
	{
		CONNECTION_STATE_START,
		CONNECTION_STATE_HDR_RCVD,
		CONNECTION_STATE_HDR_SENT,
		CONNECTION_STATE_HDR_EXCH,
		CONNECTION_STATE_OPEN_PIPE,
		CONNECTION_STATE_OC_PIPE,
		CONNECTION_STATE_OPEN_RCVD,
		CONNECTION_STATE_OPEN_SENT,
		CONNECTION_STATE_CLOSE_PIPE,
		CONNECTION_STATE_OPENED,
		CONNECTION_STATE_CLOSE_RCVD,
		CONNECTION_STATE_CLOSE_SENT,
		CONNECTION_STATE_DISCARDING,
		CONNECTION_STATE_END
	} CONNECTION_STATE;

	typedef void(*ON_ENDPOINT_FRAME_RECEIVED)(void* context, AMQP_VALUE performative, uint32_t frame_payload_size, const unsigned char* payload_bytes);
	typedef void(*ON_CONNECTION_STATE_CHANGED)(void* context, CONNECTION_STATE new_connection_state, CONNECTION_STATE previous_connection_state);

	extern CONNECTION_HANDLE connection_create(XIO_HANDLE xio, const char* hostname, const char* container_id);
	extern int connection_set_max_frame_size(CONNECTION_HANDLE connection, uint32_t max_frame_size);
	extern int connection_get_max_frame_size(CONNECTION_HANDLE connection, uint32_t* max_frame_size);
	extern int connection_set_channel_max(CONNECTION_HANDLE connection, uint16_t channel_max);
	extern int connection_get_channel_max(CONNECTION_HANDLE connection, uint16_t* channel_max);
	extern int connection_set_idle_timeout(CONNECTION_HANDLE connection, milliseconds idle_timeout);
	extern int connection_get_idle_timeout(CONNECTION_HANDLE connection, milliseconds* idle_timeout);
	extern int connection_get_remote_max_frame_size(CONNECTION_HANDLE connection, uint32_t* remote_max_frame_size);
	extern void connection_destroy(CONNECTION_HANDLE connection);
	extern void connection_dowork(CONNECTION_HANDLE connection);
	extern ENDPOINT_HANDLE connection_create_endpoint(CONNECTION_HANDLE connection, ON_ENDPOINT_FRAME_RECEIVED on_frame_received, ON_CONNECTION_STATE_CHANGED on_connection_state_changed, void* context);
	extern void connection_destroy_endpoint(ENDPOINT_HANDLE endpoint);
	extern int connection_encode_frame(ENDPOINT_HANDLE endpoint, const AMQP_VALUE performative, PAYLOAD* payloads, size_t payload_count);
```

###connection_create

```C
extern CONNECTION_HANDLE connection_create(XIO_HANDLE xio, const char* container_id);
```

**SRS_CONNECTION_01_001: [**connection_create shall open a new connection to a specified host/port.**]**
**SRS_CONNECTION_01_071: [**If xio or container_id is NULL, connection_create shall return NULL.**]** 
**SRS_CONNECTION_01_082: [**connection_create shall allocate a new frame_codec instance to be used for frame encoding/decoding.**]** 
**SRS_CONNECTION_01_083: [**If frame_codec_create fails then connection_create shall return NULL.**]** 
**SRS_CONNECTION_01_107: [**connection_create shall create an amqp_frame_codec instance by calling amqp_frame_codec_create.**]** 
**SRS_CONNECTION_01_108: [**If amqp_frame_codec_create fails, connection_create shall return NULL.**]** 
**SRS_CONNECTION_01_072: [**When connection_create succeeds, the state of the connection shall be CONNECTION_STATE_START.**]** 
**SRS_CONNECTION_01_081: [**If allocating the memory for the connection fails then connection_create shall return NULL.**]** 
**SRS_CONNECTION_22_002: [**connection_create shall allow registering connections state and io error callbacks.**]** 
**SRS_CONNECTION_22_001: [**If a connection state changed occurs and a callback is registered the callback shall be called.**]** 
**SRS_CONNECTION_22_005: [**If the io notifies the connection instance of an IO_STATE_ERROR state and an io error callback is registered, the connection shall call the registered callback.**]**
**SRS_CONNECTION_22_003: [**connection_create shall allow registering a custom logger instead of default console logger.**]** 
**SRS_CONNECTION_22_004: [**If no logger is provided, log messages are sent to console_logger.**]** 

###connection_set_max_frame_size

```C
extern int connection_set_max_frame_size(CONNECTION_HANDLE connection, uint32_t max_frame_size);
```

**SRS_CONNECTION_01_148: [**connection_set_max_frame_size shall set the max_frame_size associated with a connection.**]** 
**SRS_CONNECTION_01_149: [**On success connection_set_max_frame_size shall return 0.**]** 
**SRS_CONNECTION_01_163: [**If connection is NULL, connection_set_max_frame_size shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_150: [**If the max_frame_size is invalid then connection_set_max_frame_size shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_152: [**If frame_codec_set_max_frame_size fails then connection_set_max_frame_size shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_157: [**If connection_set_max_frame_size is called after the initial Open frame has been sent, it shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_164: [**If connection_set_max_frame_size fails, the previous max_frame_size setting shall be retained.**]** 

###connection_get_max_frame_size

```C
int connection_get_max_frame_size(CONNECTION_HANDLE connection, uint32_t* max_frame_size);
```

**SRS_CONNECTION_01_168: [**connection_get_max_frame_size shall return in the max_frame_size argument the current max frame size setting.**]** 
**SRS_CONNECTION_01_169: [**On success, connection_get_max_frame_size shall return 0.**]** 
**SRS_CONNECTION_01_170: [**If connection or max_frame_size is NULL, connection_get_max_frame_size shall fail and return a non-zero value.**]** 

###connection_set_channel_max

```C
extern int connection_set_channel_max(CONNECTION_HANDLE connection, uint16_t channel_max);
```

**SRS_CONNECTION_01_153: [**connection_set_channel_max shall set the channel_max associated with a connection.**]** 
**SRS_CONNECTION_01_154: [**On success connection_set_channel_max shall return 0.**]** 
**SRS_CONNECTION_01_181: [**If connection is NULL then connection_set_channel_max shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_156: [**If connection_set_channel_max is called after the initial Open frame has been sent, it shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_165: [**If connection_set_channel_max fails, the previous channel_max setting shall be retained.**]** 
**SRS_CONNECTION_01_257: [**connection_set_channel_max shall fail if channel_max is higher than the outgoing channel number of any already created endpoints.**]** 

###connection_get_channel_max

```C
extern int connection_get_channel_max(CONNECTION_HANDLE connection, uint16_t* channel_max);
```

**SRS_CONNECTION_01_182: [**connection_get_channel_max shall return in the channel_max argument the current channel_max setting.**]** 
**SRS_CONNECTION_01_183: [**On success, connection_get_channel_max shall return 0.**]** 
**SRS_CONNECTION_01_184: [**If connection or channel_max is NULL, connection_get_channel_max shall fail and return a non-zero value.**]** 

###connection_set_idle_timeout

```C
extern int connection_set_idle_timeout(CONNECTION_HANDLE connection, milliseconds idle_timeout);
```

**SRS_CONNECTION_01_159: [**connection_set_idle_timeout shall set the idle_timeout associated with a connection.**]** 
**SRS_CONNECTION_01_160: [**On success connection_set_idle_timeout shall return 0.**]** 
**SRS_CONNECTION_01_191: [**If connection is NULL, connection_set_idle_timeout shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_158: [**If connection_set_idle_timeout is called after the initial Open frame has been sent, it shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_166: [**If connection_set_idle_timeout fails, the previous idle_timeout setting shall be retained.**]** 

###connection_get_idle_timeout

```C
extern int connection_get_idle_timeout(CONNECTION_HANDLE connection, milliseconds* idle_timeout);
```

**SRS_CONNECTION_01_188: [**connection_get_idle_timeout shall return in the idle_timeout argument the current idle_timeout setting.**]** 
**SRS_CONNECTION_01_189: [**On success, connection_get_idle_timeout shall return 0.**]** 
**SRS_CONNECTION_01_190: [**If connection or idle_timeout is NULL, connection_get_idle_timeout shall fail and return a non-zero value.**]** 

###connection_get_remote_max_frame_size

```C
extern int connection_get_remote_max_frame_size(CONNECTION_HANDLE connection, uint32_t* remote_max_frame_size);
```

###connection_destroy

```C
extern void connection_destroy(CONNECTION_HANDLE handle);
```

**SRS_CONNECTION_01_073: [**connection_destroy shall free all resources associated with a connection.**]** 
**SRS_CONNECTION_01_074: [**connection_destroy shall close the socket connection.**]** 
**SRS_CONNECTION_01_075: [**If an Open frame has been sent then a Close frame shall be sent before closing the socket.**]** 
**SRS_CONNECTION_01_079: [**If handle is NULL, connection_destroy shall do nothing.**]** 

###connection_dowork

```C
extern void connection_dowork(CONNECTION_HANDLE connection);
```

**SRS_CONNECTION_01_076: [**connection_dowork shall schedule the underlying IO interface to do its work by calling xio_dowork.**]** 
**SRS_CONNECTION_01_078: [**If handle is NULL, connection_dowork shall do nothing.**]** 
**SRS_CONNECTION_01_084: [**The connection state machine implementing the protocol requirements shall be run as part of connection_dowork.**]** 
**SRS_CONNECTION_01_202: [**If the io notifies the connection instance of an IO_STATE_ERROR state the connection shall be closed and the state set to END.**]** 
**SRS_CONNECTION_01_104: [**Sending the protocol header shall be done by using xio_send.**]** 
**SRS_CONNECTION_01_106: [**When sending the protocol header fails, the connection shall be immediately closed.**]** 
**SRS_CONNECTION_01_151: [**The connection max_frame_size setting shall be passed down to the frame_codec when the Open frame is sent.**]** 
**SRS_CONNECTION_01_207: [**If frame_codec_set_max_frame_size fails the connection shall be closed and the state set to END.**]** 

###connection_create_endpoint

```C
extern ENDPOINT_HANDLE connection_create_endpoint(CONNECTION_HANDLE connection, ON_ENDPOINT_FRAME_RECEIVED on_endpoint_frame_received, ON_CONNECTION_STATE_CHANGED on_connection_state_changed, void* context);
```

**SRS_CONNECTION_01_112: [**connection_create_endpoint shall create a new endpoint that can be used by a session.**]** 
**SRS_CONNECTION_01_127: [**On success, connection_create_endpoint shall return a non-NULL handle to the newly created endpoint.**]** 
**SRS_CONNECTION_01_113: [**If connection, on_endpoint_frame_received or on_connection_state_changed is NULL, connection_create_endpoint shall fail and return NULL.**]** 
**SRS_CONNECTION_01_193: [**The context argument shall be allowed to be NULL.**]** 
**SRS_CONNECTION_01_115: [**If no more endpoints can be created due to all channels being used, connection_create_endpoint shall fail and return NULL.**]** 
**SRS_CONNECTION_01_128: [**The lowest number outgoing channel shall be associated with the newly created endpoint.**]** 
**SRS_CONNECTION_01_196: [**If memory cannot be allocated for the new endpoint, connection_create_endpoint shall fail and return NULL.**]** 
**SRS_CONNECTION_01_197: [**The newly created endpoint shall be added to the endpoints list, so that it can be tracked.**]** 
**SRS_CONNECTION_01_198: [**If adding the endpoint to the endpoints list tracked by the connection fails, connection_create_endpoint shall fail and return NULL.**]** 

###connection_destroy_endpoint

```C
extern void connection_destroy_endpoint(ENDPOINT_HANDLE endpoint);
```

**SRS_CONNECTION_01_129: [**connection_destroy_endpoint shall free all resources associated with an endpoint created by connection_create_endpoint.**]** 
**SRS_CONNECTION_01_130: [**The outgoing channel associated with the endpoint shall be released by removing the endpoint from the endpoint list.**]** 
**SRS_CONNECTION_01_131: [**Any incoming channel number associated with the endpoint shall be released.**]** 
**SRS_CONNECTION_01_199: [**If endpoint is NULL, connection_destroy_endpoint shall do nothing.**]** 

###connection_encode_frame

```C
extern int connection_encode_frame(ENDPOINT_HANDLE endpoint, const AMQP_VALUE performative, PAYLOAD* payloads, size_t payload_count);
```

**SRS_CONNECTION_01_247: [**connection_encode_frame shall send a frame for a certain endpoint.**]** 
**SRS_CONNECTION_01_248: [**On success it shall return 0.**]** 
**SRS_CONNECTION_01_249: [**If endpoint or performative are NULL, connection_encode_frame shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_250: [**connection_encode_frame shall initiate the frame send by calling amqp_frame_codec_begin_encode_frame.**]** 
**SRS_CONNECTION_01_251: [**The channel number passed to amqp_frame_codec_begin_encode_frame shall be the outgoing channel number associated with the endpoint by connection_create_endpoint.**]** 
**SRS_CONNECTION_01_252: [**The performative passed to amqp_frame_codec_begin_encode_frame shall be the performative argument of connection_encode_frame.**]** 
**SRS_CONNECTION_01_255: [**The payload size shall be computed based on all the payload chunks passed as argument in payloads.**]** 
**SRS_CONNECTION_01_253: [**If amqp_frame_codec_begin_encode_frame or amqp_frame_codec_encode_payload_bytes fails, then connection_encode_frame shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_254: [**If connection_encode_frame is called before the connection is in the OPENED state, connection_encode_frame shall fail and return a non-zero value.**]** 
**SRS_CONNECTION_01_256: [**Each payload passed in the payloads array shall be passed to amqp_frame_codec by calling amqp_frame_codec_encode_payload_bytes.**]** 

###on_connection_state_changed

**SRS_CONNECTION_01_258: [**on_connection_state_changed shall be invoked whenever the connection state changes.**]** 
**SRS_CONNECTION_01_260: [**Each endpoint's on_connection_state_changed shall be called.**]** 
**SRS_CONNECTION_01_259: [**As context, the callback_context passed in connection_create_endpoint shall be given.**]** 

##Frame construction

###Open frame construction

**SRS_CONNECTION_01_134: [**The container id field shall be filled with the container id specified in connection_create.**]** 
**SRS_CONNECTION_01_135: [**If hostname has been specified by a call to connection_set_hostname, then that value shall be stamped in the open frame.**]** 
**SRS_CONNECTION_01_136: [**If no hostname value has been specified, no value shall be stamped in the open frame (no call to open_set_hostname shall be made).**]** 
**SRS_CONNECTION_01_137: [**If max_frame_size has been specified by a call to connection_set_max_frame, then that value shall be stamped in the open frame.**]** 
**SRS_CONNECTION_01_139: [**If channel_max has been specified by a call to connection_set_channel_max, then that value shall be stamped in the open frame.**]** 
**SRS_CONNECTION_01_141: [**If idle_timeout has been specified by a call to connection_set_idle_timeout, then that value shall be stamped in the open frame.**]** 
**SRS_CONNECTION_01_142: [**If no idle_timeout value has been specified, no value shall be stamped in the open frame (no call to open_set_idle_time_out shall be made).**]** 
**SRS_CONNECTION_01_205: [**Sending the AMQP OPEN frame shall be done by calling amqp_frame_codec_begin_encode_frame with channel number 0, the actual performative payload and 0 as payload_size.**]** 
**SRS_CONNECTION_01_206: [**If sending the frame fails, the connection shall be closed and state set to END.**]** 
**SRS_CONNECTION_01_208: [**If the open frame cannot be constructed, the connection shall be closed and set to the END state.**]** 

###Close frame construction

**SRS_CONNECTION_01_217: [**The CLOSE frame shall be constructed by using close_create.**]** 
**SRS_CONNECTION_01_215: [**Sending the AMQP CLOSE frame shall be done by calling amqp_frame_codec_begin_encode_frame with channel number 0, the actual performative payload and 0 as payload_size.**]** 
**SRS_CONNECTION_01_214: [**If the close frame cannot be constructed or sent, the connection shall be closed and set to the END state.**]**

##Frame reception

**SRS_CONNECTION_01_212: [**After the initial handshake has been done all bytes received from the io instance shall be passed to the frame_codec for decoding by calling frame_codec_receive_bytes.**]** 
**SRS_CONNECTION_01_213: [**When passing the bytes to frame_codec fails, a CLOSE frame shall be sent and the state shall be set to DISCARDING.**]** 
**SRS_CONNECTION_01_218: [**The error amqp:internal-error shall be set in the error.condition field of the CLOSE frame.**]** 
**SRS_CONNECTION_01_219: [**The error description shall be set to an implementation defined string.**]** 
**SRS_CONNECTION_01_223: [**If the amqp_frame_received_callback is called with a NULL performative then the connection shall be closed with the error condition amqp:internal-error and an implementation defined error description.**]** 
**SRS_CONNECTION_01_241: [**The connection module shall accept OPEN frames even if they have extra payload bytes besides the Open performative.**]** 
**SRS_CONNECTION_01_242: [**The connection module shall accept CLOSE frames even if they have extra payload bytes besides the Close performative.**]** 

###Open frame reception

**SRS_CONNECTION_01_143: [**If any of the values in the received open frame are invalid then the connection shall be closed.**]** 
**SRS_CONNECTION_01_220: [**The error amqp:invalid-field shall be set in the error.condition field of the CLOSE frame.**]** 
**SRS_CONNECTION_01_221: [**The error description shall be set to an implementation defined string.**]** 
**SRS_CONNECTION_01_222: [**If an Open frame is received in a manner violating the ISO specification, the connection shall be closed with condition amqp:not-allowed and description being an implementation defined string.**]** 
**SRS_CONNECTION_01_239: [**If an Open frame is received in the Opened state the connection shall be closed with condition amqp:illegal-state and description being an implementation defined string.**]** 

###Begin frame reception

**SRS_CONNECTION_01_144: [**When a begin frame is received, the connection shall only look at the remote_channel field and if the remote channel field matches one of the outgoing channels of an already created endpoint, then the channel number on which the Begin frame was received shall be assigned as incoming channel number for the endpoint.**]** 
**SRS_CONNECTION_01_145: [**If the endpoint already has a channel number assigned then this shall be considered a protocol violation and the connection shall be closed.**]** 
**SRS_CONNECTION_01_146: [**If no endpoint can be found for the specified remote channel, this shall be considered a protocol violation and the connection shall be closed.**]** 

Not implemented: outgoing_locales, incoming_locales, offered_capabilities, desired_capabilities, properties.

##Connection ISO section

2.2 Version Negotiation

**SRS_CONNECTION_01_086: [**Prior to sending any frames on a connection, each peer MUST start by sending a protocol header that indicates the protocol version used on the connection.**]** 
**SRS_CONNECTION_01_087: [**The protocol header consists of the upper case ASCII letters "AMQP" followed by a protocol id of zero, followed by three unsigned bytes representing the major, minor, and revision of the protocol version (currently 1 (MAJOR), 0 (MINOR), 0 (REVISION)). In total this is an 8-octet sequence**]**:

...

Figure 2.11: Protocol Header Layout
**SRS_CONNECTION_01_088: [**Any data appearing beyond the protocol header MUST match the version indicated by the protocol header.**]** 
**SRS_CONNECTION_01_089: [**If the incoming and outgoing protocol headers do not match, both peers MUST close their outgoing stream**]** 
**SRS_CONNECTION_01_090: [**and SHOULD read the incoming stream until it is terminated.**]**
**SRS_CONNECTION_01_091: [**The AMQP peer which acted in the role of the TCP client (i.e. the peer that actively opened the connection) MUST immediately send its outgoing protocol header on establishment of the TCP connection.**]** 
**SRS_CONNECTION_01_092: [**The AMQP peer which acted in the role of the TCP server MAY elect to wait until receiving the incoming protocol header before sending its own outgoing protocol header. This permits a multi protocol server implementation to choose the correct protocol version to fit each client.**]**
Two AMQP peers agree on a protocol version as follows (where the words "client" and "server" refer to the roles being played by the peers at the TCP connection level):
**SRS_CONNECTION_01_093: [**_ When the client opens a new socket connection to a server, it MUST send a protocol header with the client's preferred protocol version.**]** 
**SRS_CONNECTION_01_094: [**_ If the requested protocol version is supported, the server MUST send its own protocol header with the requested version to the socket, and then proceed according to the protocol definition.**]** 
**SRS_CONNECTION_01_095: [**_ If the requested protocol version is not supported, the server MUST send a protocol header with a supported protocol version and then close the socket.**]** 
**SRS_CONNECTION_01_096: [**_ When choosing a protocol version to respond with, the server SHOULD choose the highest supported version that is less than or equal to the requested version.**]** 
**SRS_CONNECTION_01_097: [**If no such version exists, the server SHOULD respond with the highest supported version.**]** 
**SRS_CONNECTION_01_098: [**_ If the server cannot parse the protocol header, the server MUST send a valid protocol header with a supported protocol version and then close the socket.**]** 
**SRS_CONNECTION_01_099: [**_ Note that if the server only supports a single protocol version, it is consistent with the above rules for the server to send its protocol header prior to receiving anything from the client and to subsequently close the socket if the client's protocol header does not match the server's.**]** 
**SRS_CONNECTION_01_100: [**Based on this behavior a client can discover which protocol versions a server supports by attempting to connect with its highest supported version and reconnecting with a version less than or equal to the version received back from the server.**]** 

...

Figure 2.12: Version Negotiation Examples
Please note that the above examples use the literal notation defined in RFC 2234 [RFC2234] for non alphanumeric values.

**SRS_CONNECTION_01_101: [**The protocol id is not a part of the protocol version and thus the rule above regarding the highest supported version does not apply.**]** 
**SRS_CONNECTION_01_102: [**A client might request use of a protocol id that is unacceptable to a server - for example, it might request a raw AMQP connection when the server is configured to require a TLS or SASL security layer (See Part 5: section 5.1). In this case, the server MUST send a protocol header with an acceptable protocol id (and version) and then close the socket.**]** 
**SRS_CONNECTION_01_103: [**It MAY choose any protocol id.**]** 

...

Figure 2.13: Protocol ID Rejection Example

2.4 Connections

**SRS_CONNECTION_01_061: [**AMQP connections are divided into a number of unidirectional channels.**]** 
**SRS_CONNECTION_01_062: [**A connection endpoint contains two kinds of channel endpoints: incoming and outgoing.**]** 
**SRS_CONNECTION_01_063: [**A connection endpoint maps incoming frames other than open and close to an incoming channel endpoint based on the incoming channel number, as well as relaying frames produced by outgoing channel endpoints, marking them with the associated outgoing channel number before sending them.**]** 
**SRS_CONNECTION_01_058: [**This requires connection endpoints to contain two mappings.**]**
**SRS_CONNECTION_01_059: [**One from incoming channel number to incoming channel endpoint**]** , 
**SRS_CONNECTION_01_060: [**and one from outgoing channel endpoint, to outgoing channel number.**]** 

...

Figure 2.17: Unidirectional Channel Multiplexing

**SRS_CONNECTION_01_064: [**Channels are unidirectional, and thus at each connection endpoint the incoming and outgoing channels are completely distinct.**]** 
**SRS_CONNECTION_01_065: [**Channel numbers are scoped relative to direction, thus there is no causal relation between incoming and outgoing channels that happen to be identified by the same number.**]** 
**SRS_CONNECTION_01_066: [**This means that if a bidirectional endpoint is constructed from an incoming channel endpoint and an outgoing channel endpoint, the channel number used for incoming frames is not necessarily the same as the channel number used for outgoing frames.**]** 

...

Figure 2.18: Bidirectional Channel Multiplexing

Although not strictly directed at the connection endpoint, the begin and end frames are potentially useful for the connection endpoint to intercept as these frames are how sessions mark the beginning and ending of communication on a given channel (see section 2.5).

2.4.1 Opening A Connection

**SRS_CONNECTION_01_002: [**Each AMQP connection begins with an exchange of capabilities and limitations, including the maximum frame size.**]** 
**SRS_CONNECTION_01_003: [**Prior to any explicit negotiation, the maximum frame size is 512 (MIN-MAX-FRAME-SIZE) and the maximum channel number is 0.**]** 
**SRS_CONNECTION_01_004: [**After establishing or accepting a TCP connection and sending the protocol header, each peer MUST send an open frame before sending any other frames.**]** 
**SRS_CONNECTION_01_005: [**The open frame describes the capabilities and limits of that peer.**]** 
**SRS_CONNECTION_01_006: [**The open frame can only be sent on channel 0.**]** 
**SRS_CONNECTION_01_007: [**After sending the open frame and reading its partner's open frame a peer MUST operate within mutually acceptable limitations from this point forward.**]** 

...

Figure 2.19: Synchronous Connection Open Sequence

2.4.2 Pipelined Open

For applications that use many short-lived connections, it MAY be desirable to pipeline the connection negotiation process. A peer MAY do this by starting to send subsequent frames before receiving the partner's connection header or open frame. This is permitted so long as the pipelined frames are known a priori to conform to the capabilities and limitations of its partner. For example, this can be accomplished by keeping the use of the connection within the capabilities and limits expected of all AMQP implementations as defined by the specification of the open frame.

...

Figure 2.20: Pipelined Connection Open Sequence
The use of pipelined frames by a peer cannot be distinguished by the peer's partner from non-pipelined use so long as the pipelined frames conform to the partner's capabilities and limitations.

2.4.3 Closing A Connection

**SRS_CONNECTION_01_008: [**Prior to closing a connection, each peer MUST write a close frame with a code indicating the reason for closing.**]** 
**SRS_CONNECTION_01_009: [**This frame MUST be the last thing ever written onto a connection.**]** 
**SRS_CONNECTION_01_010: [**After writing this frame the peer SHOULD continue to read from the connection until it receives the partner's close frame **]** 
**SRS_CONNECTION_01_011: [**(in order to guard against erroneously or maliciously implemented partners, a peer SHOULD implement a timeout to give its partner a reasonable time to receive and process the close before giving up and simply closing the underlying transport mechanism).**]** 
**SRS_CONNECTION_01_012: [**A close frame MAY be received on any channel up to the maximum channel number negotiated in open.**]**
**SRS_CONNECTION_01_013: [**However, implementations SHOULD send it on channel 0**]** , and **SRS_CONNECTION_01_014: [**MUST send it on channel 0 if pipelined in a single batch with the corresponding open.**]** 

...

Figure 2.21: Synchronous Connection Close Sequence

**SRS_CONNECTION_01_015: [**Implementations SHOULD NOT expect to be able to reuse open TCP sockets after close performatives have been exchanged.**]** 
**SRS_CONNECTION_01_240: [**There is no requirement for an implementation to read from a socket after a close performative has been received.**]** 

2.4.4 Simultaneous Close

Normally one peer will initiate the connection close, and the partner will send its close in response. However, because both endpoints MAY simultaneously choose to close the connection for independent reasons, it is possible for a simultaneous close to occur. In this case, the only potentially observable difference from the perspective of each endpoint is the code indicating the reason for the close.

...

Figure 2.22: Simultaneous Connection Close Sequence

2.4.5 Idle Timeout Of A Connection

**SRS_CONNECTION_01_017: [**Connections are subject to an idle timeout threshold.**]** 
**SRS_CONNECTION_01_018: [**The timeout is triggered by a local peer when no frames are received after a threshold value is exceeded.**]** 
**SRS_CONNECTION_01_019: [**The idle timeout is measured in milliseconds, and starts from the time the last frame is received.**]** 
**SRS_CONNECTION_01_020: [**If the threshold is exceeded, then a peer SHOULD try to gracefully close the connection using a close frame with an error explaining why.**]** 
**SRS_CONNECTION_01_021: [**If the remote peer does not respond gracefully within a threshold to this, then the peer MAY close the TCP socket.**]** 
**SRS_CONNECTION_01_022: [**Each peer has its own (independent) idle timeout.**]** 
**SRS_CONNECTION_01_023: [**At connection open each peer communicates the maximum period between activity (frames) on the connection that it desires from its partner.**]** 
**SRS_CONNECTION_01_024: [**The open frame carries the idletime-out field for this purpose.**]** 
**SRS_CONNECTION_01_025: [**To avoid spurious timeouts, the value in idle-time-out SHOULD be half the peer's actual timeout threshold.**]** 
**SRS_CONNECTION_01_026: [**If a peer can not, for any reason support a proposed idle timeout, then it SHOULD close the connection using a close frame with an error explaining why.**]** 
**SRS_CONNECTION_01_027: [**There is no requirement for peers to support arbitrarily short or long idle timeouts.**]** 
**SRS_CONNECTION_01_028: [**The use of idle timeouts is in addition to any network protocol level control.**]** 
**SRS_CONNECTION_01_029: [**Implementations SHOULD make use of TCP keep-alive wherever possible in order to be good citizens.**]** 
**SRS_CONNECTION_01_030: [**If a peer needs to satisfy the need to send traffic to prevent idle timeout, and has nothing to send, it MAY send an empty frame, i.e., a frame consisting solely of a frame header, with no frame body.**]**
**SRS_CONNECTION_01_031: [**Implementations MUST be prepared to handle empty frames arriving on any valid channel**]** , 
**SRS_CONNECTION_01_032: [**though implementations SHOULD use channel 0 when sending empty frames**]** , 
**SRS_CONNECTION_01_033: [**and MUST use channel 0 if a maximum channel number has not yet been negotiated (i.e., before an open frame has been received).**]** 
**SRS_CONNECTION_01_034: [**Apart from this use, empty frames have no meaning.**]** 
**SRS_CONNECTION_01_035: [**Empty frames can only be sent after the open frame is sent.**]** 
**SRS_CONNECTION_01_036: [**As they are a frame, they MUST NOT be sent after the close frame has been sent.**]** 
**SRS_CONNECTION_01_037: [**As an alternative to using an empty frame to prevent an idle timeout, if a connection is in a permissible state, an implementation MAY choose to send a flow frame for a valid session.**]** 
**SRS_CONNECTION_01_038: [**If during operation a peer exceeds the remote peer's idle timeout's threshold, e.g., because it is heavily loaded, it SHOULD gracefully close the connection by using a close frame with an error explaining why.**]** 

2.4.6 Connection States

**SRS_CONNECTION_01_039: [**START In this state a connection exists, but nothing has been sent or received. This is the state an implementation would be in immediately after performing a socket connect or socket accept.**]** 
**SRS_CONNECTION_01_040: [**HDR RCVD In this state the connection header has been received from the peer but a connection header has not been sent.**]** 
**SRS_CONNECTION_01_041: [**HDR SENT In this state the connection header has been sent to the peer but no connection header has been received.**]** 
**SRS_CONNECTION_01_042: [**HDR EXCH In this state the connection header has been sent to the peer and a connection header has been received from the peer.**]** 
**SRS_CONNECTION_01_043: [**OPEN PIPE In this state both the connection header and the open frame have been sent but nothing has been received.**]** 
**SRS_CONNECTION_01_044: [**OC PIPE In this state, the connection header, the open frame, any pipelined connection traffic, and the close frame have been sent but nothing has been received.**]** 
**SRS_CONNECTION_01_045: [**OPEN RCVD In this state the connection headers have been exchanged. An open frame has been received from the peer but an open frame has not been sent.**]** 
**SRS_CONNECTION_01_046: [**OPEN SENT In this state the connection headers have been exchanged. An open frame has been sent to the peer but no open frame has yet been received.**]** 
**SRS_CONNECTION_01_047: [**CLOSE PIPE In this state the connection headers have been exchanged. An open frame, any pipelined connection traffic, and the close frame have been sent but no open frame has yet been received from the peer.**]** 
**SRS_CONNECTION_01_048: [**OPENED In this state the connection header and the open frame have been both sent and received.**]** 
**SRS_CONNECTION_01_049: [**CLOSE RCVD In this state a close frame has been received indicating that the peer has initiated an AMQP close.**]**
**SRS_CONNECTION_01_050: [**No further frames are expected to arrive on the connection;**]** 
**SRS_CONNECTION_01_051: [**however, frames can still be sent.**]** 
**SRS_CONNECTION_01_052: [**If desired, an implementation MAY do a TCP half-close at this point to shut down the read side of the connection.**]** 
**SRS_CONNECTION_01_053: [**CLOSE SENT In this state a close frame has been sent to the peer. It is illegal to write anything more onto the connection, however there could potentially still be incoming frames.**]** 
**SRS_CONNECTION_01_054: [**If desired, an implementation MAY do a TCP half-close at this point to shutdown the write side of the connection.**]** 

**SRS_CONNECTION_01_055: [**DISCARDING The DISCARDING state is a variant of the CLOSE SENT state where the close is triggered by an error.**]** 
**SRS_CONNECTION_01_056: [**In this case any incoming frames on the connection MUST be silently discarded until the peer's close frame is received.**]** 
**SRS_CONNECTION_01_057: [**END In this state it is illegal for either endpoint to write anything more onto the connection. The connection can be safely closed and discarded.**]** 

2.4.7 Connection State Diagram

The graph below depicts a complete state diagram for each endpoint. The boxes represent states, and the arrows represent state transitions. Each arrow is labeled with the action that triggers that particular transition.

...

Figure 2.23: Connection State Diagram

State Legal Sends Legal Receives Legal Connection Actions

**SRS_CONNECTION_01_224: [**START HDR HDR**]** 

**SRS_CONNECTION_01_225: [**HDR_RCVD HDR OPEN**]** 

**SRS_CONNECTION_01_226: [**HDR_SENT OPEN HDR**]** 

**SRS_CONNECTION_01_227: [**HDR_EXCH OPEN OPEN**]** 

**SRS_CONNECTION_01_228: [**OPEN_RCVD OPEN \* **]** 

**SRS_CONNECTION_01_229: [**OPEN_SENT \*\* OPEN**]** 

**SRS_CONNECTION_01_230: [**OPEN_PIPE \*\* HDR**]** 

**SRS_CONNECTION_01_231: [**CLOSE_PIPE - OPEN TCP Close for Write**]** 

**SRS_CONNECTION_01_232: [**OC_PIPE - HDR TCP Close for Write**]** 

**SRS_CONNECTION_01_233: [**OPENED \* ***]** 

**SRS_CONNECTION_01_234: [**CLOSE_RCVD \* - TCP Close for Read**]** 

**SRS_CONNECTION_01_235: [**CLOSE_SENT - \* TCP Close for Write**]** 

**SRS_CONNECTION_01_236: [**DISCARDING - \* TCP Close for Write**]** 

**SRS_CONNECTION_01_237: [**END - - TCP Close**]** 

\* = any frames
- = no frames
\*\* = any frame known a priori to conform to the
peer's capabilities and limitations
Figure 2.24: Connection State Table

##Open performative ISO section

2.7.1 Open

Negotiate connection parameters.

\<type name="open" class="composite" source="list" provides="frame">

\<descriptor name="amqp:open:list" code="0x00000000:0x00000010"/>

**SRS_CONNECTION_01_171: [**\<field name="container-id" type="string" mandatory="true"/>**]** 

**SRS_CONNECTION_01_172: [**\<field name="hostname" type="string"/>**]** 

**SRS_CONNECTION_01_173: [**\<field name="max-frame-size" type="uint" default="4294967295"/>**]** 

**SRS_CONNECTION_01_174: [**\<field name="channel-max" type="ushort" default="65535"/>**]** 

**SRS_CONNECTION_01_175: [**\<field name="idle-time-out" type="milliseconds"/>**]** 

**SRS_CONNECTION_01_176: [**\<field name="outgoing-locales" type="ietf-language-tag" multiple="true"/>**]** 

**SRS_CONNECTION_01_177: [**\<field name="incoming-locales" type="ietf-language-tag" multiple="true"/>**]** 

**SRS_CONNECTION_01_178: [**\<field name="offered-capabilities" type="symbol" multiple="true"/>**]** 

**SRS_CONNECTION_01_179: [**\<field name="desired-capabilities" type="symbol" multiple="true"/>**]** 

**SRS_CONNECTION_01_180: [**\<field name="properties" type="fields"/>**]** 

\</type>

The first frame sent on a connection in either direction MUST contain an open performative. Note that the connection header which is sent first on the connection is not a frame. The fields indicate the capabilities and limitations of the sending peer.

Field Details

container-id the id of the source container

hostname the name of the target host

The name of the host (either fully qualified or relative) to which the sending peer is connecting. It is
not mandatory to provide the hostname. If no hostname is provided the receiving peer SHOULD
select a default based on its own configuration. This field can be used by AMQP proxies to
determine the correct back-end service to connect the client to. This field MAY already have been specified by the sasl-init frame, if a SASL layer is used, or, the server name indication extension as described in RFC-4366, if a TLS layer is used, in which case this field SHOULD be null or contain the same value. It is undefined what a different value to that already specified means.

max-frame-size proposed maximum frame size

The largest frame size that the sending peer is able to accept on this connection. If this field is not set it means that the peer does not impose any specific limit. A peer MUST NOT send
frames larger than its partner can handle. A peer that receives an oversized frame MUST close
the connection with the framing-error error-code. **SRS_CONNECTION_01_167: [**Both peers MUST accept frames of up to 512 (MIN-MAX-FRAME-SIZE) octets.**]** 

channel-max the maximum channel number that can be used on the connection

The channel-max value is the highest channel number that can be used on the connection. This
value plus one is the maximum number of sessions that can be simultaneously active on the connection. A peer MUST not use channel numbers outside the range that its partner can handle. A peer that receives a channel number outside the supported range MUST close the connection with the framing-error error-code.

idle-time-out idle time-out

The idle timeout REQUIRED by the sender (see subsection 2.4.5). **SRS_CONNECTION_01_192: [**A value of zero is the same as if it was not set (null).**]** If the receiver is unable or unwilling to support the idle time-out then it SHOULD close the connection with an error explaining why (e.g., because it is too small).
If the value is not set, then the sender does not have an idle time-out. However, senders doing
this SHOULD be aware that implementations MAY choose to use an internal default to efficiently
manage a peer's resources.

outgoing-locales locales available for outgoing text

A list of the locales that the peer supports for sending informational text. This includes connection,
session and link error descriptions. A peer MUST support at least the en-US locale (see subsection
2.8.12 IETF Language Tag). Since this value is always supported, it need not be supplied in
the outgoing-locales. A null value or an empty list implies that only en-US is supported.
incoming-locales desired locales for incoming text in decreasing level of preference
A list of locales that the sending peer permits for incoming informational text. This list is ordered
in decreasing level of preference. The receiving partner will choose the first (most preferred)
incoming locale from those which it supports. If none of the requested locales are supported, en-
US will be chosen. Note that en-US need not be supplied in this list as it is always the fallback. A
peer MAY determine which of the permitted incoming locales is chosen by examining the partner's
supported locales as specified in the outgoing-locales field. A null value or an empty list implies
that only en-US is supported.

offered-capabilities extension capabilities the sender supports

If the receiver of the offered-capabilities requires an extension capability which is not present in
the offered-capability list then it MUST close the connection.
A registry of commonly defined connection capabilities and their meanings is maintained [AMQPCONNCAP].
desired-capabilities extension capabilities the sender can use if the receiver supports them
The desired-capability list defines which extension capabilities the sender MAY use if the receiver
offers them (i.e., they are in the offered-capabilities list received by the sender of the desired capabilities).
The sender MUST NOT attempt to use any capabilities it did not declare in the desired-capabilities field. If the receiver of the desired-capabilities offers extension capabilities which are not present in the desired-capabilities list it received, then it can be sure those (undesired) capabilities will not be used on the connection.
properties connection properties
The properties map contains a set of fields intended to indicate information about the connection
and its container.
A registry of commonly defined connection properties and their meanings is maintained [AMQPCONNPROP].

##Close frame performative section

2.7.9 Close
Signal a connection close.

\<type name="close" class="composite" source="list" provides="frame">

\<descriptor name="amqp:close:list" code="0x00000000:0x00000018"/>

\<field name="error" type="error"/>

\</type>

Sending a close signals that the sender will not be sending any more frames (or bytes of any other kind) on the connection. Orderly shutdown requires that this frame MUST be written by the sender. It is illegal to send any more frames (or bytes of any other kind) after sending a close frame.
Field Details

error error causing the close

**SRS_CONNECTION_01_238: [**If set, this field indicates that the connection is being closed due to an error condition.**]** The value of the field SHOULD contain details on the cause of the error.
