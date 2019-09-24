# flatpack
Wrapper for msgpack-lite that flattens objects

## Usage

### shared/socket_helper.js

In a shared file, define new Flatpack models for each event. Leave the value of each property empty unless it is an object.

```
import { Flatpack } from './flatpack.js';

const fp = new Flatpack();

fp.add('laserFire', {
  userID: '',
  team: '',
  position: {
    x: '',
    y: '',
    angle: ''
  }
});

fp.add('laserHit', {
  origin: {
    userID: '',
    team: '',
    pointsAwarded: ''
  },
  target: {
    userID: '',
    team: ''
  }
});

export { fp };
```

### client/client_socket.js

On the sending side (the client in this example), use a Flatpack model to encode data.

```
import { fp } from '../shared/socket_helper.js';

sendLaserFire(data) {
  const buffer = fp.encode('laserFire', data);
  this.socket.emit('laserFire', buffer);
}

sendLaserHit(data) {
  const buffer = fp.encode('laserHit', data);
  this.socket.emit('laserHit', buffer);
}

```

### server/server_socket.js

On the receiving end (the server in this example), use a Flatpack model to decode data.

```
import { fp } from '../shared/socket_helper.js';

recvLaserFire(buffer) {
  const data = fp.decode('laserFire', buffer);
  this.game.handleLaserFire(data);
}

recvLaserHit(buffer) {
  const data = fp.decode('laserHit', buffer);
  this.game.handleLaserHit(data);
}
```

## Additional Usage

Filepack can also list all the available events, which are stored as unique integers, so rather than sharing an event string through the socket, we can refrence an event with an integer.

```
const EVENTS = fp.list();

// SEND

sendLaserFire(data) {
  const buffer = fp.encode(EVENTS.laserFire, data);
  this.socket.emit(EVENTS.laserFire, buffer);
}

// RECEIVE

socket.on(EVENTS.laserFire, (buffer) => this.recvLaserFire(socket, buffer))
recvLaserFire(buffer) {
  const data = fp.decode(EVENTS.laserFire, buffer);
  this.game.handleLaserFire(data);
}
```
