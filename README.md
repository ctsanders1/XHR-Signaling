## XHR/XMLHttpRequest for [WebRTC](https://www.webrtc-experiment.com/) Signaling

XHR/XMLHttpRequest based [WebRTC](https://www.webrtc-experiment.com/) signaling implementation. 

Though, this repository is part of [WebRTC Experiments](https://github.com/muaz-khan/WebRTC-Experiment) however you can use it within any WebRTC project!

```javascript
// database has a single table; which has two columns: 
// 1) Message (required to store JSON data)
// 2) ID (optional: as primary key)

// a simple function to make XMLHttpRequests
function xhr(url, callback, data) {
    if (!window.XMLHttpRequest || !window.JSON) return;

    var request = new XMLHttpRequest();
    request.onreadystatechange = function () {
        if (callback && request.readyState == 4 && request.status == 200) {
            // server MUST return JSON text
            callback(JSON.parse(request.responseText));
        }
    };
    request.open('POST', url);

    var formData = new FormData();

    // you're passing "message" parameter
    formData.append('message', data);

    request.send(formData);
}

// this object is used to store "onmessage" callbacks from "openSignalingChannel handler
var onMessageCallbacks = {};

// this object is used to make sure identical messages are not used multiple times
var messagesReceived = {};

function repeatedlyCheck() {
    xhr('/Home/GetData', function (data) {
        // if server says nothing; wait.
        if (data == false) return setTimeout(repeatedlyCheck, 400);

        // if already receied same message; skip.
        if (messagesReceived[data.ID]) return setTimeout(repeatedlyCheck, 400);
        messagesReceived[data.ID] = data.Message;

        // "Message" property is JSON-ified in "openSignalingChannel handler
        data = JSON.parse(data.Message);

        // don't pass self messages over "onmessage" handlers
        if (data.sender != connection.userid && onMessageCallbacks[data.channel]) {
            onMessageCallbacks[data.channel](data.message);
        }

        // repeatedly check the database
        setTimeout(repeatedlyCheck, 1);
    });
}

repeatedlyCheck();

// overriding "openSignalingChannel handler
connection.openSignalingChannel = function (config) {
    var channel = config.channel || this.channel;
    onMessageCallbacks[channel] = config.onmessage;

    // let RTCMultiConnection know that server connection is opened!
    if (config.onopen) setTimeout(config.onopen, 1);

    // returning an object to RTCMultiConnection
    // so it can send data using "send" method
    return {
        send: function (data) {
            data = {
                channel: channel,
                message: data,
                sender: connection.userid
            };

            // posting data to server
            // data is also JSON-ified.
            xhr('/Home/PostData', null, JSON.stringify(data));
        },
        channel: channel
    };
};
```

Source code is available here: https://github.com/muaz-khan/XHR-Signaling

Remember: You can use same code JavaScript code both for PHP and ASP.NET.

=

## Other examples

* http://www.RTCMultiConnection.org/docs/openSignalingChannel/

=

## Don't forget to check this one!

* https://github.com/muaz-khan/WebRTC-Experiment/blob/master/Signaling.md

=

## Links

1. www.rtcmulticonnection.org/docs/
2. https:/www.webrtc-experiment.com/
3. https://www.webrtc-experiment.com/docs/WebRTC-Signaling-Concepts.html

=

## Muaz Khan (muazkh@gmail.com) - [@muazkh](https://twitter.com/muazkh) / [@WebRTCWeb](https://twitter.com/WebRTCWeb)

<a href="http://www.MuazKhan.com"><img src="https://www.webrtc-experiment.com/images/Muaz-Khan.gif" /></a>

=

## License

[WebRTC Experiments](https://www.webrtc-experiment.com/) are released under [MIT licence](https://www.webrtc-experiment.com/licence/) . Copyright (c) [Muaz Khan](https://plus.google.com/+MuazKhan).
