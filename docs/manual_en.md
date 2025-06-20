
# Documentation

## Kea2's tutorials 

1. A small tutorial of applying Kea2's Feature 2 and 3 on [WeChat](docs/Scenario_Examples_zh.md).

## Kea2's scripts

Kea2 uses [Unittest](https://docs.python.org/3/library/unittest.html) to manage scripts. All the Kea2's scripts can be found in unittest's rules (i.e., the test methods should start with `test_`, the test classes should extend `unittest.TestCase`).

Kea2 uses [Uiautomator2](https://github.com/openatx/uiautomator2) to manipulate android devices. Refer to [Uiautomator2's docs](https://github.com/openatx/uiautomator2?tab=readme-ov-file#quick-start) for more details.

Basically, you can write Kea2's scripts by following two steps:

1. Create a test class which extends `unittest.TestCase`.

```python
import unittest

class MyFirstTest(unittest.TestCase):
    ...
```

2. Write your own script by defining test methods

By default, only the test method starts with `test_` will be found by unittest. You can decorate the function with `@precondition`. The decorator `@precondition` takes a function which returns boolean as an arugment. When the function returns `True`, the precondition is satisified and the script will be activated, and Kea2 will run the script based on certain probability defined by the decorator `@prob`.

Note that if a test method is not decorated with `@precondition`.
This test method will never be activated during automated UI testing, and will be treated as a normal `unittset` test method.
Thus, you need to explicitly specify `@precondition(lambda self: True)` when the test method should be always executed. When a test method is not decorated with `@prob`, the default probability is 1 (always execute when precondition satisfied).

```python
import unittest
from kea2 import precondition

class MyFirstTest(unittest.TestCase):

    @prob(0.7)
    @precondition(lambda self: ...)
    def test_func1(self):
        ...
```

You can read [Kea - Write your fisrt property](https://kea-docs.readthedocs.io/en/latest/part-keaUserManuel/first_property.html) for more details.


## Launching Kea2

We offer two ways to launch Kea2.

### 1. Launch by shell commands

Kea2 is compatible with `unittest` framework. You can manage your test cases in unittest style. You can launch Kea2 with `kea run` with driver options and sub-command `unittest` (for unittest options).

The shell command:
```
kea2 run <Kea2 cmds> unittest <unittest cmds> 
```

Sample shell commands:

```bash
# Launch Kea2 and load one single script quicktest.py.
kea2 run -s "emulator-5554" -p it.feio.android.omninotes.alpha --agent u2 --running-minutes 10 --throttle 200 --driver-name d unittest discover -p quicktest.py

# Launch Kea2 and load multiple scripts from the directory mytests/omni_notes
kea2 run -s "emulator-5554" -p it.feio.android.omninotes.alpha --agent u2 --running-minutes 10 --throttle 200 --driver-name d unittest discover -s mytests/omni_notes
```

Options:

| arg | meaning | default | 
| --- | --- | --- |
| -s | The serial of your device, which can be found by `adb devices` | |
| -p | The tested app's package name (e.g., com.example.app) | 
| -o | The ouput directory for logs and results | `output` |
| --agent |  {native, u2}. By default, `u2` is used and supports all the three important features of Kea2. If you hope to run the orignal Fastbot, please use `native`.| `u2` |
| --running-minutes | The time (in minutes) to run Kea2 | `10` |
| --max-step | The maxium number of monkey events to send (only available in `--agent u2`) | `inf` (infinite) |
| --throttle | The delay time (in milliseconds) between two monkey events | `200` |
| --driver-name | The name of driver used in the kea2's scripts. If `--driver-name d` is specified, you should use `d` to interact with a device, e..g, `self.d(..).click()`. |
| --log-stamp | the stamp for log file and result file. (e.g., if `--log-stamp 123` is specified, the log files will be named as `fastbot_123.log` and `result_123.json`.) | current time stamp |
| unittest | Specify to load which scripts. This  sub-command `unittest` is fully compatible with unittest. See `python3 -m unittest -h` for more options of unittest. This option is only available in `--agent u2`.


### 2. Launch by `unittest.main`

Like unittest, we can launch Kea2 through the method `unittest.main`.

Here is an example (named as `mytest.py`). You can see that the options are directly defined in the script.

```python
import unittest

from kea2 import KeaTestRunner, Options
from kea2.u2Driver import U2Driver

class MyTest(unittest.TestCase):
    ...
    # <your test methods here>

if __name__ == "__main__":
    KeaTestRunner.setOptions(
        Options(
            driverName="d",
            Driver=U2Driver,
            packageNames=[PACKAGE_NAME],
            # serial="emulator-5554",   # specify the serial
            maxStep=100,
            # running_mins=10,  # specify the maximal running time in minutes, default value is 10m
            # throttle=200,   # specify the throttle in milliseconds, default value is 200ms
            # agent='native'  # 'native' for running the vanilla Fastbot
        )
    )
    # Declare the KeaTestRunner
    unittest.main(testRunner=KeaTestRunner)
```

We can directly run the script `mytest.py` to launch Kea2, e.g.,
```python
python3 mytest.py
```

Here's all the available options in `Options`.

```python
# the driver_name in script (if self.d, then d.) 
driverName: str
# the driver (only U2Driver available now)
Driver: U2Driver
# list of package names. Specify the apps under test
packageNames: List[str]
# target device
serial: str = None
# test agent. "u2" is the default agent
agent: "u2" | "native" = "u2"
# max step in exploration (availble in stage 2~3)
maxStep: int # default "inf"
# time(mins) for exploration
running_mins: int = 10
# time(ms) to wait when exploring the app
throttle: int = 200
# the output_dir for saving logs and results
output_dir: str = "output"
# the stamp for log file and result file, default: current time stamp
log_stamp: str = None
```

## Examining the running statistics of scripts .

If you want to examine whether your scripts have been executed or how many times they have been executed during testing. Open the file `result.json` after the testing is finished.

Here's an example.

```json
{
    "test_goToPrivacy": {
        "precond_satisfied": 8,
        "executed": 2,
        "fail": 0,
        "error": 1
    },
    ...
}
```

**How to read `result.json`**

Field | Description | Meaning
--- | --- | --- |
precond_satisfied | During exploration, how many times has the test method's precondition been satisfied? | Does we reach the state during exploration? 
executed | During UI testing, how many times the test method has been executed? | Has the test method ever been executed?
fail | How many times did the test method fail the assertions during UI testing? | When failed, the test method found a likely functional bug. 
error | How many times does the test method abort during UI tsting due to some unexpected errors (e.g. some UI widgets used in the test method cannot be found) | When some error happens, the script needs to be updated/fixed because the script leads to some unexpected errors.

## Configuration File

After executing `Kea2 init`, some configuration files will be generated in the `configs` directory. 
These configuration files belong to `Fastbot`, and their specific introductions are provided in [Introduction to configuration files](https://github.com/bytedance/Fastbot_Android/blob/main/handbook-cn.md#%E4%B8%93%E5%AE%B6%E7%B3%BB%E7%BB%9F).