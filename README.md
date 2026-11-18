# Three-Floor Elevator Control with Siemens S7-1200

This project simulates a single elevator serving three floors. The control program uses Ladder Diagram (LAD) and runs on a Siemens SIMATIC S7-1200 CPU emulated by S7-PLCSIM. Factory I/O models the push buttons, position sensors, elevator drive, and floor-level actuators.

![Three-floor elevator model in Factory I/O](docs/factory-io-elevator.jpg)

## Project files

| File | Description |
| --- | --- |
| [TIA Portal V21 project](tia-portal/FactoryIO_Template_S7-1200_V20_V21.zip) | ZIP archive containing the STEP 7 project folder and its `.ap21` project file. |
| [Elevator Project scene](factory-io/Elevator%20Project.factoryio) | Factory I/O scene with the Siemens S7-PLCSIM driver mapping. |

## Control logic

1. **System enable.** The START push button latches an internal enable bit through a set coil. The STOP push button provides the input to the reset logic.
2. **Floor requests.** Each floor has two push-button inputs. When the system is enabled, either input latches a floor request through a set coil. Separate networks select and clear the target floor and reset stored requests.
3. **Travel direction and position.** Capacitive sensors indicate the elevator's position at each of the three floors. The up and down networks evaluate the current position and target floor. Each network checks that the opposite direction is inactive before setting its output. The program also records the last detected floor sensor and the previous travel direction.
4. **Door actuation.** The program uses MOVE instructions to write a door-state code. The output network checks the elevator position and the code to control the pusher at the corresponding floor. The `TIMER DOOR DELAY` network introduces a delay before resetting the door state.

This description is based on the LAD network structure, the stored PLC symbols, and the scene's I/O mapping. The service order for simultaneous requests still requires simulation testing.

## I/O mapping

Addresses follow the PLC addressing used by the Siemens S7-PLCSIM driver. The scene sets the Boolean input and output offsets to zero.

| PLC address | Program function | Factory I/O component |
| --- | --- | --- |
| `%I0.0` | START | Push Button (NO) 7 |
| `%I0.1` | STOP | Push Button (NC) 0 |
| `%I0.2`–`%I0.4` | GET FLOOR 1–3 | Push Button (NO) 3, 4, 5 |
| `%I0.5`–`%I0.7` | BUTTON FLOOR 1–3 | Push Button (NO) 0, 1, 2 |
| `%I1.0`–`%I1.2` | Floor position sensors 1–3 | Capacitive Sensor 0, 1, 2 |
| `%Q0.0` | Elevator up | Elevator 0 (Up) |
| `%Q0.1` | Elevator down | Elevator 0 (Down) |
| `%Q0.2`–`%Q0.4` | Door actuation for floors 1–3 | Pusher 1, 0, 2 |

## Running the simulation

The project requires TIA Portal V21 with STEP 7, S7-PLCSIM V21, and Factory I/O Siemens Edition or Ultimate Edition with support for the S7-PLCSIM driver. The configured PLC is a CPU 1211C DC/DC/DC. The scene was saved in Factory I/O 2.5.10.

1. Extract the entire ZIP archive and open `FactoryIO_Template_S7-1200_V20_V21.ap21` in TIA Portal V21.
2. Select the CPU and choose **Start simulation**. Load the configuration and program into S7-PLCSIM, then set the CPU to **RUN**. Keep Network 1 in OB1 because the template uses it to communicate with Factory I/O.
3. Open `Elevator Project.factoryio`. In **File → Driver Configuration**, select **Siemens S7-PLCSIM** and the **S7-1200 (V20-21)** model. Click **Connect** and run the scene.
4. Press START and request each floor. Monitor the floor sensors, the up and down outputs, and the pusher at the destination. Test the STOP button and enter consecutive floor requests to observe the changes in state.

For detailed configuration instructions, see the [S7-PLCSIM V20–21 setup guide](https://docs.factoryio.com/tutorials/siemens/setting-up-s7-plcsim-v20/) and the [driver reference](https://docs.factoryio.com/manual/drivers/s7-plcsim/).

## License

The [MIT License](LICENSE) applies to the original work by Yunachz, including the elevator logic, scene design, and documentation. The TIA Portal project uses the Factory I/O integration template. The template, built-in simulator components, and other third-party materials remain subject to their respective terms.
