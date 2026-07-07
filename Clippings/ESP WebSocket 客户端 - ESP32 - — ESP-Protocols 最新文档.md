---
title: ESP WebSocket 客户端 - ESP32 - — ESP-Protocols 最新文档
source: https://docs.espressif.com/projects/esp-protocols/esp_websocket_client/docs/latest/index.html#
author:
published:
created: 2026-07-06
description:
tags:
  - clippings
  - esp32
  - "#小智"
---
## ESP WebSocket客户端

## 概述

ESP WebSocket客户端是ESP32的 [WebSocket协议客户端](https://tools.ietf.org/html/rfc6455) 的实现

## 特色

> - 支持 WebSocket over TCP，TLS 与 mbedtls
> - 用URI设置很简单
> - 多个实例（一个应用程序中的多个客户端）

## 配置

### URI

- 支持、方案 `ws` `wss`
- WebSocket示例：

最小配置：

```c
const esp_websocket_client_config_t ws_cfg = {
    .uri = "ws://echo.websocket.org",
};
```

WebSocket客户端支持在URI中同时使用路径和查询。示例：

```c
const esp_websocket_client_config_t ws_cfg = {
    .uri = "ws://echo.websocket.org/connectionhandler?id=104",
};
```

如果存在与 中 URI 相关的任何选项，URI 定义的选项将为 被覆盖。示例：

```c
const esp_websocket_client_config_t ws_cfg = {
    .uri = "ws://echo.websocket.org:123",
    .port = 4567,
};
//WebSocket client will connect to websocket.org using port 4567
```

### TLS

配置：

```c
const esp_websocket_client_config_t ws_cfg = {
    .uri = "wss://echo.websocket.org",
    .cert_pem = (const char *)websocket_org_pem_start,
};
```

> [!note] 注释
> 如果你想验证服务器，那么你需要提供一份PEM格式的证书，并且在 中提供 。如果没有提供证书认证，TLS连接默认就不需要验证。 `cert_pem` `websocket_client_config_t`

本例中，PEM 证书可从连接 websocket.org 的 openssl s\_client 命令中提取。 如果主机操作系统安装了 openssl 和 sed 包，可以执行以下命令下载并保存根或中间根证书到文件中（给Windows用户注意：可以同时使用类Linux环境或Windows原生包）。

```
echo "" | openssl s_client -showcerts -connect websocket.org:443 \
    | sed -n "1,/Root/d; /BEGIN/,/END/p" \
    | openssl x509 -outform PEM \
    > websocket_org.pem
```

This command will extract the second certificate in the chain and save it as a pem-file.

#### Mutual TLS with DS Peripheral

To leverage the Digital Signature (DS) peripheral on supported targets, use [esp\_secure\_cert\_mgr](https://github.com/espressif/esp_secure_cert_mgr/) to flash an encrypted client certificate. In your project, add the dependency:

```
idf.py add-dependency esp_secure_cert_mgr
```

Set and in the config struct:`client_cert` `client_ds_data`

```c
char *client_cert = NULL;
uint32_t client_cert_len = 0;
esp_err_t err = esp_secure_cert_get_device_cert(&client_cert, &client_cert_len);
assert(err == ESP_OK);

esp_ds_data_ctx_t *ds_data = esp_secure_cert_get_ds_ctx();
assert(ds_data != NULL);

esp_websocket_client_config_t config = {
    .uri = "wss://echo.websocket.org",
    .cert_pem = (const char *)websocket_org_pem_start,
    .client_cert = client_cert,
    .client_ds_data = ds_data,
};
```

> [!note] Note
> `client_cert` provided by esp\_secure\_cert\_mgr is a null-terminated PEM; so (DER format) should not be set.`client_cert_len`

### Subprotocol

The subprotocol field in the config struct can be used to request a subprotocol

```c
const esp_websocket_client_config_t ws_cfg = {
    .uri = "ws://websocket.org",
    .subprotocol = "soap",
};
```

> [!note] Note
> The client is indifferent to the subprotocol field in the server response and will accept the connection no matter what the server replies.

For more options on, please refer to API reference below

## Events

- WEBSOCKET\_EVENT\_BEGIN: The client thread is running.
- WEBSOCKET\_EVENT\_BEFORE\_CONNECT: The client is about to connect.
- WEBSOCKET\_EVENT\_CONNECTED: The client has successfully established a connection to the server. The client is now ready to send and receive data. Contains no event data.
- WEBSOCKET\_EVENT\_DATA: The client has successfully received and parsed a WebSocket frame. The event data contains a pointer to the payload data, the length of the payload data as well as the opcode of the received frame. A message may be fragmented into multiple events if the length exceeds the buffer size. This event will also be posted for non-payload frames, e.g. pong or connection close frames.
- WEBSOCKET\_EVENT\_ERROR: The client has experienced an error. Examples include transport write or read failures.
- WEBSOCKET\_EVENT\_DISCONNECTED: The client has aborted the connection due to the transport layer failing to read data, e.g. because the server is unavailable. Contains no event data.
- WEBSOCKET\_EVENT\_CLOSED: The connection has been closed cleanly.
- WEBSOCKET\_EVENT\_FINISH: The client thread is about to exit.

If the client handle is needed in the event handler it can be accessed through the pointer passed to the event handler:

```c
esp_websocket_client_handle_t client = (esp_websocket_client_handle_t)handler_args;
```

## Limitations and Known Issues

- The client is able to request the use of a subprotocol from the server during the handshake, but does not do any subprotocol related checks on the response from the server.

## Application Example

A simple WebSocket example that uses esp\_websocket\_client to establish a websocket connection and send/receive data with the [websocket.org](https://websocket.org/) server can be found here: [example](https://github.com/espressif/esp-protocols/tree/master/components/esp_websocket_client/examples)

### Sending Text Data

The WebSocket client supports sending data as a text data frame, which informs the application layer that the payload data is text data encoded as UTF-8. Example:

```cpp
esp_websocket_client_send_text(client, data, len, portMAX_DELAY);
```

## API Reference

### Functions

esp\_websocket\_client\_init(const \*config) [](#_CPPv425esp_websocket_client_initPK29esp_websocket_client_config_t "Permalink to this definition")

Start a Websocket session This function must be the first function to call, and it returns a esp\_websocket\_client\_handle\_t that you must use as input to other functions in the interface. This call MUST have a corresponding call to esp\_websocket\_client\_destroy when the operation is complete.

Parameters:

**config** – **\[in\]** The configuration

Returns:

- `esp_websocket_client_handle_t`
- NULL if any errors

esp\_err\_t esp\_websocket\_client\_set\_uri( client, const char \*uri) [](#_CPPv428esp_websocket_client_set_uri29esp_websocket_client_handle_tPKc "Permalink to this definition")

Set URL for client, when performing this behavior, the options in the URL will replace the old ones Must stop the WebSocket client before set URI if the client has been connected.

Parameters:

- **client** – **\[in\]** The client
- **uri** – **\[in\]** The uri

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_set\_headers( client, const char \*headers) [](#_CPPv432esp_websocket_client_set_headers29esp_websocket_client_handle_tPKc "Permalink to this definition")

Set additional websocket headers for the client, when performing this behavior, the headers will replace the old ones.

- This API should be used after the WebSocket client connection has succeeded (i.e., once the transport layer is initialized).
- If you wish to set or append headers before the WebSocket client connection is established(before handshake), consider the following options:
	1. Input headers directly into the config options, terminating each item with \[CR\]\[LF\]. This approach will replace any previous headers. Example: websocket\_cfg.headers = “Sec-WebSocket-Key: my\_key\\r\\nPassword: my\_pass\\r\\n”;
		2. Use the API to append a single header to the current set.`esp_websocket_client_append_header`

Pre:

Must stop the WebSocket client before set headers if the client has been connected

Parameters:

- **client** – **\[in\]** The WebSocket client handle
- **headers** – **\[in\]** Additional header strings each terminated with \[CR\]\[LF\]

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_append\_header( client, const char \*key, const char \*value) [](#_CPPv434esp_websocket_client_append_header29esp_websocket_client_handle_tPKcPKc "Permalink to this definition")

Appends a new key-value pair to the headers of a WebSocket client.

Parameters:

- **client** – **\[in\]** The WebSocket client handle
- **key** – **\[in\]** The header key to append
- **value** – **\[in\]** The associated value for the given key

Pre:

Ensure that this function is called before starting the WebSocket client.

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_start( client) [](#_CPPv426esp_websocket_client_start29esp_websocket_client_handle_t "Permalink to this definition")

Open the WebSocket connection.

Parameters:

**client** – **\[in\]** The client

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_stop( client) [](#_CPPv425esp_websocket_client_stop29esp_websocket_client_handle_t "Permalink to this definition")

Stops the WebSocket connection without websocket closing handshake.

This API stops ws client and closes TCP connection directly without sending close frames. It is a good practice to close the connection in a clean way using esp\_websocket\_client\_close().

Notes:

- Cannot be called from the websocket event handler

Parameters:

**client** – **\[in\]** The client

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_destroy( client) [](#_CPPv428esp_websocket_client_destroy29esp_websocket_client_handle_t "Permalink to this definition")

Destroy the WebSocket connection and free all resources. This function must be the last function to call for an session. It is the opposite of the esp\_websocket\_client\_init function and must be called with the same handle as input that a esp\_websocket\_client\_init call returned. This might close all connections this handle has used.

Notes:

- Cannot be called from the websocket event handler
- This function cannot be called if was used for the same handle.`esp_websocket_client_destroy_on_exit`

Parameters:

**client** – **\[in\]** The client

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_destroy\_on\_exit( client) [](#_CPPv436esp_websocket_client_destroy_on_exit29esp_websocket_client_handle_t "Permalink to this definition")

If this API called, WebSocket client will destroy and free all resources at the end of event loop.

Notes:

- After event loop finished, client handle would be dangling and should never be used
- This function is mutually exclusive with. Do not call manually if this API is used.`esp_websocket_client_destroy` `esp_websocket_client_destroy`

Parameters:

**client** – **\[in\]** The client

Returns:

esp\_err\_t

int esp\_websocket\_client\_send\_bin( client, const char \*data, int len, TickType\_t timeout) [](#_CPPv429esp_websocket_client_send_bin29esp_websocket_client_handle_tPKci10TickType_t "Permalink to this definition")

Write binary data to the WebSocket connection (data send with WS OPCODE=02, i.e. binary)

Parameters:

- **client** – **\[in\]** The client
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_bin\_partial( client, const char \*data, int len, TickType\_t timeout) [](#_CPPv437esp_websocket_client_send_bin_partial29esp_websocket_client_handle_tPKci10TickType_t "Permalink to this definition")

Write binary data to the WebSocket connection and sends it without setting the FIN flag(data send with WS OPCODE=02, i.e. binary)

Notes:

- To send continuation frame, you should use ‘esp\_websocket\_client\_send\_cont\_msg(…)’ API.
- To mark the end of fragmented data, you should use the ‘esp\_websocket\_client\_send\_fin(…)’ API. This sends a FIN frame.

Parameters:

- **client** – **\[in\]** The client
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_text( client, const char \*data, int len, TickType\_t timeout) [](#_CPPv430esp_websocket_client_send_text29esp_websocket_client_handle_tPKci10TickType_t "Permalink to this definition")

Write textual data to the WebSocket connection (data send with WS OPCODE=01, i.e. text)

Parameters:

- **client** – **\[in\]** The client
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_text\_partial( client, const char \*data, int len, TickType\_t timeout) [](#_CPPv438esp_websocket_client_send_text_partial29esp_websocket_client_handle_tPKci10TickType_t "Permalink to this definition")

Write textual data to the WebSocket connection and sends it without setting the FIN flag(data send with WS OPCODE=01, i.e. text)

Notes:

- To send continuation frame, you should use ‘esp\_websocket\_client\_send\_cont\_mgs(…)’ API.
- To mark the end of fragmented data, you should use the ‘esp\_websocket\_client\_send\_fin(…)’ API. This sends a FIN frame.

Parameters:

- **client** – **\[in\]** The client
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_cont\_msg( client, const char \*data, int len, TickType\_t timeout) [](#_CPPv434esp_websocket_client_send_cont_msg29esp_websocket_client_handle_tPKci10TickType_t "Permalink to this definition")

Write textual data to the WebSocket connection and sends it as continuation frame (OPCODE=0x0)

Notes:

- Continuation frames have an opcode of 0x0 and do not explicitly signify whether they are continuing a text or a binary message.
- You determine the type of message (text or binary) being continued by looking at the opcode of the initial frame in the sequence of fragmented frames.
- To mark the end of fragmented data, you should use the ‘esp\_websocket\_client\_send\_fin(…)’ API. This sends a FIN frame.

Parameters:

- **client** – **\[in\]** The client
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_fin( client, TickType\_t timeout) [](#_CPPv429esp_websocket_client_send_fin29esp_websocket_client_handle_t10TickType_t "Permalink to this definition")

Sends FIN frame.

Parameters:

- **client** – **\[in\]** The client
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

int esp\_websocket\_client\_send\_with\_opcode( client, ws\_transport\_opcodes\_t opcode, const uint8\_t \*data, int len, TickType\_t timeout) [](#_CPPv437esp_websocket_client_send_with_opcode29esp_websocket_client_handle_t22ws_transport_opcodes_tPK7uint8_ti10TickType_t "Permalink to this definition")

Write opcode data to the WebSocket connection.

Notes:

- In order to send a zero payload, data and len should be set to NULL/0
- This API sets the FIN bit on the last fragment of message

Parameters:

- **client** – **\[in\]** The client
- **opcode** – **\[in\]** The opcode
- **data** – **\[in\]** The data
- **len** – **\[in\]** The length
- **timeout** – **\[in\]** Write data timeout in RTOS ticks

Returns:

- Number of data was sent
- (-1) if any errors

esp\_err\_t esp\_websocket\_client\_close( client, TickType\_t timeout) [](#_CPPv426esp_websocket_client_close29esp_websocket_client_handle_t10TickType_t "Permalink to this definition")

Close the WebSocket connection in a clean way.

Sequence of clean close initiated by client:

- Client sends CLOSE frame
- Client waits until server echos the CLOSE frame
- Client waits until server closes the connection
- Client is stopped the same way as by the `esp_websocket_client_stop()`
	Notes:
	- Cannot be called from the websocket event handler

Parameters:

- **client** – **\[in\]** The client
- **timeout** – **\[in\]** Timeout in RTOS ticks for waiting

Returns:

esp\_err\_t

esp\_err\_t esp\_websocket\_client\_close\_with\_code( client, int code, const char \*data, int len, TickType\_t timeout) [](#_CPPv436esp_websocket_client_close_with_code29esp_websocket_client_handle_tiPKci10TickType_t "Permalink to this definition")

Close the WebSocket connection in a clean way with custom code/data Closing sequence is the same as for esp\_websocket\_client\_close()

Notes:

- Cannot be called from the websocket event handler

Parameters:

- **client** – **\[in\]** The client
- **code** – **\[in\]** Close status code as defined in RFC6455 section-7.4
- **data** – **\[in\]** Additional data to closing message
- **len** – **\[in\]** The length of the additional data
- **timeout** – **\[in\]** Timeout in RTOS ticks for waiting

Returns:

esp\_err\_t

bool esp\_websocket\_client\_is\_connected( client) [](#_CPPv433esp_websocket_client_is_connected29esp_websocket_client_handle_t "Permalink to this definition")

Check the WebSocket client connection state.

Parameters:

**client** – **\[in\]** The client handle

Returns:

- true
- false

size\_t esp\_websocket\_client\_get\_ping\_interval\_sec( client) [](#_CPPv442esp_websocket_client_get_ping_interval_sec29esp_websocket_client_handle_t "Permalink to this definition")

Get the ping interval sec for client.

Parameters:

**client** – **\[in\]** The client

Returns:

The ping interval in sec

esp\_err\_t esp\_websocket\_client\_set\_ping\_interval\_sec( client, size\_t ping\_interval\_sec) [](#_CPPv442esp_websocket_client_set_ping_interval_sec29esp_websocket_client_handle_t6size_t "Permalink to this definition")

Set new ping interval sec for client.

Parameters:

- **client** – **\[in\]** The client
- **ping\_interval\_sec** – **\[in\]** The new interval

Returns:

esp\_err\_t

int esp\_websocket\_client\_get\_reconnect\_timeout( client) [](#_CPPv442esp_websocket_client_get_reconnect_timeout29esp_websocket_client_handle_t "Permalink to this definition")

Get the next reconnect timeout for client. Returns -1 when client is not initialized or automatic reconnect is disabled.

Parameters:

**client** – **\[in\]** The client

Returns:

Reconnect timeout in msec

esp\_err\_t esp\_websocket\_client\_set\_reconnect\_timeout( client, int reconnect\_timeout\_ms) [](#_CPPv442esp_websocket_client_set_reconnect_timeout29esp_websocket_client_handle_ti "Permalink to this definition")

Set next reconnect timeout for client.

Notes:

- Changing this value when reconnection delay is already active does not immediately affect the active delay and may have unexpected result.
- Good place to change this value is when handling WEBSOCKET\_EVENT\_DISCONNECTED or WEBSOCKET\_EVENT\_ERROR events.

Parameters:

- **client** – **\[in\]** The client
- **reconnect\_timeout\_ms** – **\[in\]** The new timeout

Returns:

esp\_err\_t

Register the Websocket Events.

Parameters:

- **client** – The client handle
- **event** – The event id
- **event\_handler** – The callback function
- **event\_handler\_arg** – User context

Returns:

esp\_err\_t

Unegister the Websocket Events.

Parameters:

- **client** – The client handle
- **event** – The event id
- **event\_handler** – The callback function

Returns:

esp\_err\_t

### Structures

struct esp\_websocket\_error\_codes\_t [](#_CPPv427esp_websocket_error_codes_t "Permalink to this definition")

Websocket error code structure to be passed as a contextual information into ERROR event.

Public Members

esp\_err\_t esp\_tls\_last\_esp\_err [](#_CPPv4N27esp_websocket_error_codes_t20esp_tls_last_esp_errE "Permalink to this definition")

last esp\_err code reported from esp-tls component

int esp\_tls\_stack\_err [](#_CPPv4N27esp_websocket_error_codes_t17esp_tls_stack_errE "Permalink to this definition")

tls specific error code reported from underlying tls stack

int esp\_tls\_cert\_verify\_flags [](#_CPPv4N27esp_websocket_error_codes_t25esp_tls_cert_verify_flagsE "Permalink to this definition")

tls flags reported from underlying tls stack during certificate verification

int esp\_ws\_handshake\_status\_code [](#_CPPv4N27esp_websocket_error_codes_t28esp_ws_handshake_status_codeE "Permalink to this definition")

http status code of the websocket upgrade handshake

int esp\_transport\_sock\_errno [](#_CPPv4N27esp_websocket_error_codes_t24esp_transport_sock_errnoE "Permalink to this definition")

errno from the underlying socket

struct esp\_websocket\_event\_data\_t [](#_CPPv426esp_websocket_event_data_t "Permalink to this definition")

Websocket event data.

Public Members

const char \*data\_ptr [](#_CPPv4N26esp_websocket_event_data_t8data_ptrE "Permalink to this definition")

Data pointer

int data\_len [](#_CPPv4N26esp_websocket_event_data_t8data_lenE "Permalink to this definition")

Data length

bool fin [](#_CPPv4N26esp_websocket_event_data_t3finE "Permalink to this definition")

Fin flag

uint8\_t op\_code [](#_CPPv4N26esp_websocket_event_data_t7op_codeE "Permalink to this definition")

Received opcode

client [](#_CPPv4N26esp_websocket_event_data_t6clientE "Permalink to this definition")

esp\_websocket\_client\_handle\_t context

void \*user\_context [](#_CPPv4N26esp_websocket_event_data_t12user_contextE "Permalink to this definition")

user\_data context, from user\_data

int payload\_len [](#_CPPv4N26esp_websocket_event_data_t11payload_lenE "Permalink to this definition")

Total payload length, payloads exceeding buffer will be posted through multiple events

int payload\_offset [](#_CPPv4N26esp_websocket_event_data_t14payload_offsetE "Permalink to this definition")

Actual offset for the data associated with this event

error\_handle [](#_CPPv4N26esp_websocket_event_data_t12error_handleE "Permalink to this definition")

esp-websocket error handle including esp-tls errors as well as internal websocket errors

int close\_status\_code [](#_CPPv4N26esp_websocket_event_data_t17close_status_codeE "Permalink to this definition")

RFC 6455 close status code received from the server (0 if none or client-initiated)

struct esp\_websocket\_client\_config\_t [](#_CPPv429esp_websocket_client_config_t "Permalink to this definition")

Websocket client setup configuration.

Public Members

const char \*uri [](#_CPPv4N29esp_websocket_client_config_t3uriE "Permalink to this definition")

Websocket URI, the information on the URI can be overrides the other fields below, if any

const char \*host [](#_CPPv4N29esp_websocket_client_config_t4hostE "Permalink to this definition")

Domain or IP as string

int port [](#_CPPv4N29esp_websocket_client_config_t4portE "Permalink to this definition")

Port to connect, default depend on esp\_websocket\_transport\_t (80 or 443)

const char \*username [](#_CPPv4N29esp_websocket_client_config_t8usernameE "Permalink to this definition")

Using for Http authentication, only support basic auth now

const char \*password [](#_CPPv4N29esp_websocket_client_config_t8passwordE "Permalink to this definition")

Using for Http authentication

const char \*path [](#_CPPv4N29esp_websocket_client_config_t4pathE "Permalink to this definition")

HTTP Path, if not set, default is `/`

bool disable\_auto\_reconnect [](#_CPPv4N29esp_websocket_client_config_t22disable_auto_reconnectE "Permalink to this definition")

Disable automatic reconnect on unexpected disconnection (transport errors, poll failures). Default: false (auto-reconnect is enabled). Reconnect delay is controlled by. Note: this flag does not affect behaviour after a clean WebSocket CLOSE handshake; use for that. `reconnect_timeout_ms` `enable_close_reconnect`

bool enable\_close\_reconnect [](#_CPPv4N29esp_websocket_client_config_t22enable_close_reconnectE "Permalink to this definition")

Reconnect after a clean WebSocket CLOSE handshake (RFC 6455 close frame exchange), whether the close was initiated by the server or by calling esp\_websocket\_client\_close(). Default: false (client stops after a clean close). When true, esp\_websocket\_client\_close() returns immediately (non-blocking) after sending the CLOSE frame; the task handles the handshake and reconnects autonomously. Requires to be false; setting this flag while is true has no effect on error-triggered disconnections and may result in a single reconnect attempt that stops on any subsequent error. `disable_auto_reconnect` `disable_auto_reconnect`

void \*user\_context [](#_CPPv4N29esp_websocket_client_config_t12user_contextE "Permalink to this definition")

HTTP user data context

bool task\_core\_id\_set [](#_CPPv4N29esp_websocket_client_config_t16task_core_id_setE "Permalink to this definition")

Set to true to use task\_core\_id. If false, the websocket task uses tskNO\_AFFINITY(default)

int task\_core\_id [](#_CPPv4N29esp_websocket_client_config_t12task_core_idE "Permalink to this definition")

Core ID for the websocket task when task\_core\_id\_set is true. Must be explicitly set by the user, otherwise a zero-initialized config, will pin the task to core 0. Use tskNO\_AFFINITY for no pinning.

int task\_prio [](#_CPPv4N29esp_websocket_client_config_t9task_prioE "Permalink to this definition")

Websocket task priority

const char \*task\_name [](#_CPPv4N29esp_websocket_client_config_t9task_nameE "Permalink to this definition")

Websocket task name

int task\_stack [](#_CPPv4N29esp_websocket_client_config_t10task_stackE "Permalink to this definition")

Websocket task stack

int buffer\_size [](#_CPPv4N29esp_websocket_client_config_t11buffer_sizeE "Permalink to this definition")

Websocket buffer size

const char \*cert\_pem [](#_CPPv4N29esp_websocket_client_config_t8cert_pemE "Permalink to this definition")

Pointer to certificate data in PEM or DER format for server verify (with SSL), default is NULL, not required to verify the server. PEM-format must have a terminating NULL-character. DER-format requires the length to be passed in cert\_len.

size\_t cert\_len [](#_CPPv4N29esp_websocket_client_config_t8cert_lenE "Permalink to this definition")

Length of the buffer pointed to by cert\_pem. May be 0 for null-terminated pem

const char \*client\_cert [](#_CPPv4N29esp_websocket_client_config_t11client_certE "Permalink to this definition")

Pointer to certificate data in PEM or DER format for SSL mutual authentication, default is NULL, not required if mutual authentication is not needed. If it is not NULL, also or (if supported) has to be provided. PEM-format must have a terminating NULL-character. DER-format requires the length to be passed in client\_cert\_len. `client_key` `client_ds_data`

size\_t client\_cert\_len [](#_CPPv4N29esp_websocket_client_config_t15client_cert_lenE "Permalink to this definition")

Length of the buffer pointed to by client\_cert. May be 0 for null-terminated pem

const char \*client\_key [](#_CPPv4N29esp_websocket_client_config_t10client_keyE "Permalink to this definition")

Pointer to private key data in PEM or DER format for SSL mutual authentication, default is NULL, not required if mutual authentication is not needed. If it is not NULL, also has to be provided and (if supported) gets ignored. PEM-format must have a terminating NULL-character. DER-format requires the length to be passed in client\_key\_len `client_cert` `client_ds_data`

size\_t client\_key\_len [](#_CPPv4N29esp_websocket_client_config_t14client_key_lenE "Permalink to this definition")

Length of the buffer pointed to by client\_key\_pem. May be 0 for null-terminated pem

transport [](#_CPPv4N29esp_websocket_client_config_t9transportE "Permalink to this definition")

Websocket transport type, see \`esp\_websocket\_transport\_t

const char \*subprotocol [](#_CPPv4N29esp_websocket_client_config_t11subprotocolE "Permalink to this definition")

Websocket subprotocol

const char \*user\_agent [](#_CPPv4N29esp_websocket_client_config_t10user_agentE "Permalink to this definition")

Websocket user-agent

const char \*headers [](#_CPPv4N29esp_websocket_client_config_t7headersE "Permalink to this definition")

Websocket additional headers

int pingpong\_timeout\_sec [](#_CPPv4N29esp_websocket_client_config_t20pingpong_timeout_secE "Permalink to this definition")

Period before connection is aborted due to no PONGs received

bool disable\_pingpong\_discon [](#_CPPv4N29esp_websocket_client_config_t23disable_pingpong_disconE "Permalink to this definition")

Disable auto-disconnect due to no PONG received within pingpong\_timeout\_sec

bool use\_global\_ca\_store [](#_CPPv4N29esp_websocket_client_config_t19use_global_ca_storeE "Permalink to this definition")

Use a global ca\_store for all the connections in which this bool is set.

esp\_err\_t (\*crt\_bundle\_attach)(void \*conf) [](#_CPPv4N29esp_websocket_client_config_t17crt_bundle_attachE "Permalink to this definition")

Function pointer to esp\_crt\_bundle\_attach. Enables the use of certification bundle for server verification, MBEDTLS\_CERTIFICATE\_BUNDLE must be enabled in menuconfig. Include esp\_crt\_bundle.h, and use here to include bundled CA certificates. `esp_crt_bundle_attach`

const char \*cert\_common\_name [](#_CPPv4N29esp_websocket_client_config_t16cert_common_nameE "Permalink to this definition")

Expected common name of the server certificate

bool skip\_cert\_common\_name\_check [](#_CPPv4N29esp_websocket_client_config_t27skip_cert_common_name_checkE "Permalink to this definition")

Skip any validation of server certificate CN field

bool keep\_alive\_enable [](#_CPPv4N29esp_websocket_client_config_t17keep_alive_enableE "Permalink to this definition")

Enable keep-alive timeout

int keep\_alive\_idle [](#_CPPv4N29esp_websocket_client_config_t15keep_alive_idleE "Permalink to this definition")

Keep-alive idle time. Default is 5 (second)

int keep\_alive\_interval [](#_CPPv4N29esp_websocket_client_config_t19keep_alive_intervalE "Permalink to this definition")

Keep-alive interval time. Default is 5 (second)

int keep\_alive\_count [](#_CPPv4N29esp_websocket_client_config_t16keep_alive_countE "Permalink to this definition")

Keep-alive packet retry send count. Default is 3 counts

int reconnect\_timeout\_ms [](#_CPPv4N29esp_websocket_client_config_t20reconnect_timeout_msE "Permalink to this definition")

Reconnect after this value in miliseconds if disable\_auto\_reconnect is not enabled (defaults to 10s)

int network\_timeout\_ms [](#_CPPv4N29esp_websocket_client_config_t18network_timeout_msE "Permalink to this definition")

Abort network operation if it is not completed after this value, in milliseconds (defaults to 10s)

size\_t ping\_interval\_sec [](#_CPPv4N29esp_websocket_client_config_t17ping_interval_secE "Permalink to this definition")

Websocket ping interval, defaults to 10 seconds if not set

struct ifreq \*if\_name [](#_CPPv4N29esp_websocket_client_config_t7if_nameE "Permalink to this definition")

The name of interface for data to go through. Use the default interface without setting

esp\_transport\_handle\_t ext\_transport [](#_CPPv4N29esp_websocket_client_config_t13ext_transportE "Permalink to this definition")

External WebSocket tcp\_transport handle to the client; or if null, the client will create its own transport handle.

### Macros

WS\_TRANSPORT\_HEADER\_CALLBACK\_SUPPORT [](#c.WS_TRANSPORT_HEADER_CALLBACK_SUPPORT "Permalink to this definition")

### Type Definitions

typedef struct esp\_websocket\_client \*esp\_websocket\_client\_handle\_t [](#_CPPv429esp_websocket_client_handle_t "Permalink to this definition")

### Enumerations

enum esp\_websocket\_event\_id\_t [](#_CPPv424esp_websocket_event_id_t "Permalink to this definition")

Websocket Client events id.

*Values:*

enumerator WEBSOCKET\_EVENT\_ANY [](#_CPPv4N24esp_websocket_event_id_t19WEBSOCKET_EVENT_ANYE "Permalink to this definition")

enumerator WEBSOCKET\_EVENT\_ERROR [](#_CPPv4N24esp_websocket_event_id_t21WEBSOCKET_EVENT_ERRORE "Permalink to this definition")

This event occurs when there are any errors during execution

enumerator WEBSOCKET\_EVENT\_CONNECTED [](#_CPPv4N24esp_websocket_event_id_t25WEBSOCKET_EVENT_CONNECTEDE "Permalink to this definition")

Once the Websocket has been connected to the server, no data exchange has been performed

enumerator WEBSOCKET\_EVENT\_DISCONNECTED [](#_CPPv4N24esp_websocket_event_id_t28WEBSOCKET_EVENT_DISCONNECTEDE "Permalink to this definition")

The connection has been disconnected

enumerator WEBSOCKET\_EVENT\_DATA [](#_CPPv4N24esp_websocket_event_id_t20WEBSOCKET_EVENT_DATAE "Permalink to this definition")

When receiving data from the server, possibly multiple portions of the packet

enumerator WEBSOCKET\_EVENT\_CLOSED [](#_CPPv4N24esp_websocket_event_id_t22WEBSOCKET_EVENT_CLOSEDE "Permalink to this definition")

The connection has been closed cleanly

enumerator WEBSOCKET\_EVENT\_BEFORE\_CONNECT [](#_CPPv4N24esp_websocket_event_id_t30WEBSOCKET_EVENT_BEFORE_CONNECTE "Permalink to this definition")

The event occurs before connecting

enumerator WEBSOCKET\_EVENT\_BEGIN [](#_CPPv4N24esp_websocket_event_id_t21WEBSOCKET_EVENT_BEGINE "Permalink to this definition")

The event occurs once after thread creation, before event loop

enumerator WEBSOCKET\_EVENT\_FINISH [](#_CPPv4N24esp_websocket_event_id_t22WEBSOCKET_EVENT_FINISHE "Permalink to this definition")

The event occurs once after event loop, before thread destruction

enumerator WEBSOCKET\_EVENT\_MAX [](#_CPPv4N24esp_websocket_event_id_t19WEBSOCKET_EVENT_MAXE "Permalink to this definition")

enum esp\_websocket\_error\_type\_t [](#_CPPv426esp_websocket_error_type_t "Permalink to this definition")

Websocket connection error codes propagated via ERROR event.

*Values:*

enumerator WEBSOCKET\_ERROR\_TYPE\_NONE [](#_CPPv4N26esp_websocket_error_type_t25WEBSOCKET_ERROR_TYPE_NONEE "Permalink to this definition")

enum esp\_websocket\_transport\_t [](#_CPPv425esp_websocket_transport_t "Permalink to this definition")

Websocket Client transport.

*Values:*

enumerator WEBSOCKET\_TRANSPORT\_UNKNOWN [](#_CPPv4N25esp_websocket_transport_t27WEBSOCKET_TRANSPORT_UNKNOWNE "Permalink to this definition")

Transport unknown