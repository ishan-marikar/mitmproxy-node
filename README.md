# mitmproxy-node

A bridge between Python's [`mitmproxy`](https://mitmproxy.org/) and Node.JS programs. Rewrite network requests using Node.JS!

## Why?

It is far easier to rewrite JavaScript/HTML/etc using JavaScript than Python, but mitmproxy only accepts Python plugins.
There are no decent alternatives to mitmproxy, so this package lets me use mitmproxy with Node.js-based rewriting code.

## What can I use this for?

For transparently rewriting HTTP/HTTPS responses. The mitmproxy plugin lets every HTTP request go through to the server uninhibited, and then passes it to Node.js via a WebSocket for rewriting.

If you want to add additional functionality, such as filtering or whatnot, I'll accept pull requests so long as they do not noticeably hinder performance.

## How does it work?

A Python plugin for mitmproxy starts a websocket server

## Your Python plugin is bad and you should feel bad

I have no idea what I am doing. PRs to improve my Python code are appreciated!

## Pre-requisites

* [`mitmproxy`](https://mitmproxy.org/) must be installed and runnable from the terminal. Tested with 2.0.2.
* Python 3.5, since I use the new async/await syntax in the mitmproxy plugin
* `cd scripts; python setup.py install`
* `npm install` to pull in Node dependencies.

## Using

You can either start `mitmproxy` manually with `mitmdump --anticache -s scripts/proxy.py`, or `mitmproxy-node` will do so automatically for you.
`mitmproxy-node` auto-detects if `mitmproxy` is already running.
If you frequently start/stop the proxy, it may be best to start it manually.

```javascript
import MITMProxy from 'mitmproxy-node';

// Returns Promise<MITMProxy>
async function makeProxy() {
  return MITMProxy.Create(function(interceptedMsg) {
    const req = interceptedMsg.request;
    const res = interceptedMsg.response;
    if (req.rawUrl.contains("target.js") && res.getHeader('content-type').indexOf("javascript") !== -1) {
      interceptedMsg.setResponseBody(Buffer.from(`Hacked!`, 'utf8'));
    }
  });
}

async function main() {
  const proxy = await makeProxy();
  // when done:
  await proxy.shutdown();
}
```

Without fancy async/await:

```javascript
import MITMProxy from 'mitmproxy-node';

// Returns Promise<MITMProxy>
function makeProxy() {
  return MITMProxy.Create(function(interceptedMsg) {
    const req = interceptedMsg.request;
    const res = interceptedMsg.response;
    if (req.rawUrl.contains("target.js") && res.getHeader('content-type').indexOf("javascript") !== -1) {
      interceptedMsg.setResponseBody(Buffer.from(`Hacked!`, 'utf8'));
    }
  });
}

function main() {
  makeProxy().then((proxy) => {
    // when done
    proxy.shutdown.then(() => {
      // Proxy is closed!
    });
  });
}
```

## Building

`npm run build`