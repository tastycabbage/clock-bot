# better OWOP.js (BETA)

## TO-DO
remove WeirdDataView from not needed places like setPixel move etc.

## changelog
added canvasUtils
added pasteImageData
changed paste to pasteChunk

repaired unsafe

added isWorldConnected\
added Client.destroyed\
added Client.destroy()\
### removed id event use join insted

OWOPUnlocked variable fix

wolfMove fix

now it will use webpack emitter on unlocked OWOPs\
removed html check (now you will be able to see motd)

getPixel and requestChunk is now queued so you can just use await getPixel\
EventEmitter for browser should be faster and node js uses normal EventEmitter\
Fixed gae memory leak which causing 2 gb of memory while requesting `200x200x2` chunks I did 500x500x2 screenshot using this [link](https://freeimage.host/i/1588530749940.JYf3Sj)

Added parseMessage\
Added wolfMove to setPixel which checks if you must move or not\
Added login using Captcha token\
Added bot nick

Render captcha returns captcha token




Installing: `npm i better-owop-js`.\
**REQUIRES NODE.JS 12.0+!**

You can use it in opm.

#### OPM Example
You can connect to my owop from opm using server selector
```js
const BOJS = OPM.require("better-owop-js");
const Client = new BOJS.Client({
	ws: OWOP.options.serverAddress[0].url
});

Client.on("join", () => {
    console.log(Client.player.id);
});
```

#### Browser Example (just paste)
```js
const BOJS = OPM.require("better-owop-js");
const Client = new BOJS.Client({
	protocol: 0 // if you want connecto to bop it's owop,
	ws: "ws://104.237.150.24:1337",
	world: "owop", // default world is owop on his owop
	unsafe: true // because pixel bucket is ignored in bop it's owop and then you will can use infinity setPixel
});

Client.on("join", () => {
    console.log(Client.player.id);
});
```

#### Node.js Example
```js
const BOJS = require("better-owop-js");
const Client = new BOJS.Client({
    reconnect: true,
    controller: true
});

Client.on("join", () => {
    Client.chat.send("Hello, OWOP from BOJS!");
});
```

# Events
`open` - WebSocket connecting got opened.\
`close` - WebSocket connecting got closed [close reason].\
`join` - Joined to world [world name, id].\
`rawMessage` - Raw websocked message (ArrayBuffer or string) [data].\
`updatedPlayers` - Players updates [players object].\
`updatedPixels` - Pixels update [pixels object].\
`playersLeft` - a player left [players id object].\
`newPlayers` - a player connected [player id].\
`teleport` - got 'teleport' opcode. Very rare. [x, y].\
`rank` - Got new rank. [rank].\
`captcha` - Captcha state. [gcaptcha id].\
`chunkProtect` - Chunk (un)protected. [x, y, newState].\
`pquota` - New PQuota. [rate, per].\
`chunk` - New chunk. [x, y, chunk, protected, isNew].\
`message` - New message in chat. [msg, parsedMessage].\
`destroy` - Socket was destroyed and will not reconnect until you will call bot.makeSocket().\
Emits when you gets ban.

# Options
`ws` - Websocket server address. (default - `wss://ourworldofpixels.com`)\
`origin` - Origin header (default - `https://ourworldofpixels.com`).\
`autoMakeSocket` - should make socket automatically (default - true)\
`autoConnectWorld` - should join world automatically (default - true)\
`protocol` - protocol id so if you set to 0 you will be able to connect bop it owop (default - 1)\
`captchaSiteKey` - captcha key used only on browser to render captcha (default - 6LcgvScUAAAAAARUXtwrM8MP0A0N70z4DHNJh-KI)\
`id` - ID for logging. If not set, OWOP ID will be used.\
`agent` - Proxy Agent.\
`world` -  World name. (default - `main`).\
`log` - default true.\
`reconnect` - Reconnect if disconnected.\
`adminlogin` -  Admin login.\
`modlogin` - Mod login.\
`pass` -  Pass for world.\
`nick` - Nick of bot.\
`captchaPass` -  Captcha pass.\
`captchaToken` - Automatically sends to server captcha token if provided.\
`teleport` -  Teleport on 'teleport' opcode.\
`controller` - Enable controller for this bot. (Use only once!).\
`reconnectTime` - Reconnect time (ms) after disconnect (default - 5000).\
`worldVerification` - world verification code (default - 25565)
`unsafe` - Use methods that are supposed to be only for admin or moderator or checking bucket.

# Module
When you require lib, you get object with:

`Client` - main OJS Client class (requires `options` object).\
`Bucket` - Bucket class for quota.\
`ChunkSystem` - Class for chunks, pixels management.\
`EventEmitter` - opcionally if you use browser version\
`WeirdDataView` - Normal dataView which automatically adds offset
```js
let dv = new BOJS.WeirdDataView(new ArrayBuffer(1));
//dv.setUint8(value, offset = this.offset, addToOffset = true)
dv.setUint8(1, null, false);
dv.getUint8(); // 1
```

# API

## Client
Client API is similar to OWOP, and some methods have same 'path'.
### <\static> Client.options
Object with OWOP options. Check code to see them.
### Client.clientOptions
Options that you passed in `options` argument.
### Client.chat
#### Client.chat.parseMessage
Parses message to [userInfo, messageContent, isTell, rawMessage]
#### Client.chat.send(msg)
Send message in chat.
#### Client.chat.sendModifier
Function for modifying and getting messages that you gonna send.
#### Client.chat.recvModifier
Function for modifying and getting messages that you're getting from server.
#### Client.chat.messages
All messages that you got. Keep in mind that it can only hold maximum of `Client.options.maxChatBuffer` messages in it (default - 256).

### Client.world

#### Client.world.pasteChunk(chunkX, chunkY, chunkData)
Pastes chunk
#### async Client.world.pasteImageData(pixelX, pixelY, imageData, isCanvasOrImage)
Pastes imageData/image/canvas
```js
let imageData = canvasUtils.createImageData(10, 10);
let i = canvasUtils.getIbyXY(1, 3);

imageData.data[i] = 123;
imageData.data[i + 1] = 32;
imageData.data[i + 2] = 123;
imageData.data[i + 3] = 255; // then it will not be transparent

await bot.world.pasteImageData(12, 12, imageData); // pastes one pixel
bot.on("chunk", (x, y) => {
	if(x === Math.floor(12 / 16) && y === Math.floor(12 / 16)) bot.world.pasteImageData(12, 13, canvasUtils.imageDataToCtx(imageData).canvas, true); // pastes pixel under pixel
});
```

#### Client.world.join(name)
Function to join world. Should not be used, only for internal use! For connections to new worlds you should use new `Client` with `world` option in it.
#### Client.world.leave()
Leave world. If there's `reconnect` option enabled, client will try to reconnect after `options.reconnectTime` (default - 5000ms) seconds.
#### Client.world.move(x = 0, y = 0)
Move bot to X, Y.
#### Client.world.setPixel(x = player.x, y = player.y, color = player.color, wolfMove, sneaky, move = isAdmin)
Move and set pixel. If `sneaky` option is set to true, bot will return to old location.\
wolf move checks if it must move when it is more than 3 chunks from x and y
#### Client.world.setTool(id = 0)
Set tool that bot has eqquiped.
#### Client.world.setColor(color = [0, 0, 0])
Set color of bot.
#### Client.world.protectChunk(x = player.x, y = player.y, newState = 1)
Protect chunk. You need to be admin to use this but you can ignore this if you'll use `unsafe` option.
#### Client.world.clearChunk(x = player.x, y = player.y, rgb = player.color)
Clear chunk. You need to be admin to use this but you can ignore this if you'll use `unsafe` option.
#### await Client.world.requestChunk(x = player.x, y = player.y, innacurate)
Request chunk, it'll be loaded to `ChunkSystem`. If `inaccurate` argument is passed, it'll transform `x` and `y` to `chunkX` and `chunkY`, so you can use normal coords to request chunks. Returns raw chunk.
```js
if(inaccurate) {
	x = Math.floor(x/Client.options.chunkSize);
	y = Math.floor(y/Client.options.chunkSize);
};
```
#### await Client.world.getPixel(x = player.x, y = player.y)
Request chunk and get pixel.
#### Client.destroy()
makes that it will not reconnect until you will call bot.makeSocket()
#### Client.makeSocket()
makes socket of bot

### Client.player
- Client.player.x
- Client.player.y
- Client.player.tool
- Client.player.rank
- Client.player.nick
- Client.player.id
- Client.player.color
- Client.player.pixelBucket - instance of `Bucket` for pixelQuota.
- Client.player.chatBucket - instance of `Bucket` for chatQuota.

### Client.players
List of players. Every player is object with properties:
- x
- y
- id
- color
- tool
- rank
- nick

Example: `Client.players[15035]`.

### Client.captcha
- renderCaptcha(uniquename = true) - renders captcha. If used on browser renders captcha otherwise throws error
- login(token) - tries to login using captcha token

### <\static> Client.utils
- **Client.utils.Player** (id) - class of player
- **Client.utils.createChunkFromRGB** (color) - used for older protocol which is on bop it's owop
- **Client.utils.decompress** (compressedChunk) - Chunk decompressor. Vars :(
- **Client.utils.shouldMove** (x1, y1, x2, y2) - used for wolf move (go to setPixel)
- <s>**Client.utils.removeAlphaFromImageData** (imagedata) - removes alpha channel</s> - renamed and moved to canvasUtils
- **Client.utils.isArraysSame** (...arrays) - Checks if arrays are same

### Client.chunkSystem
Instance of `ChunkSystem`

## ChunkSystem
This class is created just for **`Client.chunkSystem`**.

## Chunks
All chunks and pixels stuff goes here.
- **Chunks.chunks** - array with chunks. In this array chunks are saved like this: `Chunks.chunks[x][y]`.
- **Chunks.chunkProtected** - same thing as `chunks` but only for protected chunks.

Keep in mind, that you'll usually only need `Chunks.getPixel` from all this stuff here.

### ChunkSystem.setChunk(x, y, data)
Set chunk data.

### ChunkSystem.getChunk(x, y)
Get chunk data.

### ChunkSystem.removeChunk(x, y)
Remove chunk.

### ChunkSystem.setPixel(x, y, rgb)
Set pixel in chunk.

### **ChunkSystem.getPixel(x, y)**
Get pixel from chunk.

### ChunkSystem.setChunkProtect(x, y, newState)
(un)Protect chunk.

### </static> ChunkSystem.getIbyXY(x, y, width)
so chunk is saved like that\
Uint8ClampedArray(768) [\
123, 123, 213, 123, 123, 213, ... to 16 * 3 (because pixel is 3 places)\
123, 123, 213, 123, 123, 213, ... to 16 * 3\
... to 16\
]\
1 pixel is 3 places in this array\

so to get one pixel you need multiply y * width and add x and multiply it by 3 `(y*width+x)*3`\
if you have alpha channel (ex. in image) it doesn't exists in normal chunks because it is always 255 it will be multiplied by 4\

if you want get x and y from i you will need do\
```
i = i/3; // or 4 read before

let pos = {
	x: i % width,
	y: Math.floor(i/width)
};
```

### ChunkSystem.isProtected(x, y)
Is chunk protected.

## canvasUtils

### canvasUtils.\_lerp(color1, color2, factor = 0.5)
lerps color
```js
canvasUtils._lerp(255, 123, 0.1); // 242
```

### canvasUtils.lerp(color1, color2, factor = 0.5)
lerps RGB array
```js
canvasUtils.lerp([255, 255, 255], [123, 123, 123], 0.1); // Uint8ClampedArray(3) [242, 242, 242]
```

### canvasUtils.imageDataToCtx(imageData)
converts imageData to ctx\
ps. canvas = ctx.canvas

### canvasUtils.createCanvas(width, height)
Creates canvas.\
Remeber that if you know on what engine(browser, nodejs) you are working faster will be creating canvas normal way

### canvasUtils.createImageData(...arguments)
Creates imageData
```js
canvasUtils.createImageData(width, height);
canvasUtils.createImageData(imageData);
```

### canvasUtils.dataToImageData(data, width, height, hasAlpha = true, alpha = 255)
Creates imageData
```js
let chunkImageData = canvasUtils.dataToImageData(bot.chunkSystem.getChunk(0, 0), BOJS.Client.options.chunkSize, BOJS.Client.options.chunkSize, false); // default alpha is 255

let imageDataCopy = canvasUtils.dataToImageData(imageData.data, imageData.width, imageData.height);
```

### canvasUtils.removeAlphaFromImageData(data (imageData.data))
Removes alpha channel from imageData

```js
canvasUtils.removeAlphaFromImageData([255, 252, 123, 32,   255, 252, 123, 32, ...]); // Uint8ClampedArray(data.length - data.length / 4) [255, 252, 123,   255, 252, 123, ...]
```

### canvasUtils.addAlphaToData(data, alpha = 255)
Adds alpha channel to data
```js
canvasUtils.addAlphaToData(new Uint8Array([123,54,32, 145,23,43]), 123) // Uint8ClampedArray(data.length + data.length/4) [123,54,32,123, 145,23,43,123]
```

# Author
License - Mit\
Originally created by [dimden](https://dimden.dev/).\
Edited by mathias377#3326
