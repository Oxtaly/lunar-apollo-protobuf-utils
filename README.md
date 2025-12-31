# lunar-apollo-protobuf

## Node.js protobufjs based encoder/decoder for [Lunar Client](https://lunarclient.dev/) [apollo messages](https://lunarclient.dev/apollo/internals/protocol-buffers)


- Uses proto files from https://buf.build/lunarclient/apollo

## Installation

```bash
npm i lunar-apollo-protobuf
```

### Then, you can just import it as you would in any project like so:
```js
// in CommonJS
const { LunarProtoUtils } = require('lunar-apollo-protobuf');
// or in ESM
import { LunarProtoUtils } from 'lunar-apollo-protobuf'; 

/** 
 * @type {Buffer} Buffer containing a full lunarclient.apollo message
 * IE the data in a plugin packet with the channel `lunar:apollo`
 */
const buffer = null;

/** May throw if the buffer is not a valid message or is not structured correctly */
const decoded = LunarProtoUtils.decodeMessage(buffer);

if(decoded['@name'] === 'PlayerHandshakeMessage') {
    console.log(`Client is using lunar ${decoded?.lunarClientVersion?.semver}`);
    const message = LunarProtoUtils.create('OverrideServerRichPresenceMessage', {
        gameName: "Visual Studio Code",
        gameState: "Editing index.ts",
        gameVariantName: "Coding...",
        teamMaxSize: 1,
        teamCurrentSize: 1
    });
    message.mapName = "Package/lunar-apollo-protobuf";
    /** Fake function accepting a buffer to send to the client using the `lunar:apollo` channel */
    sendLunarApolloMessageToClient(LunarProtoUtils.encodeMessage(message));
}
```
...or in typescript
```ts
import { LunarProtoUtils } from 'lunar-apollo-protobuf'; 

/** 
 * Buffer containing a full lunarclient.apollo message
 * IE the data in a plugin packet with the channel `lunar:apollo`
 */
const buffer: Buffer = null;

/** May throw if the buffer is not a valid message or is not structured correctly */
const decoded = LunarProtoUtils.decodeMessage(buffer);

if(decoded['@name'] === 'PlayerHandshakeMessage') {
    console.log(`Client is using lunar ${decoded?.lunarClientVersion?.semver}`);
    const message = LunarProtoUtils.create('OverrideServerRichPresenceMessage', {
        gameName: "Visual Studio Code",
        gameState: "Editing index.ts",
        gameVariantName: "Coding...",
        teamMaxSize: 1,
        teamCurrentSize: 1
    });
    message.mapName = "Package/lunar-apollo-protobuf";
    /** Fake function accepting a buffer to send to the client using the `lunar:apollo` channel */
    sendLunarApolloMessageToClient(LunarProtoUtils.encodeMessage(message));
};
```

---

### There are two main ways of using this library to create/verify the same message, consider this:

```ts
import type { LunarProtoMessage, LunarProtoMessageClassByName } from "lunar-apollo-protobuf";
import { LunarProtoUtils } from "lunar-apollo-protobuf";

// Just calling `LunarProtoMessage.lookupType(message['@name'])` would do the same
// thing in this specific case but this is to demonstrate using the exported types
function getInstancingType<T extends LunarProtoMessage>(message: T): LunarProtoMessageClassByName<T['@name']> {
    const messageClass = LunarProtoUtils.lookupType(message['@name']);
    return messageClass as LunarProtoMessageClassByName<T['@name']>;
};

// * An already decoded message or created message
const myMessage: LunarProtoMessage = null; 

const messageClass = getInstancingType(myMessage);

// Creating a new message
const newMessage = messageClass.create({ /** */ });
// Which could also be written as this
const newMessage = LunarProtoUtils.create(myMessage['@name'], { /** */ });

// Though with `messageClass`, you can follow up calls without having to supply the type name again
const errorMessage = messageClass.verify(newMessage);
// ...instead off
const errorMessage = LunarProtoUtils.verify(myMessage['@name'], newMessage);

if(errorMessage) {
    console.error('newMessage is invalid!', errorMessage);
}
```
...or in CommonJS using jsdoc:
```js
const { LunarProtoUtils } = require("lunar-apollo-protobuf");

/**
 * @typedef {import('lunar-apollo-protobuf').LunarProtoMessage} LunarProtoMessage
*/
/**
 * @template {import("lunar-apollo-protobuf").LunarProtoMessageName} T
 * @typedef {import('lunar-apollo-protobuf').LunarProtoMessageClassByName<T>} LunarProtoMessageClassByName
 */

/**
 * @template {LunarProtoMessage} T
 * @param {T} message 
 * @returns {LunarProtoMessageClassByName<T['@name']>}
 */
function getInstancingType(message) {
    const messageClass = LunarProtoUtils.lookupType(message['@name']);
    return /** @type {LunarProtoMessageClassByName<T['@name']>} */ (messageClass);
};

// ...
// The rest is similar
// ...
```

---

## Building from source

```bash
git clone https://github.com/Oxtaly/lunar-apollo-protobuf.git
cd ./lunar-apollo-protobuf
npm i
npx tsc
```

#### Then run the setup script to download the .proto files and automatically bundle them

```bash
npm run setup
```

---

## Additional thanks

This project makes use of the [protobuf decoder by pawtip](https://github.com/pawitp/protobuf-decoder/blob/master/src/protobufDecoder.js) to decode the protobuf message type from the initial message buffer;
It was also very useful during the debugging process thanks to their [very nice to use website](https://protobuf-decoder.netlify.app/)