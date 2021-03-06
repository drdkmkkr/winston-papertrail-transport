# winston-papertrail-transport

A Papertrail transport for [winston ^3.0.0][0].
Heavily inspired by and borrows from [winston-papertrail][1] and [winston-syslog][2]

## Requirements

- winston >= 3.0.0

## Installation

### Installing npm (node package manager)

```bash
  $ curl http://npmjs.org/install.sh | sh
```

### Installing winston-papertrail-transport

```bash
  $ npm install winston
  $ npm install winston-papertrail-transport
```

### Usage

To create the transport, simply import the transport and create an instance

```typescript
import * as winston from 'winston';
import { PapertrailTransport } from 'winston-papertrail-transport';
const papertrailTransport = new PapertrailTransport({
  host: 'logs.papertrail.com',
  port: 12345,
});
const logger = winston.createLogger({
  transports: [papertrailTransport],
});
```

The following options are required for logging to Papertrail:

- **host:** FQDN or IP Address of the Papertrail Service Endpoint
- **port:** The Endpoint TCP Port

The following options are optional

- **disableTls:** Disable TLS on the transport. Defaults to `false`.
- **levels:** A mapping of log level strings to [RFC5424 severity levels](https://tools.ietf.org/html/rfc5424#section-6.2.1).
  Defaults to mapping the severity strings and shorthands as supported by Papertrail. If your app is logging with an
  unrecognized level, it will default to "notice" (5).
- **hostname:** The hostname for your transport. Defaults to `os.hostname()`.
- **program:** The program for your transport. Defaults to `default`.
- **facility:** The syslog facility for this transport. Defaults to `daemon`.
- **handleExceptions:** Make transport handle exceptions. Defaults to `false`.
- **flushOnClose:** Flush queued logs in close. Defaults to `true`.
- **attemptsBeforeDecay:** The number of retries attempted before backing off. Defaults to `5`.
- **maximumAttempts:** The number of retries attempted before buffering is disabled. Defaults to `25`.
- **connectionDelay:** The number of time between connection retries in ms attempted before buffering is disabled. Defaults to `1000`.
- **maxDelayBetweenReconnection:** The maximum number of time between connection retries in ms allowed. Defaults to `60000`.

To close the connection, you can use the `close()` function on the transport, e.g. `papertrailTransport.close()`.

The `PapertrailConnection` class which handles the connection and reconnection to Papertrail can be accessed via the `connection` field on the transport, e.g. `papertrailTransport.connection`.
`PapertrailConnection` extends `EventEmitter` and allows the user to listen to `connect` and `error` events, e.g.

```typescript
papertrailTransport.connection.on('error', error => {
  console.error('Error recieved from PapertrailTransport: ' + error.message);
});
papertrailTransport.connection.on('connect', event => {
  console.log(event);
});
```

## Example usage

Here is an example on how the papertrail transport can be used together with the console transport in a typescript express app. The example also modifies some settings to yield a meaningful log line in Papertrail.

```typescript
import * as packageFile from './package.json';
import * as winston from 'winston';
import expressWinston from 'express-winston';
import { PapertrailTransport } from 'winston-papertrail-transport';

const hostname = `${packageFile.name}:${process.env.NODE_ENV}`;

const container = new winston.Container();

function getConfig(program: string) {
  const transports = [];

  const consoleTransport = new winston.transports.Console();
  transports.push(consoleTransport);

  const papertrailTransport = new PapertrailTransport({
    host: 'logs.papertrailapp.com',
    port: 1234,
    hostname: hostname,
    program,
  });
  transports.push(papertrailTransport);

  return {
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple(),
      winston.format.printf(({ level, message }) => `${level} ${message}`)
    ),
    transports,
  };
}

export function newLogger(program: string) {
  return container.add(program, getConfig(program));
}

export const expressLogger = expressWinston.logger({
  ...getConfig('router'),
  meta: false,
  metaField: null,
  expressFormat: true,
  colorize: true,
});
```

`express-logger` will print

`Jul 09 12:55:43 exampleApp:development router info GET / 304 9ms`

`newLogger('server').info('listening')` will print

`Jul 09 12:56:35 exampleApp:development server info listening`

[0]: https://github.com/winstonjs/winston
[1]: https://github.com/kenperkins/winston-papertrail
[2]: https://github.com/winstonjs/winston-syslog
