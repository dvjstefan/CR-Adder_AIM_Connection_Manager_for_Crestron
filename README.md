# Adder AIM Connection Manager for Crestron

This project contains two Crestron SIMPL+ modules and a SimplSharp library designed to manage connections between Adder RX (receivers) and TX (transmitters) using the Adder AIM API. The solution supports connecting and disconnecting rooms (RX devices) to/from sources (TX devices), retrieving device lists, and channel management.
This module require Series 4 Processor.

---

## Modules

### 1. Adder Connection Manager

#### Description
The **Connection Manager** is the central controller module responsible for:
- Logging into the Adder AIM server.
- Retrieving device and channel lists.
- Managing room registrations.
- Processing room requests to connect or disconnect sources.
- Handling debug output.

#### Digital Inputs
| Signal Name   | Description |
| -------------- | ----------- |
| `Login`       | Pulse to initiate login to the Adder AIM server using the configured IP, port, username, and password. |
| `Logout`      | Pulse to log out from the AIM server. |
| `Debug`       | Hold high to enable detailed debug output to the Crestron console. |
| `GetDevicesTX` | Pulse to manually request an update of all available TX devices. |
| `GetDevicesRX` | Pulse to manually request an update of all available RX devices. |
| `GetChannels`  | Pulse to manually request an update of all available channels. |
| `Poll`        | Reserved for future use (optional periodic polling if needed). |
| `Disconnect_All` | Pulse to disconnect all RX devices from their current TX sources. |

#### Digital Outputs
| Signal Name      | Description |
| ---------------- | ----------- |
| `Login_Success` | Pulses high when login is successful. |
| `Login_Failed`  | Pulses high when login fails. |

#### Analog Parameters
| Signal Name   | Description |
| -------------- | ----------- |
| `HTTPS`       | Set to `1` to use HTTPS, `0` to use HTTP. |
| `API_Version` | Specifies the AIM API version to use in requests. |
| `AIM_Port`    | The port number to communicate with the AIM server. |

#### Serial Parameters
| Signal Name   | Description |
| -------------- | ----------- |
| `AIM_IP$`     | IP address of the Adder AIM server. |
| `Username$`   | Username for AIM login. |
| `Password$`   | Password for AIM login. |

---

### 2. Adder Room (RX) Module

#### Description
The **Room Module** represents an individual RX device (room) in the system. Each room must register itself with the Connection Manager after processor startup. The room module allows switching between sources (TX devices) or disconnecting from all sources.

#### Digital Inputs
| Signal Name    | Description |
| --------------- | ----------- |
| `RegisterRoom` | Must be pulsed after startup to register this room with the Adder Connection Manager. |
| `DisconnectRoom` | Pulse to disconnect this room's RX device from its current source (TX). |

#### Digital Outputs
| Signal Name       | Description |
| ----------------- | ----------- |
| `RoomIsRegistered` | High when the room has successfully registered with the Connection Manager. |

#### Serial Inputs
| Signal Name   | Description |
| -------------- | ----------- |
| `Source$`     | The name of the TX source to connect to this RX device. This name must match a known TX device name in the system. Sending an **empty string** (`""`) will disconnect the RX device from any source. |

#### Serial Outputs
| Signal Name       | Description |
| ----------------- | ----------- |
| `Active_Source_Name$` | Shows the name of the currently connected TX source. |

---

## Workflow Summary
1. **System Startup:**  
   - Adder Connection Manager starts and logs into the AIM server.
   - Each Adder Room module pulses `RegisterRoom` to register with the manager.
   - Connection Manager downloads the list of all devices and channels.

2. **Room Operation:**  
   - Each room can set `Source$` to request connection to a source.
   - Each room can send an empty `Source$` to disconnect.
   - Room status (connected source name) is automatically updated when changes occur.

3. **Debugging:**  
   - Hold the `Debug` signal high to get detailed logs to the Crestron console.

---

## Example Flow - Room Connection
1. Room 801 registers itself.
2. Room 801 sends `Source$ = "TX1"` to connect to TX1.
3. Connection Manager looks up TX1 in its channel map.
4. Connection Manager sends the API command to connect RX801 to TX1.
5. Room 801 receives `Active_Source_Name$ = "TX1"` if successful.

---

## Example Flow - Room Disconnection
1. Room 801 sends `Source$ = ""` (empty string).
2. Connection Manager sends the API command to disconnect RX801.
3. Room 801 receives `Active_Source_Name$ = "No Source"` if successful.

---

## Notes
- Each room must be registered every time the processor restarts.
- All room operations (connect, disconnect) are managed centrally by the Connection Manager.
- Debug output can be enabled per room and per manager.
- Error handling is included for cases such as invalid credentials, unreachable server, and connection failures.

---

## Author
This module was developed for Crestron systems to provide seamless integration with Adder AIM KVM matrix.

---
