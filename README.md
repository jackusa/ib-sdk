[![Logo](./ib-logo.png)](http://interactivebrokers.com/)

# Interactive Brokers SDK

Interactive Brokers SDK is a framework build atop the [native javascript API](https://github.com/pilwon/node-ib).  Straightforward programmatic access to your portfolio and market data subscriptions.

## Installation

    npm install ib-sdk

### Prerequisites

* An [Interactive Brokers](https://www.interactivebrokers.com/) trading account.
* Install the [IB Gateway](http://interactivebrokers.github.io) or [IB TWS (Trader Workstation)](https://www.interactivebrokers.com/en/index.php?f=674&ns=T).

The [IB Gateway](http://interactivebrokers.github.io) and [IB TWS (Trader Workstation)](https://www.interactivebrokers.com/en/index.php?f=674&ns=T) software are graphical Java processes that encrypt and proxy calls to back-end servers.  Without dedicated communication infrastructure, there is no IB support for direct connections to their server tier.

The SDK uses the [native javascript API](https://github.com/pilwon/node-ib) to manage a TCP socket connection from Node.js to an IB Gateway or TWS instance.

## Setup

1. Login to the [IB Gateway](http://interactivebrokers.github.io) or [IB TWS (Trader Workstation)](https://www.interactivebrokers.com/en/index.php?f=674&ns=T) software.
    * The API and SDK expect to connect to an authenticated user session.
    * The IB software must be configured to accept API connections.
    * The SDK connects over `tcp://localhost:4001` by default.
    * Use [ib-controller](https://github.com/ib-controller/ib-controller/releases) to automate UI interaction if so desired.
2. Make sure sure things work by running the terminal interface from the SDK directory.  Any issues encountered during startup will be reported and the terminal will exit.

        $ cd ib-sdk
        $ npm start terminal [config]
    
3. If the SDK can establish a working connection and load the object model, the terminal will start successfully.

        Starting...
        Use the 'ib' variable to access the environment. Type .exit to quit.
        > 

Learn more about exploring the SDK using the terminal [here](./docs/terminal.md).

## Programming

An `Environment` is a realtime object model of your brokerage account(s).  All constituent objects share an interface and pattern of behavior.

* `update` event signals a single data point has changed
* `error` event signals an asynchronous error was encountered
* `cancel` method closes the underlying data subscription and takes the object offline

```javascript
"use strict";

const sdk = require("ib-sdk");

sdk.environment((err, ib) => {
    if (err) {
        console.log("Connection error: " + err.message);
    }
    else {
        let system = ib.system,
            accounts = ib.accounts,
            positions = ib.positions,
            orders = ib.orders,
            trades = ib.executions,
            $ = ib.symbols;
            
        // watch a symbol
        $.watch("AAPL stock", err => {
            // ready
            let aapl = $.AAPL;
        });    
            
        // monitor updates
        accounts.on("update", data => {  });
        
        // handle specific errors
        accounts.on("error", err => accounts.cancel());
    
        // catch all errors
        ib.on("error", err => console.log(err));
        
        // cleanup before exit
        ib.close(() => console.log("Disconnected"));
    }
});
```

Click here for details of the [Object Model](./docs/model.md).

### Customization

The default `Environment` configuration subscribes to system notices, account information, open positions, pending orders, and trade history.  A configuration object or file may be passed to the `sdk.environment` method to customize the specific data subscriptions the environment maintains.

```javascript
sdk.environment("./config.json", (err, ib) => {
    // use a file path
});

let config = sdk.config();
sdk.environment(config, (err, ib) => {
    // use an object
});
```

Specifics of the `Configuration` object can be found [here](./docs/configuration.md).

## Advanced Use

The `Environment` is a good way to get setup quickly and focus on ultimate programming tasks.  Certain use cases benefit from a more light-weight or customized configuration.

A [Session](./docs/session.md) is a generator class used by an `Environment` to construct the realtime object model around a [Service](./docs/service.md), which is responsible for managing low-level requests and data subscriptions.

This separation allows for proxying architecture, whereby a `Session` can connect to a remote `Service`.  Read more [here](./docs/remoting.md).

## License

Copyright (c) 2016, Jonathan Hollinger

Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.