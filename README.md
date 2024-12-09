# cpp_coap_server
A simple wrapper around libcoap. That's it. Unlike my other projects in C++, this isn't specifically for ESP32s, because in my use case, that made no sense.

## Why?
Whenever you use libcoap, there is a significant amount of overhead and code required before you can even begin to work with requests themselves. This simple class intends to cut down on the amount of code required, and get you working on the fun stuff faster.

Like with [esp_coap_client](https://github.com/PearOrchards/esp_coap_client) I tried to emulate how Express.JS would look in C++, but to be honest it doesn't really do that. Still, I hope this helps you in some way.

## Limitations
libcoap has a million features and I've implemented basically none of them. **This class is only designed for the simplest use-cases.** For example, there is no:

- TCP Support
- Multicast Support
- TLS Support

... which you'd expect from any feature-complete CoAP library. Remember, this isn't that. It's just a simple wrapper around libcoap to get you moving faster.

## Basic Usage

```c++
#include "CoAPServer/CoAPServer.h"
#include <cjson/cJSON.h> 
#include <iostream>

#define COAP_LISTEN_UCAST_IP "::"

int main() {
    // At the moment I have just copied over the exampleServer here. We can move things around whenever.
    CoAPServer server;
    if ((server.init(COAP_LISTEN_UCAST_IP))) {
        std::cout << "Server initialised successfully" << std::endl;
    } else {
        std::cerr << "Failed to start server" << std::endl;
    }

    server.start();

    // You can use two different methods to create handlers, separately and inline:
    coap_method_handler_t demoHandler = [](auto, auto, const coap_pdu_t *request, auto, coap_pdu_t *response) {
        size_t len;
        const uint8_t *payloadBuffer;

        coap_show_pdu(COAP_LOG_WARN, request);
        coap_get_data(request, &len, &payloadBuffer);

        // I am using cJSON here, but you can use whatever you want
        cJSON *resJson = cJSON_CreateObject();
        cJSON_AddStringToObject(resJson, "message", "Hello, world!");
        char *res = cJSON_PrintUnformatted(resJson);

        coap_pdu_set_code(response, COAP_RESPONSE_CODE_CONTENT);
        coap_add_data(response, strlen(res), reinterpret_cast<const uint8_t *>(res));
        coap_show_pdu(COAP_LOG_WARN, response);
    };
    server.post("test", demoHandler);

    // Inline example, and without the JSON stuff
    server.get("hello", [](auto, auto, const coap_pdu_t *req, auto, coap_pdu_t *res) {
        const std::string data = "Hello world!";
        const unsigned len = data.size();
        coap_show_pdu(COAP_LOG_WARN, req);
        coap_pdu_set_code(res, COAP_RESPONSE_CODE_CONTENT);
        coap_add_data(res, len, reinterpret_cast<const uint8_t *>(data.c_str()));
        coap_show_pdu(COAP_LOG_WARN, res);
    });

    // Pretend I'm doing work.
    std::this_thread::sleep_for(std::chrono::seconds(100));

    server.stop();
    return 0;
}
```

## Installation
This will vary on what kind of project you're using this in. You should just be able to drag these files into a folder in your project, and add them in your .cpp files and include them in CMake if necessary too.

## Contributing and Issues
If you have any problems using this class, please open an issue on this repository, and I'll try get back to you as soon as possible.

If you'd like to contribute, please open a pull request. I'm happy to accept any contributions, but remember the simplicity of this "library". If you're looking to add a lot of features, it may be better to fork this repository and create your own.