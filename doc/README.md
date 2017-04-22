# Protocol

All integers are encoded in little endian. All strings are UTF-8. All floats are encoded according to [IEEE 754](https://en.wikipedia.org/wiki/IEEE_floating_point). Booleans (abbreviated "bool") are bytes; they are 0 for false, 1 for true.

Whenever there's an error field, it can be an empty string to indicate success.

## Initial connection

During this stage, the client initiates a connection and requests an environment. The server attempts to create the environment, or fails with an error (e.g. if the environment does not exist).

|Source   |Type    | Description           |
|---------|--------|-----------------------|
|Client   |uint8   | Flags (all 0)         |
|Client   |uint32  | Length of env name    |
|Client   |string  | Environment name      |
|Server   |uint32  | Error length          |
|Server   |string  | Error message         |

## Command packets

Once the handshake has completed, the client may send commands and receive responses. Only one command can be run at once. All packets take the following form:

|Source   |Type    | Description           |
|---------|--------|-----------------------|
|Client   |uint8   | Packet type           |
|Client   |varies  | Packet data           |

Valid packet types are listed below.

### Packet: Reset

This is packet type 0.

This packet resets the environment and gets the initial observation. It can be used as follows:

|Source   |Type                         | Description           |
|---------|-----------------------------|-----------------------|
|Client   |uint8                        | Packet type (0)       |
|Server   |[observation](#observations) | Initial observation   |

### Packet: Step

This is packet type 1.

This packet takes a step in the environment and gets a lot of information back. It can be used as follows:

|Source   |Type                         | Description           |
|---------|-----------------------------|-----------------------|
|Client   |uint8                        | Packet type (1)       |
|Client   |[action](#actions)           | Action to take        |
|Server   |[observation](#observations) | Next observation      |
|Server   |float64                      | Reward                |
|Server   |bool                         | Done                  |
|Server   |uint32                       | Info length           |
|Server   |string                       | Info JSON             |

## Actions

Actions are encoded in a type-specific manner. They are of the form:

|Type    | Description           |
|--------|-----------------------|
|uint8   | Action type ID        |
|uint32  | Length of data        |
|varies  | Data                  |

The available action types are listed below.

### Action: JSON

This is action type 0.

The data inside the action is a JSON string for the space's `from_jsonable` method.

## Observations

Observations are encoded in a type-specific manner. They are of the form:

|Type    | Description           |
|--------|-----------------------|
|uint8   | Observation type ID   |
|uint32  | Length of data        |
|varies  | Data                  |

The available observation types are listed below.

### Observation: JSON

This is observation type 0.

The data in the packet is a JSON string from the space's `to_jsonable` method.

### Observation: Byte List

This is observation type 1.

The data in the packet is a flattened array of bytes. This observation has the following format:

|Type     | Description           |
|---------|-----------------------|
|uint32   | Num dimensions        |
|uint32[] | Dimensions            |
|uint8[]  | Data                  |

This is for observations in things like Atari environments where the observation is a raw 3D array of bytes. The array of bytes is flattened (in C order) into a 1D list of bytes.