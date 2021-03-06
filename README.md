# Twilio Video React App

[![CircleCI](https://circleci.com/gh/twilio/twilio-video-app-react.svg?style=svg)](https://circleci.com/gh/twilio/twilio-video-app-react)

This application demonstrates a multi-party video application built with [twilio-video.js](https://github.com/twilio/twilio-video.js) and [Create React App](https://github.com/facebook/create-react-app).

![App Preview](https://user-images.githubusercontent.com/12685223/76361972-c035b700-62e5-11ea-8f9d-0bb24bd73fd4.png)

## Features

- [x] Video conferencing with real-time video and audio
- [x] Enable/disable camera
- [x] Mute/unmute mic
- [x] Screen sharing
- [x] [Dominant speaker](https://www.twilio.com/docs/video/detecting-dominant-speaker) indicator
- [x] [Network quality](https://www.twilio.com/docs/video/using-network-quality-api) indicator
- [x] [Bandwidth Profile API](https://www.twilio.com/docs/video/tutorials/using-bandwidth-profile-api)

## Requirements

Node.js Version | NPM Version
------------ | -------------
10+ | 6+

### Browser Support

See browser support table for [twilio-video.js SDK](https://github.com/twilio/twilio-video.js/tree/master/#browser-support).

## Getting Started

Run `npm install` to install all dependencies.

The fastest way to get started is to use the Twilio CLI:

1. Install and configure the [Twilio CLI](https://www.twilio.com/docs/twilio-cli/quickstart).
2. Run `twilio plugins:install @twilio-labs/plugin-rtc` to install the app's support plugin.
3. Run `npm run deploy:twilio-cli` 

This will deploy the application as a [Twilio Function](https://www.twilio.com/docs/runtime/functions) and provide a link to the app. For more information see the documentation for the [WebRTC Twilio Cli plugin](https://github.com/twilio-labs/plugin-rtc).

The link and passcode will expire after one week. To deploy a new instance of the app, run `npm run deploy:twilio-cli -- --override`.

### Running the local token server

This application requires an access token to connect to a Room. The included local token [server](server.js) provides the application with access tokens. Perform the following steps to setup the local token server:

- Create an account in the [Twilio Console](https://www.twilio.com/console).
- Click on 'Settings' and take note of your Account SID.
- Create a new API Key in the [API Keys Section](https://www.twilio.com/console/video/project/api-keys) under Programmable Video Tools in the Twilio Console. Take note of the SID and Secret of the new API key.
- Store your Account SID, API Key SID, and API Key Secret in a new file called `.env` in the root level of the application (example below).

```
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_API_KEY_SID=SKxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_API_KEY_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Now the local token server (see [server.js](server.js)) can dispense Access Tokens to connect to a Room.

### Running the App

#### `npm start`

This will start the local token server and run the app in the development mode. Open [http://localhost:3000](http://localhost:3000) to see the application in the browser.

The page will reload if you make changes to the source code in `src/`.
You will also see any linting errors in the console.

#### `npm run server`

This will run a standalone token server.

The token server runs on port 8081 and expects a `GET` request at the `/token` route with the following query parameters:

```
identity: string,  // the user's identity
roomName: string   // the room name
```

The response will be a token that can be used to connect to a room.

Try it out with this sample `curl` command:

`curl 'localhost:8081/token?identity=TestName&roomName=TestRoom'`

### Multiple Participants in a Room

If you want to see how the application behaves with multiple participants, you can simply open `localhost:3000` in multiple tabs in your browser and connect to the same room using different user names.

Additionally, if you would like to invite other participants to a room, each participant would need to have their own installation of this application and use the same room name and Account SID (the API Key and Secret can be different).

### Building

#### `npm run build`

This script will build the static assets for the application in the `build/` directory.

## Tests

This application has unit tests (using [Jest](https://jestjs.io/)) and E2E tests (using [Cypress](https://www.cypress.io/)). You can run the tests with the following scripts.

### Unit Tests

#### `npm test`

This will run all unit tests with Jest and output the results to the console.

### E2E Tests

#### `npm run cypress:open`

This will open the Cypress test runner. When it's open, select a test file to run.

Note: Be sure to complete the 'Getting Started' section before running these tests. These Cypress tests will connect to real Twilio rooms, so you may be billed for any time that is used.

## Application Architecture

The state of this application (with a few exceptions) is managed by the [room object](https://media.twiliocdn.com/sdk/js/video/releases/2.0.0/docs/Room.html) that is supplied by the SDK. The `room` object contains all information about the room that the user is connected to. The class hierarchy of the `room` object can be viewed [here](https://www.twilio.com/docs/video/migrating-1x-2x#object-model).

One great way to learn about the room object is to explore it in the browser console. When you are connected to a room, the application will expose the room object as a window variable: `window.twilioRoom`.

Since the Twilio Video SDK manages the `room` object state, it can be used as the source of truth. It isn't necessary to use a tool like Redux to track the room state. The `room` object and most child properties are [event emitters](https://nodejs.org/api/events.html#events_class_eventemitter), which means that we can subscribe to these events to update React components as the room state changes.

[React hooks](https://reactjs.org/docs/hooks-intro.html) can be used to subscribe to events and trigger component re-renders. This application frequently uses the `useState` and `useEffect` hooks to subscribe to changes in room state. Here is a simple example:

```
import { useEffect, useState } from 'react';

export default function useDominantSpeaker(room) {
  const [dominantSpeaker, setDominantSpeaker] = useState(room.dominantSpeaker);

  useEffect(() => {
    room.on('dominantSpeakerChanged', setDominantSpeaker);
    return () => {
      room.off('dominantSpeakerChanged', setDominantSpeaker);
    };
  }, [room]);

  return dominantSpeaker;
}
```

In this hook, the `useEffect` hook is used to subscribe to the `dominantSpeakerChanged` event emitted by the `room` object. When this event is emitted, the `setDominantSpeaker` function is called which will update the `dominantSpeaker` variable and trigger a re-render of any components that are consuming this hook.

For more information on how React hooks can be used with the Twilio Video SDK, see this tutorial: https://www.twilio.com/blog/video-chat-react-hooks. To see all of the hooks used by this application, look in the `src/hooks` directory.

## Configuration

The `connect` function from the SDK accepts a [configuration object](https://media.twiliocdn.com/sdk/js/video/releases/2.0.0/docs/global.html#ConnectOptions). The configuration object for this application can be found in [src/index.ts](https://github.com/twilio/twilio-video-app-react/blob/AHOYAPPS-30-readme/src/index.tsx#L19). In this object, we 1) enable dominant speaker detection, 2) enable the network quality API, and 3) supply various options to configure the [bandwidth profile](https://www.twilio.com/docs/video/tutorials/using-bandwidth-profile-api).

### Track Priority Settings

This application dynamically changes the priority of remote video tracks to provide an optimal collaboration experience. Any video track that will be displayed in the main video area will have `track.setPriority('high')` called on it (see the [VideoTrack](https://github.com/twilio/twilio-video-app-react/blob/AHOYAPPS-30-readme/src/components/VideoTrack/VideoTrack.tsx#L24) component) when the component is mounted. This higher priority enables the track to be rendered at a high resolution. `track.setPriority(null)` is called when the component is unmounted so that the track's priority is set to its publish priority (low).

## Google Authentication using Firebase (optional)

This application can be configured to authenticate users before they use the app. Once users have signed into the app with their Google credentials, their Firebase ID Token will be included in the Authorization header of the HTTP request that is used to obtain an access token. The Firebase ID Token can then be [verified](https://firebase.google.com/docs/auth/admin/verify-id-tokens) by the server that dispenses access tokens for connecting to a room. 

See [.env.example](.env.example) for an explanation of the environment variables that must be set to enable Google authentication.

## Related

- [Twilio Video Android App](https://github.com/twilio/twilio-video-app-android)
- [Twilio Video iOS App](https://github.com/twilio/twilio-video-app-ios)
- [Twilio CLI RTC Plugin](https://github.com/twilio-labs/plugin-rtc)

## License

See the [LICENSE](LICENSE) file for details.
