
## Visualized radio-streaming with React/Vite/Node/Socket.io

<img src="https://i.imgur.com/YCAPTMe.png" style="width: 100%">

As an intro: [coderadio.freecodecamp.org](https://coderadio.freecodecamp.org/) was an inspiration for me to make an app and write an article about it.

> _There are files in this article where there is a lot of code and no comments or explanations, but I think it will not be difficult to understand the logic written in them. However, if there are any questions about individual parts, feel free to ask some questions here, I will try to respond as soon as possible._

Initially, I wanted to build an online radio web app that would be able to play random audio tracks of my choice. But I ran into a problem where the server had very limited memory to keep too many audio files.

***

The idea to avoid that problem in this article is the following:
The app gets audio file information from the remote editable document, downloads the appropriate audio file from the cloud, and continuously adds it to the stream. After the song is finished, it is deleted from the server. And repeats this process over and over by selecting some random track from the tracklist.

As the owner of the remote tracklist document, you can modify it to change the tracklist.
You can also change the next-track selection logic, so it's up to you.

<img src="https://i.imgur.com/xYjem0Z.jpg" style="width: 100%">

The remote document containing all the audio file information in my case will be a [GitHub Gist file](https://gist.github.com/boolfalse/6b66a0065c70a33f95e0e831cb0c7e9f#file-tracks-json).
The cloud storage in my case will be [Google Drive](https://drive.google.com/drive/folders/1F0Nw6B_aTuwXXy3VcqQOZlZInTgjghbL?usp=sharing), where we'll keep our audio files with open access.

At the end of this article, you will have a single-page app that will play the music you want with visualization.

<img src="https://i.imgur.com/KwGKjNj.gif" style="width: 100%">


Below is the main tech-stack, that will be used:

- [React.js](https://react.dev/) - as a frontend library
- [Vite.js](https://vitejs.dev/) - as a frontend build tool
- [Node.js](https://nodejs.org/en) - as a backend for streaming audio
- [FFmpeg](https://www.ffmpeg.org/) - as a streaming software
- [TypeScript](https://www.typescriptlang.org/) - for better coding experience
- [Socket.io](https://socket.io/) - a library for real-time communications

> _**Requirements:** Make sure you have [Node.js](https://nodejs.org/en/download) & [ffmpeg](https://ffmpeg.org/download.html) installed on your machine._

At first, make sure you have Node.js installed on your machine. In my case, there's Node.js v20.9.0 installed on my machine.

Let's start building the project with the installation of  Vite:

```shell
npm create vite@latest
```

It will prompt you to choose some variants for your project. Type the project name, select _**React**_ with _**TypeScript**_. After the project installation, navigate to the created project and install the dependencies:

```shell
npm i
```

Make sure it's working as expected by running:

```shell
npm run dev
```

You can see the result on the default 5173 port: [http://localhost:5173](http://localhost:5173)

<img src="https://i.imgur.com/accmqy8.png" style="width: 100%">

Before starting the actual development, modify your **_.gitignore_** file to be sure that your project is ready for working with Git as well.
Here's a [link](https://www.toptal.com/developers/gitignore/api/webstorm+all,intellij+all,jetbrains+all,visualstudiocode,node,react) you can get the content from.
Just copy the content and overwrite your existing **_.gitignore_**.

For the frontend, install some necessary packages by running:

```shell
npm i tsx dotenv cors concurrently axios rc-progress socket.io-client
```

- [tsx](https://www.npmjs.com/package/tsx) - for executing TypeScript
- [dotenv](https://npmjs.com/package/dotenv) - for loading environment variables from a ".env" file
- [cors](https://www.npmjs.com/package/cors) - for using CORS capabilities
- [concurrently](https://www.npmjs.com/package/concurrently) - for running backend and frontend by a single command
- [axios](https://www.npmjs.com/package/axios) - for the HTTPS requests
- [rc-progress](https://www.npmjs.com/package/rc-progress) - for the HTML progress bar
- [socket.io-client](https://www.npmjs.com/package/socket.io-client) - client-side library for real-time communication between server and client

Now let's do some server-side development.

Create a new file _**server/index.ts**_ in a new server directory:

```javascript
console.log("Hello from Server!");
```

Add a new run command in **_package.json_** in the _"scripts"_ object:

```json
"server": "tsx server/index.ts",
```

So you can run the server with this:

```shell
npm run server
```

Now, as you can run the backend successfully, you can work on it.

Add a new run command in **_package.json_** in the _"scripts"_ object:

```json
"project": "concurrently \"npm run server\" \"npm run dev\"",
```

So you can run backend and frontend by a single command:

```shell
npm run project
```

Create **_.env_** and **_.env.example_** files in a root of your project with the following content:

```dotenv
APP_ENV="development"
VITE_BACKEND_PORT=3001
```

_VITE_BACKEND_PORT_ will be the port for the backend server. You can set that as you want, but I will leave it _3001_.

Add [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) features to the **_vite.config.ts_**, and get environment variables there by using _dotenv_ package. So it will be like this:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import 'dotenv/config';

export default defineConfig({
  build: {
    minify: process.env.APP_ENV === 'production' ? 'esbuild' : false,
    cssMinify: process.env.APP_ENV === 'production',
  },
  plugins: [react()],
  server: {
    https: process.env.APP_ENV === 'production',
    proxy: {
      '/api': {
        target: `http://localhost:${process.env.VITE_BACKEND_PORT || 3001}`,
        changeOrigin: true,
        secure: true,
      },
      '/socket.io': {
        target: `http://localhost:${process.env.VITE_BACKEND_PORT || 3001}`,
        ws: true,
      },
    }
  },
});
```

Let's do some improvements and add asset files.

Add the default track image **_public/default.png_**. I will take it from [here](https://avatars.githubusercontent.com/u/22226570).

Add the background GIF **_public/moving-clouds.gif_**. I will take it from [here](https://i.imgur.com/xCIr6kN.gif).

Now let's create some types & interfaces for later work.

Create a new file **_src/types/trackInfoType.ts_**:

```javascript
type trackInfoType = {
    title: string;
    image: string;
    duration: number;
    time: string;
    difference_in_seconds: number;
}

export default trackInfoType;
```

Create a new file **_src/interfaces/InfoInterface.ts_**:

```javascript
import trackInfoType from "../types/trackInfoType";

export default interface InfoInterface {
    setCurrentTrackInfo: (trackInfo: trackInfoType) => void;
    setIsTrackChanged: (isTrackChanged: boolean) => void;
}
```

Create a new file **_src/interfaces/PlayerInterface.ts_**:

```javascript
import trackInfoType from "../types/trackInfoType";

export default interface PlayerInterface {
    defaultTrackInfo: trackInfoType;
    isPlaying: boolean;
    setIsPlaying: (isPlaying: boolean) => void;
    currentTrackInfo: trackInfoType;
    setCurrentTrackInfo: (trackInfo: trackInfoType) => void;
    isTrackChanged: boolean;
    setIsTrackChanged: (isTrackChanged: boolean) => void;
}
```

Create a new file **_src/interfaces/VisualizerInterface.ts_**:

```javascript
export default interface VisualizerInterface {
    isPlaying: boolean;
    isTrackInfoReceived: boolean;
}
```

Now it's time to do some backend stuff.

In the server folder add some necessary packages:

```shell
npm i --prefix=server/ axios cors dotenv express fluent-ffmpeg openradio socket.io
npm i --prefix=server/ -D @types/express @types/fluent-ffmpeg
```

- [fluent-ffpeg](https://www.npmjs.com/package/fluent-ffmpeg) - a fluent API to FFMPEG (make sure it's installed on your local machine)
- [openradio](https://www.npmjs.com/package/openradio) - a simple live streaming library
- [socket.io](https://www.npmjs.com/package/socket.io) - server-side library for real-time communication between server and client

Add **_server/utils.ts_** file for some functionality that you will need:

```javascript
const axios = require("axios");
const fs = require("fs");
const ffmpeg = require("fluent-ffmpeg");

module.exports = {
    downloadFileFromGoogleDrive: async (fileUrl, destinationFile) => {
        try {
            const response = await axios({
                url: fileUrl,
                method: 'GET',
                responseType: 'stream',
            });

            const totalSize = response.headers['content-length'];
            let downloadedSize = 0;
            let previousProgress = 0;

            const writer = fs.createWriteStream(destinationFile);
            response.data.pipe(writer);

            response.data.on('data', (chunk) => {
                downloadedSize += chunk.length;
                const progress = Math.round((downloadedSize / totalSize) * 100);
                if (progress !== previousProgress) {
                    previousProgress = progress;
                    process.stdout.write(`\rDownloading file: ${progress}%; `);
                }
            });

            return new Promise((resolve, reject) => {
                writer.on('finish', resolve);
                writer.on('error', reject);
            });
        } catch (err) {
            console.error(err.message);
        }
    },
    getGistFileContent: async (gistId, fileName) => {
        try {
            const response = await axios.get(`https://api.github.com/gists/${gistId}`);
            const file = response.data.files[fileName];
            return file.content;
        } catch (err) {
            return null;
        }
    },
    getTrackDuration: (trackPath) => {
        return new Promise((resolve, reject) => {
            ffmpeg.ffprobe(trackPath, (err, metadata) => {
                if (err) {
                    console.error(err.message || 'Error while getting track duration!');
                    reject(err);
                }
                const trackDuration = metadata.format.duration;
                const duration = trackDuration ? Math.floor(trackDuration) : 0;

                resolve(duration);
            });
        })
    },
    exitHandler: (message = '') => {
        console.log(message);
        process.exit(1);
    },
};
```

Add these environment variables to your **_.env_** (it's good to add to **_.env.example_** as well):

```dotenv
VITE_SOCKET_PORT=3000
VITE_SOCKET_HOST="http://localhost"

RADIO_GIST_ID="6b66a0065c70a33f95e0e831cb0c7e9f"
RADIO_PLAYLIST_FILE="tracks"
```

Write the server logic in **_src/index.ts_**:

```javascript
// modules
import 'dotenv/config';
import fs from 'fs';
import path from 'path';
import http from 'http';
import express from 'express';
import openRadio from 'openradio';
import cors from 'cors';
import {Server as SocketIO} from 'socket.io';
import {
    downloadFileFromGoogleDrive,
    getGistFileContent,
    getTrackDuration,
    exitHandler
} from './utils';

// configs
const app = express();
const radio = openRadio();
app.use(cors());
app.use("/", express.static(path.join(__dirname, "..", "dist")));

// constants
const playlistFile = process.env.RADIO_PLAYLIST_FILE || 'tracks';
const backendPort = process.env.VITE_BACKEND_PORT || 3001;
const socketPort = process.env.VITE_SOCKET_PORT || 3000;
const trackPath = path.join(__dirname, '..', 'public', 'radio.mp3');

// socket related
const socketServer = http.createServer(app);
const io = new SocketIO(socketServer, {
    cors: {
        origin: [
            `http://localhost:${backendPort}`,
        ],
    },
    transports: ['websocket'],
});
let trackInfo = {
    title: '',
    image: '',
    duration: 0,
    started_at: 0,
};
let listenersCount = 0;
const manualDelayTrackChangedEventSeconds = 0;

// routes
app.get("/", (req, res) => {
    res.setHeader("Content-Type", "text/html");
    return res.sendFile(path.join(__dirname, "..", "dist", "index.html"));
});
app.get("/api", (req, res) => {
    return res.json({
        message: "API URL-endpoint.",
    });
});
app.get("/api/track-info", (req, res) => {
    const differenceInSeconds = Math.floor(Date.now() / 1000) - trackInfo.started_at;

    return res.status(200).json({
        ...trackInfo,
        difference_in_seconds: differenceInSeconds,
    });
});
app.get('/stream', (req, res) => {
    res.setHeader("Content-Type", "audio/mp3");
    radio.pipe(res);
});
app.get("/*", (_req, res) => {
    return res.json({
        message: "API URL-endpoint not found!",
    });
});

// Handling errors for any other cases from whole application
app.use((err, req, res) => {
    return res.status(500).json({ error: "Something went wrong!" });
});

// create a server
http.createServer((req, res) => {
    res.setHeader("Content-Type", "audio/mp3");
    radio.pipe(res);
});

// listen on port
socketServer.listen(socketPort, () => {
    console.log(`Socket server running at: \x1b[36mhttp://localhost:\x1b[1m${socketPort}/\x1b[0m`);
});

// socket.io
io.on('connection', (socket) => {
    listenersCount++;
    io.emit('listeners_count', listenersCount);
    socket.on('disconnect', () => {
        listenersCount--;
        io.emit('listeners_count', listenersCount);
    });
    if (trackInfo.duration > 0) {
        setTimeout(() => {
            // console.log(`Playing track: ${trackInfo.title}`);
            socket.emit('track_changed', trackInfo);
        }, manualDelayTrackChangedEventSeconds * 1000);
    }
});

app.listen(backendPort, () => {
    console.log(`Backend running at: \x1b[36mhttp://localhost:\x1b[1m${backendPort}/\x1b[0m`);
});

// play track function
const playTrack = () => {
    // check if old track exists, delete it
    if (fs.existsSync(trackPath)) {
        fs.unlinkSync(trackPath);
    }
    // get playlist from gist
    getGistFileContent(process.env.RADIO_GIST_ID, `${playlistFile}.json`)
        .then((data) => {
            if (!data) {
                exitHandler('Playlist file is empty!');
            }

            const playlist = JSON.parse(data);
            const randomNumber = Math.floor(Math.random() * playlist.length);
            // download random track from the source
            downloadFileFromGoogleDrive(playlist[randomNumber].file, trackPath)
                .then(async () => {
                    const duration = await getTrackDuration(trackPath);
                    if (duration === 0) {
                        console.log('Track duration is 0, skipping...');
                        playTrack();
                        return;
                    }

                    radio.play(fs.createReadStream(trackPath));

                    trackInfo.title = playlist[randomNumber].title;
                    trackInfo.image = playlist[randomNumber].image;
                    trackInfo.duration = duration; // playlist[randomNumber].duration;
                    trackInfo.started_at = Math.floor(Date.now() / 1000);

                    setTimeout(() => {
                        console.log(`Playing track: ${trackInfo.title}`);
                        io.sockets.emit('track_changed', trackInfo);
                    }, manualDelayTrackChangedEventSeconds * 1000);
                })
                .catch((err) => {
                    exitHandler(err.message || 'Error while downloading track!');
                });
        })
        .catch((err) => {
            exitHandler(err.message || 'Error while getting playlist!');
        });
}

// play track on start
playTrack();
// play next track when current track ends
radio.on('finish', () => {
    playTrack();
});
```

As you can see in **_index.ts_** there is a line:

```javascript
const trackPath = path.join(__dirname, '..', 'public', 'radio.mp3');
```

It means, you have to have some **_public/radio.mp3_**, which always will be sitting in the **_public_** folder and will be constantly changing. So ignore it from the Git-visibility by creating a **_public/.gitignore_** file:

```gitignore
*.mp3
```

Now, let's work on frontend.

Create these components in the new **_src/components_** folder:

- **_src/components/Info.tsx_**

```javascript
import React from 'react';
import {io, Socket} from 'socket.io-client';
import InfoInterface from './../interfaces/InfoInterface.ts'
import trackInfoType from "../types/trackInfoType.ts";

const socketHost: string = import.meta.env.VITE_SOCKET_HOST;
const socketPort: string = import.meta.env.VITE_SOCKET_PORT;
const socket: Socket = io(`${socketHost}:${socketPort}`, {
    transports: ['websocket'],
});

function Info({
                  setCurrentTrackInfo,
                  setIsTrackChanged
}: InfoInterface) {
    const [listenersCount, setListenersCount] = React.useState(0);

    React.useEffect(() => {
        socket.on('listeners_count', (count: number) => {
            setListenersCount(count);
        });
        socket.on('track_changed', (data: trackInfoType) => {
            setCurrentTrackInfo(data);
            setIsTrackChanged(true);
        });
    });

    return (
        <div id="info_container">
            <p>📻 Listeners: <strong>{listenersCount}</strong></p>
            <h2>Online Radio by <a href='https://boolfalse.com/'>@BoolFalse</a></h2>
            <p>This is a simple online radio station. It plays random songs from a defined playlist, which can be updated.</p>
            <p>You can find the source code of this project at GitHub: ⭐ <a href="https://github.com/boolfalse/radio-streaming-project">boolfalse/radio-streaming-project</a></p>
        </div>
    );
}

export default Info;
```

- **_src/components/Player.tsx_**

```javascript
import React from "react";
import { Line } from 'rc-progress';
import PlayerInterface from './../interfaces/PlayerInterface.ts'
import {getErrorMessage, getTrackInfo, timeFormat} from "../utils.ts";
import trackInfoType from "../types/trackInfoType.ts";

function Player({
                    defaultTrackInfo,
                    isPlaying,
                    setIsPlaying,
                    currentTrackInfo,
                    setCurrentTrackInfo,
                    isTrackChanged,
                    setIsTrackChanged,
}: PlayerInterface) {
    const [firstLoad, setFirstLoad] = React.useState(true);
    const progressIntervalSeconds = 1;
    const [startTiming, setStartTiming] = React.useState(defaultTrackInfo.time); // start timing (for example, '1:04')
    const [trackTiming, setTrackTiming] = React.useState(defaultTrackInfo.time); // the current timing (for example, '1:12')
    const [trackDuration, setTrackDuration] = React.useState(defaultTrackInfo.time); // formatted duration (for example, '3:45')

    const [progressPercent, setProgressPercent] = React.useState(0);
    const [btnPlayDisplay, setBtnPlayDisplay] = React.useState(true);

    const [cubeDegree, setCubeDegree] = React.useState(0);
    const cubeRef = React.useRef<HTMLDivElement>(null);

    const startProgress = () => {
        if (currentTrackInfo.duration === 0) {
            return;
        }
        // here the progressPercent already calculated and set
        // here currentTrackInfo.duration already set
        const intervalId = setInterval(() => {
            setProgressPercent((prevProgressPercent) => {
                const newProgressPercent = Math.floor((prevProgressPercent + progressIntervalSeconds / currentTrackInfo.duration * 100) * 100) / 100;
                if (newProgressPercent >= 100) {
                    clearInterval(intervalId);
                    // Progress finished!
                    return 0;
                }
                return newProgressPercent;
            });
        }, progressIntervalSeconds * 1000);
    }
    const rotateCube = (trackImage: string) => {
        const newCubeDegree = (cubeDegree - 90) % 360;
        const nextImage = cubeRef.current!.querySelector(`.pos-${newCubeDegree * -1}`);
        nextImage!.querySelector('img')!.src = trackImage; // currentTrackInfo.image
        setTimeout(() => {
            cubeRef.current!.style.transform = `rotateY(${newCubeDegree}deg)`;
            setCubeDegree(newCubeDegree);
        }, 0.1 * 1000); // manual delay
    }

    const handlePlay = (play: boolean) => {
        setIsPlaying(play); setBtnPlayDisplay(!play);
    };

    React.useEffect(() => {
        if (isTrackChanged) {
            getTrackInfo().then((data: {
                                     success: boolean,
                                     message: string,
                                     track: trackInfoType,
            }) => {
                if (data.track.duration > 0) {
                    setTrackDuration(timeFormat(data.track.duration));
                    setStartTiming(timeFormat(data.track.difference_in_seconds));
                    setTrackTiming(timeFormat(data.track.difference_in_seconds));

                    const percent = Math.floor(data.track.difference_in_seconds / data.track.duration * 100);
                    setProgressPercent(percent);

                    setCurrentTrackInfo(data.track);
                    startProgress();

                    if (firstLoad) {
                        const firstImage = cubeRef.current!.querySelector(`.pos-0`);
                        firstImage!.querySelector('img')!.src = data.track.image;
                    }

                    rotateCube(data.track.image);
                } else {
                    console.error("Track duration is 0!");
                }
            }).catch((error: Error) => {
                console.error(getErrorMessage(error));
            });

            if (firstLoad) {
                setFirstLoad(false);
            }
            setIsTrackChanged(false);
        }
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [isTrackChanged]);

    React.useEffect(() => {
        setTrackTiming(startTiming === defaultTrackInfo.time ? defaultTrackInfo.time : startTiming);
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [startTiming]);
    React.useEffect(() => {
        if (currentTrackInfo.time !== defaultTrackInfo.time) {
            const interval = setInterval(() => {
                const [currentMin, currentSec] = trackTiming.split(':').map(Number);
                const [durationMin, durationSec] = trackDuration.split(':').map(Number);
                if (currentMin === durationMin && currentSec === durationSec) {
                    clearInterval(interval);
                    // Track ended!
                } else {
                    let newSec = currentSec + 1;
                    let newMin = currentMin;
                    if (newSec >= 60) {
                        newSec = 0;
                        newMin += 1;
                    }
                    const formattedSec = (newSec < 10) ? `0${newSec}` : newSec;
                    const formattedMin = (newMin < 10) ? `0${newMin}` : newMin;
                    setTrackTiming(`${formattedMin}:${formattedSec}`);
                }
            }, 1000);

            return () => clearInterval(interval);
        }
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [trackTiming, trackDuration]);

    return <>
        <div id="bg_image"></div>
        <div id="bg_shadow"></div>
        <main>
            <div id="image_wrapper">
                <div className="container">
                    <div className="image-cube" ref={cubeRef}>
                        <div className="pos-0">
                            <img src={defaultTrackInfo.image} alt="Track"/>
                        </div>
                        <div className="pos-90">
                            <img src={defaultTrackInfo.image} alt="Track"/>
                        </div>
                        <div className="pos-180">
                            <img src={defaultTrackInfo.image} alt="Track"/>
                        </div>
                        <div className="pos-270">
                            <img src={defaultTrackInfo.image} alt="Track"/>
                        </div>
                    </div>
                </div>
            </div>
            <div id="track_container">
                <div id="track" style={{display: (isPlaying ? 'block' : 'none')}}>
                    <div id="progress">
                        <Line percent={progressPercent} strokeWidth={2} strokeColor="#D3D3D3" />
                    </div>
                    <div id="title">
                        <div id="track_title">{currentTrackInfo.title}</div>
                    </div>
                    <div id="duration_timing">
                        <span id="track_duration">{trackDuration}</span> / <span id="track_timing">{trackTiming}</span>
                    </div>
                </div>
            </div>
            <div id="controls_container">
                <div className="control" id="btn_controls">
                    <i id="btn_play"
                       aria-hidden="true"
                       onClick={() => handlePlay(true)}
                       style={{display: btnPlayDisplay ? 'block' : 'none'}}
                       className="material-icons icon">&#xE037;</i>
                    <i id="btn_pause"
                       aria-hidden="true"
                       onClick={() => handlePlay(false)}
                       style={{display: btnPlayDisplay ? 'none' : 'block'}}
                       className="material-icons icon">&#xE034;</i>
                </div>
            </div>
        </main>
    </>
}

export default Player;
```

- **_src/components/Visualizer.tsx_**

```javascript
import React from 'react';
import VisualizerInterface from './../interfaces/VisualizerInterface.ts';
import {getErrorMessage} from "../utils.ts";

function Visualizer({
                        isPlaying,
                        isTrackInfoReceived
}: VisualizerInterface) {
    const [isAudioPlaying, setIsAudioPlaying] = React.useState(isPlaying);
    const streamUrl = `http://localhost:${import.meta.env.VITE_BACKEND_PORT}/stream`;

    const [audioElement, setAudioElement] = React.useState<HTMLAudioElement | null>(null);
    const [analyser, setAnalyser] = React.useState<AnalyserNode | null>(null);
    const [dataArray, setDataArray] = React.useState<Uint8Array>(new Uint8Array([]));
    const visualizerRef = React.useRef<HTMLCanvasElement | null>(null);
    const audioContextRef = React.useRef<AudioContext | null>(null);

    React.useEffect(() => {
        if (isTrackInfoReceived) {
            audioContextRef.current = new AudioContext();
            const analyserNode = audioContextRef.current?.createAnalyser();
            analyserNode.fftSize = 128;
            const bufferLength = analyserNode.frequencyBinCount;
            const dataArray = new Uint8Array(bufferLength);

            if ("destination" in audioContextRef.current) {
                analyserNode.connect(audioContextRef.current.destination);
            }
            setAnalyser(analyserNode);
            setDataArray(dataArray);

            return () => {
                analyserNode.disconnect();
            };
        }
    }, [isTrackInfoReceived]);

    React.useEffect(() => {
        if (audioElement && analyser) {
            if (audioContextRef.current && "createMediaElementSource" in audioContextRef.current) {
                const sourceNode = audioContextRef.current.createMediaElementSource(audioElement);
                sourceNode.connect(analyser);
            }
        }
    }, [audioElement, analyser]);

    React.useEffect(() => {
        if (analyser) {
            const renderVisualization = () => {
                if (visualizerRef.current) {
                    const canvas: HTMLCanvasElement | null = visualizerRef.current;
                    if ("getContext" in canvas) {
                        const canvasContext = canvas.getContext('2d');

                        if (canvasContext) {
                            const { width, height } = canvas;

                            analyser.getByteFrequencyData(dataArray);

                            canvasContext.clearRect(0, 0, width, height);

                            const barWidth = width / dataArray.length;
                            const barHeightMultiplier = height / 255;
                            canvasContext.globalAlpha = 0.5;
                            for (let i = 0; i < dataArray.length; i++) {
                                const barHeight = dataArray[i] * barHeightMultiplier;
                                const x = i * barWidth;
                                const y = 0;
                                canvasContext.fillStyle = `hsl(${i * 2}, 100%, 50%)`;
                                canvasContext.fillRect(x, y, barWidth, barHeight);
                            }

                            requestAnimationFrame(renderVisualization);
                        }
                    }
                }
            };

            renderVisualization();
        }
    }, [analyser, dataArray]);

    React.useEffect(() => {
        handleTogglePlay().then(r => r).catch(err => {
            console.error(getErrorMessage(err));
        });
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [isPlaying]);

    const handleTogglePlay = async () => {
        await audioContextRef.current?.resume();
        if (audioElement) {
            if (isAudioPlaying) {
                audioElement.pause();
            } else {
                await audioElement.play();
            }
            setIsAudioPlaying(!isAudioPlaying);
        }
    };

    return (
        <div>
            <div id="visualizer-container">
                <canvas id="visualizer" ref={visualizerRef} style={{position: "fixed", zIndex: 1}} />
            </div>
            {isTrackInfoReceived && (
                <audio crossOrigin='anonymous' ref={setAudioElement} controls style={{ display: 'none' }}>
                    <source src={streamUrl} type='audio/mpeg' />
                    <track kind='captions' />
                </audio>
            )}
        </div>
    );
}

export default Visualizer;
```

Create **_src/utils.ts_** file for helper functions available in frontend:

```javascript
import axios from "axios";
import trackInfoType from "./types/trackInfoType.ts";

export const getErrorMessage = (error: unknown): string => {
    if (error instanceof Error) return error.message
    return String(error)
}

export const timeFormat = (duration: number): string => {
    const minutes = Math.floor(duration / 60);
    const seconds = duration % 60;
    const formattedSeconds = (seconds < 10) ? `0${seconds}` : seconds;
    const formattedMinutes = (minutes < 10) ? `0${minutes}` : minutes;

    return `${formattedMinutes}:${formattedSeconds}`;
}

export const getTrackInfo = async (): Promise<{
    success: boolean;
    message: string;
    track: trackInfoType;
}> => {
    try {
        const response: {
            data: trackInfoType
        } = await axios.get('/api/track-info');

        return {
            success: true,
            message: "Track info fetched successfully.",
            track: response.data, // { title, image, duration, difference_in_seconds, time }
        };
    } catch (err) {
        return {
            success: false,
            message: getErrorMessage(err),
            track: { title: '', image: '', duration: 0, difference_in_seconds: 0, time: '' },
        }
    }
}
```

Modify **_src/vite-env.d.ts_** to be able to access vite-specific environment variables․

```javascript
/// <reference types="vite/client" />

interface ImportMetaEnv {
    readonly VITE_SOCKET_PORT: string,
    readonly VITE_BACKEND_PORT: string,
    readonly VITE_SOCKET_HOST: string,
}

interface ImportMeta {
    readonly env: ImportMetaEnv
}
```

Delete **_src/App.css_**.

Modify **_src/index.css_**:

```css
body {background-color: whitesmoke;color: rgba(255, 255, 255, 0.7);font-family: "Montserrat", sans-serif;cursor: default;margin: auto 0;}
#bg_image {background: url('/moving-clouds.gif') 0/cover fixed;position: fixed;top: 0;right: 0;bottom: 0;left: 0;box-shadow: inset 0 0 200px #000;filter: blur(20px);animation: blurAnimation 24s infinite;}
#bg_shadow {position: fixed;top: 0;right: 0;bottom: 0;left: 0;background-color: rgba(0, 0, 0, 0.6);animation: blurAnimation 18s infinite;}

main {position: absolute;top: 2rem;right: 0;left: 0;bottom: 2rem;padding: 0 calc(50% - 8rem);text-align: center;}
#track_container {margin-top: 20px;height: 96px;}

#image_wrapper {border: 1px solid #ffffff;height: 160px;width: 210px; /*position: absolute;*/ margin: auto;left: 0;right: 0;top: 0;bottom: 0;}
#image_wrapper .container {height: 100%;width: 100%;display: flex;justify-content: center;align-items: center;perspective: 800px;perspective-origin: 50%;}
#image_wrapper .image-cube {width: 210px;height: 160px;transform-style: preserve-3d;position: relative;transition: 2s;}
#image_wrapper .image-cube div {height: 160px;width: 210px;position: absolute;}
#image_wrapper img {width: 100%;height: 100%;transform: translateZ(0);}
#image_wrapper .pos-0 {transform: translateZ(105px);}
#image_wrapper .pos-90 {transform: rotateY(-270deg) translateX(105px);transform-origin: 100% 0;}
#image_wrapper .pos-180 {transform: translateZ(-105px) rotateY(180deg);}
#image_wrapper .pos-270 {transform: rotateY(270deg) translateX(-105px);transform-origin: 0 50%;}

#progress {width: 100%;background-color: rgba(142, 166, 208, 0.2);}
#title {margin: 1rem 0;width: 100%;overflow: hidden;height: 1.1rem;}
#track_title {animation: textAnimation 16s linear infinite;}

#duration_timing {margin: 0;width: 100%;text-align: center;}
#controls_container {display: flex;align-items: center;justify-content: center;}

.control {width: 2.7rem;height: 2.7rem;border-radius: 50%;border: 1px solid;cursor: pointer;display: inline-block;margin: 1px;transition: transform 0.3s, box-shadow 0.5s, text-shadow 0.5s;}
.control:hover, .control:focus {color: #fff;transform: scale(1.05);box-shadow: 0 0 8px rgba(0, 0, 0, 0.5);text-shadow: 0 0 8px rgba(0, 0, 0, 0.5);}
.control:active {transform: scale(0.9);box-shadow: none;text-shadow: none;}
.control .icon {font-size: 2.7rem;}

#btn_controls {width: 4rem;height: 4rem;font-size: 4rem;}
#btn_controls .icon {font-size: 4rem;}

#visualizer-container {position: fixed;top: 0;left: 0;width: 100%;height: 200px;background-color: transparent;overflow: hidden;z-index: 1;}
#visualizer {width: 100%;height: 200px;display: flex;justify-content: center;align-items: flex-start;transition: opacity 1s ease;}

#info_container {position: absolute;width: 100%;bottom: 0;z-index: 1;color: white;text-align: center;}
#info_container a {color: white;text-decoration: none;}

.ss-rotate-90 {display: inline-block;transform: rotate(90deg);}
.ss-ml-10 {margin-left: 10px;}

@keyframes blurAnimation {0% {filter: blur(20px);} 25% {filter: blur(5px);} 50% {filter: blur(15px);} 75% {filter: blur(0);} 100% {filter: blur(20px);} }
@keyframes textAnimation {from { transform: translateX(100%);} to {transform: translateX(-100%);} }
```

Modify **_src/App.tsx_**:

```javascript
import Player from "./components/Player.tsx";
import Visualizer from "./components/Visualizer.tsx";
import React from "react";
import Info from "./components/Info.tsx";

function App() {
    const [isPlaying, setIsPlaying] = React.useState(false);
    const [isTrackChanged, setIsTrackChanged] = React.useState(false);
    const defaultTrackInfo = {
        title: 'Artist - Title',
        image: '/default.png',
        duration: 0,
        time: '0:00',
        difference_in_seconds: 0,
    };
    const [currentTrackInfo, setCurrentTrackInfo] = React.useState(defaultTrackInfo);

    return (
        <>
            <Visualizer isPlaying={isPlaying}
                        isTrackInfoReceived={currentTrackInfo.duration !== defaultTrackInfo.duration}
            />
            <Player defaultTrackInfo={defaultTrackInfo}
                    isPlaying={isPlaying}
                    setIsPlaying={setIsPlaying}
                    currentTrackInfo={currentTrackInfo}
                    setCurrentTrackInfo={setCurrentTrackInfo}
                    isTrackChanged={isTrackChanged}
                    setIsTrackChanged={setIsTrackChanged}
            />
            <Info setCurrentTrackInfo={setCurrentTrackInfo}
                  setIsTrackChanged={setIsTrackChanged}
            />
        </>
    )
}

export default App
```

Lastly, do some small changes in **_index.html_**, so you will have something like this:


```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <link rel="icon" type="image/svg+xml" href="/vite.svg" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Radio Streaming Project</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Montserrat">
  <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
</head>
<body>
<div id="root"></div>
<script type="module" src="/src/main.tsx"></script>
</body>
</html>
```

Now, your backend and frontend are ready.

Let's build the frontend using Vite, which will generate a dist folder in the root of the project:

```shell
npm run build
```

Run the project:

```shell
npm run project
```

You can check the result at [http://localhost:3000/](http://localhost:3000/).

So you have done all the things you wanted.

**That's it!**

***

Feel free to ask any questions you may have about this article.
If you liked this article, please feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more information, you can visit my website: [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!
