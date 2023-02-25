# Super Bassoon: A Mobile App for Collecting Device Sensor Readings (Private Beta Test I)

## Purpose

We need to train a machine learning model to analyze user motion and determined it types. Model will analyze sensor readings of accelerometer, gyroscope, and magnetometer. In the future, the number of sensors may increase.

Collection of such an enormous amount of data is quite a challenging task.

Firstly, we should have an easy way to communicate with beta testers i.e., tell them which type of data they should collect, and what data format they should use for storing it. 

Secondly, we should be able to onboard new people and manage present beta testers without a hustle. 

Tridly, we should be able to collect data using the same API as one that will be used in production because different APIs has different interfaces, overhead etc. that may affect the efficiency of the machine learning model.

Therefore, we decided to develop a mobile app designed to satisfy all of our needs.

## How Does A Mobile App Work 

### Signing in v1.0.0

0. Bob completes an application for participation in our Private Beta Testers Program. He specifies his name, email, NEAR wallet address, country of residense, age, gender, height, weight, a model of mobile device etc. Bob may be asked to update his 'configuration' in the future.
1. Bob signs in a mobile app using their account's private key (`Welcome Screen`).
2. Now, Bob may swith between `Session History Screen`, `Home Screen`, and `Account Screen`. Initially, Bob sees `Home Screen`.

### Collecting Motion Sensor Readings v1.0.0

1. To start a sensor readings recording session, Bob clicks a corresponding button in `Home Screen`.
2. After that, Bob may configure a new session metadata in `Session Configuration Screen`. See *Special Notes #2* for more details.
3. Now, the most interesting part. Bob is establising connection with `Services @WebSocketGateway`. He may not be able to establish the connection if a number of active connections to the server (i.e., how many other beta testers are recording motion sensors data now) reaches WEBSOCKET_GATEWAY_CONNECTION_LIMIT.
4. Once he has connected to `Services @WebSocketGateway`, he can click a corresponding button to create session and start recording sensor readings. Bob will be disconnected from `Services @WebSocketGateway` if he does not start session within 30 seconds after connection. After 10 minutes since session has been created, the session will be destroyed, and Bob will be disconnected from `Services @WebSocketGateway`. The same will happen if Bob requests to destroy session. To destroy session, Bob clicks the corresponding button. He can also observe time left until the session will be destroyed using a timer in `Session Screen`. Bob can also see status of packages that he has sent. 
5. After the session has been destroyed, Bob sees how much time the session has taken, how many packages he has sent and how many packages he has lost in `Session Summary Screen`. 

### Viewing The Session History vX.Y.Z
Soon...

### Sending & Receiving Notifications vX.Y.Z
Soon...

### Creating Tasks

### Applying for Early Access vX.Y.Z
Soon...

### AND MUCH MORE
Soon...


### Special Notes

1. Stage 0 may be moved to the app in the future. For example, Bob downloads the mobile app and fills out the application. Until he get an approvement from the team, he sees `Early Access Application Screen` and the message `"We are reviewing your application. You will get an access once we proceed it. You will be also notified on your email bob@gmail.com`. Bob will have role `Applicant` until he 
2. Session's metadata is not finalized yet and may be changed in the future. It may include a type of activity he is going to record, position of his device while recording, a type of his bike and a terrain type of the surrounding area.
3. Bob may be required to add the photo of the bike he is going to record his data with and specify the bike's state.
4. In the future, when user motion recognition app is built, there must be a mechanism for @WebSocketGateway horizontal scailing in order to handle millions of users.
5. To avoid the situation in which beta testers will 'overrecord' a specific type of data, and not record other types, manual metadata configuration should be replaces by a predetermined set of tasks. Tasks are set by our team that will allow us to collect data relevant to us at a given moment. Players will be limited in a number. In the future it will allow to create a leaderboard, but we will not tell anyone that they will receive rewards for helping us, and that there will be any leaderboard to avoid losses in the quality of data.
6. In the future, we may also collect location-related data and data from position sensors.
7. We should also collect how many sensor readings has been lost, however they were generated. 
8. `Account Screen` should show beta testers the requests from server to update their info (height, weight etc.), notifications that new tasks were added etc. This patter called Server-Sent Events.


## Prisma Schema

```prisma

// Models For Session-Related Business Logic

enum Sensor {
  ACCELEROMETER
  GYROSCOPE
  MAGNETOMETER
}

// A sensor reading must always have a session
model SensorReading {
  id        Int      @id @default(autoincrement())
  session   Session  @relation(fields: [sessionId], references: [id])
  sessionId Int
  timestamp DateTime
  sensor    Sensor
  xAxis     Float
  yAxis     Float
  zAxis     Float
}

// A session must always have a user
// A session can have zero or more sensor readings
model Session {
  id             Int              @id @default(autoincrement())
  user           User             @relation(fields: [userId], references: [id])
  userId         Int
  createdAt      DateTime         @default(now())
  destroyedAt    DateTime?
  sensorReadings SensorReading[]
}


// Models For Session-Related Business Logic

enum Role {
  ADMIN
  BETATESTER
}

// A user can have zero or more sessions
model User {
  id        Int       @id @default(autoincrement())
  name      String
  publicKey String    @unique
  email     String    @unique
  role      Role
  sessions  Session[]
}
```

## `Users @Service`

```typescript
/**
 *
 **/
function createUser(name: String, publicKey: String, email: String, role: Role) -> User;

/**
 *
 **/
function findUserByPublicKey(userPublicKey: String) -> User | Null;

/**
 *
 **/
function fetchUsers(take: Int, skip: Int) -> User[];

/**
 *
 **/
function deleteUser(userId: Int) -> User;
```

## `Auth @Service`

```typescript

// function authentificateUser(...) -> ...

// function authorizeUser(...) -> ...

```

## `Sessions @Service`

```typescript
/**
 *
 **/
function createSession(userId: Int) -> Session;

/**
 *
 **/
function writeSensorReadingsIntoSession(sessionId: Int, sensorReadings: SensorReading[]) -> Int;

/**
 *
 **/
function findSessionById(sessionId: Int) -> Session | Null;

/**
 *
 **/
function fetchSessionsForUser(userId: Int, take: Int, skip: Int) -> Session[];

/**
 *
 **/
function fetchSensorReadingsForSession(sessionId: Int, take: Int, skip: Int) -> SensorReading[];

/**
 *
 **/
function destroySession(sessionId: Int) -> Session;
```

## `Tasks @Service`
Soon...

## `Sessions @WebSocketGateway`

### Client-to-Server Events

#### ⛔ `event CreateSession`

```typescript
// add session metadata  

type CreateSessionEvent = {
  event: "create_session",
  data: {},
}
```


#### ⛔ `event WriteSensorReadingsBatch`

```typescript

type SensorReading = {
  timestamp: Date,
  sensor: "Accelerometer" | "Gyroscope" | "Magnetometer",
  xAxis: number, 
  yAxis: number,
  zAxis: number,
};

// See SENSOR_READINGS_BATCH_SIZE
// batch_id is a sequence number. It starts from 0 and increases by 1 every time the client sends a batch of sensor readings. Server needs to know `batch_id` to include in InvalidSensorReadingsBatchSize or InvalidSensorReadingSchema errors.
type WriteSensorReadingsBatchEvent = {
  event: "write_sensor_readings_batch",
  data: {
    batchId: number,
    sensorReadings: SensorReading[],
  },
}
```

#### ⛔ `event DestroySession`

```typescript
type DestroySessionEvent = {
  event: "destroy_session",
  data: {},
}
```

### Server-to-Client Events

#### ⛔ `event SessionCreated`

```typescript
type SessionCreatedEvent = {
  event: "session_created",
  data: {
    sessionId: number,
    createdAt: Date,
  },
}
```

#### ⛔ `error SessionAlreadyExist`

```typescript
type SessionAlreadyExistError = {
  event: "session_already_exist",
  data: {
    sessionId: number,
    createdAt: Date,
  },
}
```

#### ⛔ `event SensorReadingsBatchWritten`

```typescript
type SensorReadingsBatchWrittenEvent = {
  event: "sensor_readings_batch_written",
  data: {
    sessionId: number,
    totalSensorReadingsWritten: number,
  },
}
```

#### ⛔ `error InvalidSensorReadingsBatchSize`

```typescript
// ERROR
// The size of a sensor readings batch must be equal SENSOR_READINGS_BATCH_SIZE.
// SENSOR_READINGS_BATCH_SIZE is set as an environmental variable.
type InvalidSensorReadingsBatchSizeError = {
  event: "invalid_sensor_readings_batch_size",
  data: {
    sessionId: number,
    batchId: number,
  },
}
```

#### ⛔ `error InvalidSensorReadingSchema`

```typescript
type InvalidSensorReadingSchemaError = {
  event: "invalid_sensor_reading_schema",
  data: {
    sessionId: number,
    batchId: number,
  },
}
```

#### ⛔ `event SessionDestroyed`

```typescript
type SessionDestroyedEvent = {
  event: "session_destroyed",
  data: {
    sessionId: number,
    destroyedAt: Date,
    totalSensorReadingsWritten: number,
  },
}
```

#### ⛔ `error SessionNotFound`

```typescript
type SessionNotFoundError = {
  event: "session_not_found",
  data: {},
}
```

## `Users @Controller`
Soon...

## `Sessions @Controller`
Soon...

## `Tasks @Controller`
Soon...


## Special Notes

For Task #1, everything related to logic with WebSocketGateway should be done

We should assume that username is public_key while developing user authentification strategy.

Only BETATESTERs may start sessions. ADMINs may not start session, but they can manage users, manage tasks, fetch sessions etc.

If user sends more than ~ 300 batches, user should be banned

number_of_batches = 10 minutes * 60 seconds * 0.5 sensor_reading_batches_per_second = 300 batches

Admin
BetaTester

Example of e2e testing for a websocket 
https://github.com/nestjs/nest/blob/master/integration/websockets/e2e/gateway.spec.ts

Example of using Prisma for storing timesiries in PostgreSQL
https://github.com/dweb3s/super-bassoon

Stack: Nest.JS, TypeScript, Prisma


Not Available Yet ⛔

Deprecated 🚧 

Well-Tested And Ready-To-Use ✅ 


```typescript
// CONSTANTS

const WEBSOCKET_GATEWAY_CONNECTION_TIMEOUT = 600; // seconds
const WEBSOCKET_GATEWAY_CONNECTION_LIMIT = 50; // number of connections
const SENSOR_READINGS_BATCH_SIZE = 375; // number of sensor readings per one batch
```

Is it more clear when errors are separated from events? Or it is more clear when they follow each other i.e., there are an event and possible errors related to it?


If an unknown request sent to `Services @WebSocketServer`, tell the client

#### Should be a part of REST API
```
// onlyRole(PLAYER) can request sessions for themselves
{
  event: "fetch_sessions",
  data: {
    userId: number
  },
}


// onlyRole(ADMIN) can request sessions for any user by a given user_id
{
  event: "fetch_sessions_for_user",
  data: {
    userId: number
  },
}
```


## Client

### API

```typescript

// expo-permissions
// 


type SensorReading = {
  timestamp: Date,
  sensor: 0 | 1 | 2, // "Accelerometer" | "Gyroscope" | "Magnetometer"
  xAxis: number, 
  yAxis: number,
  zAxis: number,
};

type Session = {
  id: number,
  account: string,
  createdAt: Date,
  destroyedAt: Date,
  metadata: SessionMetadata, 
  totalSensorReadings: number,
};

const SENSOR_READINGS_BATCH_SIZE = 375;

class SessionLocalService() {
  - sessionId: number | null;

  // checks permissions
  constructor() -> Promise<void>;
  
  + createSession(sessionMetadata: SessionMetadata) -> Promise<Session>;

  + writeSensorReadingBatch(sensorReadings: SensorReading[]) -> Promise<void>;

  + destroySession() -> Promise<void>;
}

  + findSessionById(sessionId: number) -> Promise<Session>;

/**
 *
 **/
function createSession(userId: Int) -> Session;

/**
 *
 **/
function writeSensorReadingsIntoSession(sessionId: Int, sensorReadings: SensorReading[]) -> Int;

/**
 *
 **/
function findSessionById(sessionId: number) -> Session | null;

/**
 *
 **/
function fetchSessions(take: number, skip: number) -> Session[];
/**
 *
 **/
function destroySession() -> Promise<Session>;
}

```


### To-Do 

- Setup TimescaleDB
- Add support for session metadata
- Allow beta testers mark sessions that was recorded incorrectly. Also we need to setup a scipt for clearing such sessions from time to time.
- Change authentification strategy (web3 auth, verify signature)
- Write CLI for administration 


```typescript
// DO NOT IMPLEMENT THAT
sensorReadings: SensorReading[], // will be in the future    
sensorReadingsTake: number, // will be in the future
sensorReadingsSkip: number, // will be in the future
```

