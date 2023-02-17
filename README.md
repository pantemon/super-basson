# Backend For Motion Sensors Data Collection App

## Context

Для того, чтобы разработать модель, которая будет анализировать движение человека и определять едет ли человек на велосипеде, необходимо собрать показатели следующих датчиков: акселерометра, гироскопа & магнетометра.



## 

### Reference Prisma Schema

```prisma

// Models for Users business-logic

enum Sensor {
  ACCELEROMETER
  GYROSCOPE
  MAGNETOMETER
}

// Can have 1 session
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

// Can have 1 user
// Can have many sensor readings
model Session {
  id             Int              @id @default(autoincrement())
  user           User             @relation(fields: [userId], references: [id])
  userId         Int
  createdAt      DateTime         @default(now())
  destroyedAt    DateTime?
  sensorReadings SensorReading[]
}


// USERS

enum Role {
  ADMIN
  BETATESTER
}

// Can have many sessions
model User {
  id        Int       @id @default(autoincrement())
  name      String
  publicKey String    @unique
  email     String    @unique
  role      Role
  sessions  Session[]
}
```

### `service Users`
- create_user() -> int;
- fetch_users(take: int, skip: int) -> User[];
- find_user_by_public_key(user_public_key: string) -> User | null;
- delete_user_by_id(user_id: int) -> int;

### `service Sessions`

type Session = {
  
  
};

type SensorReading = {
  timestamp: Date,
  sensor_id: 0 | 1 | 2, // Accelerometer | Gyroscope | Magnetometer
  x: number,
  y: number,
  z: number,
};

```

type Session = {
  
};

type SensorReading = {
  
};


create_session(
  
) -> int;

fetch_sessions_for_user(user_id: int, take: int, skip: int) -> Session[];

find_session_by_id(session_id: int) -> Session | null;

destroy_session(session_id: int) -> int

write_sensor_readings_to_session(session_id: int, take: int, skip: int = 0);

fetch_sensor_readings_for_session(session_id: int, take: int, skip: int = 0) -> SensorReading[];
```


### `@WebSocketGateway`

```javascript

// onlyRole(PLAYER)
{
  event: "create_session",
  data: {},
}


// onlyRole(PLAYER)
// const BATCH_SIZE = 375; may be increased in the future
{
  event: "write_sensor_readings_batch",
  data: {
    batch_id: number,
    sensor_readings: [{
      timestamp: Date,
      sensor: "Accelerometer" | "Gyroscope" | "Magnetometer",
      xAxis: number, 
      yAxis: number,
      zAxis: number,
    }],
  },
}


// onlyRole(PLAYER)
{
  event: "destroy_session",
  data: {},
}



// SERVER EMITS
{
  event: "session_created",
  data: {
    sessionId: number,
    createdAt: Date,
  },
}

{
  event: "session_already_exist",
  data: {
    sessionId: number,
    createdAt: Date,
  },
}


{
  event: "sensor_readings_batch_written",
  data: {
    sessionId: number,
    totalSensorReadingsWritten: number,
  },
}

// ERROR
// The size of a sensor readings batch must be equal SENSOR_READINGS_BATCH_SIZE.
// SENSOR_READINGS_BATCH_SIZE is set as a environmental variable.
{
  event: "invalid_sensor_readings_batch_size",
  data: {
    sessionId: number,
    batchId: number,
  },
}

// ERROR
{
  event: "invalid_sensor_reading_schema",
  data: {
    sessionId: number,
    batchId: number,
  },
}


{
  event: "session_destroyed",
  data: {
    sessionId: number,
    destroyedAt: Date,
  },
}

{
  event: "session_not_found",
  data: {},
}


```

### Special Notes

For Task #1, everything related to logic with WebSocketGateway should be done


You should assume that password is public_key while developing user authentification strategy

Session should be destroyed after 10 minutes (no matter whether the client requested it or not)

Admin
BetaTester

After destroyment of the session, a number of written sensor readings should be 
If it is not the same as a number of sensor readings sent to a server, BetaTester should report it

Example of e2e testing for a websocket 
https://github.com/nestjs/nest/blob/master/integration/websockets/e2e/gateway.spec.ts

Stack: Nest.JS, TypeScript, Prisma


#### Should be REST API
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


### TR;DL

- Setup TimescaleDB
- Add support for session metadata
- Change authentification strategy (web3 auth, verify signature)
- Write CLI for administration e.g., creation of users, downloading sessions etc.




