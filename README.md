# rbus-datamodels

The `rbus-datamodels` project provides a C-based executable (`rbus-datamodels`) that exposes data models via the RBus library. It registers data models (e.g., `Device.DeviceInfo.SerialNumber`, `Device.DeviceInfo.MemoryStatus.Total`) from a JSON file (`datamodels.json`) and predefined models, supporting various RBus value types such as strings, integers, booleans, and more. The executable acts as an RBus provider, allowing clients (e.g., `rbuscli`) to get/set properties and subscribe to events.

## Prerequisites

- **CMake**: Version 3.10 or higher.
- **RBus**: Install `rbus`.
  - On macOS:

    ```bash
    brew tap stepherg/tap
    brew install rbus
    ```

  - On Linux: Install from source.
  
- **cJSON**: Install the `cjson` library.
  - On macOS: will be installed as a dependency of `brew install rbus`

  - On Linux: Install via package manager (e.g., `libjson-c-dev` on Ubuntu).
- **Build Tools**: `gcc`, `make`.

## Building the Project

To build the `rbus-datamodels` executable, follow these steps:

1. **Create Build Directory**:

   ```bash
   cd ./rbus-datamodels
   mkdir build && cd build
   ```

2. **Run CMake**:

   ```bash
   cmake ..
   ```

   This configures the build using `CMakeLists.txt`, linking against `librbus`, `rbuscore`, and `cjson`. Ensure `datamodels.json` is in the project root.

3. **Build the Executable**:

   ```bash
   make
   ```

   This compiles `rbus-datamodels.c` into the `rbus-datamodels` executable and copies `datamodels.json` to the build directory.

4. **Run the Executable**:

   ```bash
   ./rbus-datamodels
   ```

   The executable registers data models and listens for RBus events, printing "Successfully registered X data models". It runs until interrupted (Ctrl+C) or sigterm.

### Examples with `rbuscli`

Use `rbuscli` in a separate terminal to interact with the running `rbus-datamodels` provider. Ensure `rbuscli` is installed (typically included with RBus).

#### 1. Get a Property Value
Retrieve the value of a data model property, e.g., `Device.DeviceInfo.SerialNumber`.

```bash
rbuscli get Device.DeviceInfo.SerialNumber
```

**Expected Output**:

```bash
Parameter  1:
              Name  : Device.DeviceInfo.SerialNumber
              Type  : string
              Value : (e.g., 1234567890AB)
```

#### 2. Set a Property Value
Set a writable property defined in `datamodels.json` (e.g., `Device.Test.Property`). Note: Predefined properties like `Device.DeviceInfo.SerialNumber` are read-only.

```bash
rbuscli set Device.Test.Property string TestValue
```

**Expected Output**:

```bash
setvalues succeeded..
```

Verify the set value:

```bash
rbuscli get Device.Test.Property
```

**Expected Output**:

```bash
Parameter  1:
              Name  : Device.Test.Property
              Type  : string
              Value : TestValue
```

#### 3. Test subscriptions

Go into the rbuscli interactive shell by running:
`rbuscli -i`

Run the following commands within the shell:
`sub Device.Test.Property != "test"`
`set Device.Test.Property string test2`

**Expected Output**:

```bash
rbuscli> sub Device.Test.Property != "test"
rbuscli> set Device.Test.Property string test2
setvalues succeeded..
rbuscli> Event received Device.Test.Property of type RBUS_EVENT_VALUE_CHANGED
Event data:
  rbusObject name=
   rbusProperty name=value
    rbusValue type:RBUS_STRING value:test2
   rbusProperty name=oldValue
    rbusValue type:RBUS_STRING value:test
   rbusProperty name=by
    rbusValue type:RBUS_STRING value:rbuscli-27002
   rbusProperty name=filter
    rbusValue type:RBUS_BOOLEAN value:true

User data: sub Device.Test.Property != test
```

## Notes

- **Data Models**: `rbus-datamodels` loads data models from `datamodels.json` and predefined models in `rbus-datamodels.c`. The default location for `datamodels.json` is in the same directory as the executable.
- **Read-Only Properties**: Predefined properties in `rbus-datamodels.c` (e.g., `Device.DeviceInfo.SerialNumber`, `Device.DeviceInfo.MemoryStatus.Total`) are read-only, as they lack `setHandler` implementations.
- **Event Handling**: The `valueChangeHandler` in `rbus-datamodels.c` logs value changes for subscribed properties, visible in the `rbus-datamodels` terminal output.

## Troubleshooting

- **Build Errors**:
  - Ensure `librbus`, `rbuscore`, and `cjson` are installed:
    ```bash
    ls /usr/lib | grep rbus
    ls /usr/lib | grep cjson
    ```
  - Check CMake errors:
    ```bash
    cat CMakeFiles/CMakeError.log
    ```
  - On macOS, verify Homebrew paths:
    ```bash
    ls /opt/homebrew/lib | grep cjson
    ```

- **Runtime Errors**:
  - Set `LD_LIBRARY_PATH`:
    ```bash
    export LD_LIBRARY_PATH=/usr/lib:/usr/local/lib:$LD_LIBRARY_PATH
    ```
  - Check `rbus-datamodels` logs for RBus errors:
    ```bash
    ./rbus-datamodels 2> stderr.log
    grep "rbus error" stderr.log
    ```
  - Ensure `datamodels.json` is valid and present in the build directory:
    ```bash
    ls build/datamodels.json
    ```

- **Event or Property Issues**:
  - Verify `rbus-datamodels` is running.
  - Check property existence:
    ```bash
    rbuscli get Device.DeviceInfo.SerialNumber
    ```
  - Ensure properties in `datamodels.json` are correctly formatted (e.g., valid `name`, `type`, and `value` fields).
  - If events don’t trigger, confirm the property is writable and supports events (e.g., defined with `eventSubHandler` in `rbus-datamodels.c`).

