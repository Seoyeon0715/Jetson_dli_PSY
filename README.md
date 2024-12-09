## 아두이노 토양수분센서 코드를 바탕으로 function calling def 정의하기
해당 코드는 jupyter notebook에서 실행한다.

**함수명: moist_sensor_info**

```bash
import pandas as pd
import serial
import time

def moisture_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    토양 수분 센서 데이터를 처리하고 반환하는 함수.

    Args:
        mode (str): 'real-time'이면 실시간으로 데이터를 읽고,
                    'file'이면 엑셀 파일에서 데이터를 읽음.
        file_path (str): 엑셀 파일 경로 (mode가 'file'일 때 필수).
        port (str): 아두이노 직렬 포트 (실시간 데이터 수집용).
        baudrate (int): 직렬 통신 속도.

    Returns:
        str: 토양 수분 정보 또는 오류 메시지를 문자열로 반환.
    """
    if mode == 'real-time':
        try:
            # 직렬 통신 설정
            arduino = serial.Serial(port, baudrate, timeout=1)
            if arduino.in_waiting > 0:
                data = arduino.readline().decode('utf-8').strip()  # 데이터 읽기
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")   # 현재 시간
                arduino.close()
                return str({'timestamp': timestamp, 'moisture': f"{data}%"})
            else:
                return str({'error': 'No data available from sensor.'})
        except Exception as e:
            return str({'error': str(e)})

    elif mode == 'file':
        try:
            if not file_path:
                return str({'error': 'File path is required in file mode.'})
            # 엑셀 파일 읽기
            df = pd.read_excel(file_path)
            if df.empty:
                return str({'error': 'No data available in the file.'})
            # 가장 최근 데이터 반환
            latest_entry = df.iloc[-1].to_dict()
            return str({
                'timestamp': latest_entry['Timestamp'],
                'moisture': f"{latest_entry['Moisture (%)']}%"
            })
        except Exception as e:
            return str({'error': str(e)})

    else:
        return str({'error': 'Invalid mode. Use "real-time" or "file".'})
```

## 토양수분센서의 use_functions 작성

```bash
use_functions = [
    {
        "type": "function",
        "function": {
            "name": "moisture_sensor_info",
            "description": "Retrieves soil moisture data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from an Excel file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the Excel file containing soil moisture data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    }
]
```

### Explanation

#### **Name**
- 함수 이름은 `moisture_sensor_info`로 지정되었습니다.

#### **Description**
- 이 함수는 실시간 센서 데이터 또는 파일에서 데이터를 가져옵니다.

#### **Parameters**
입력 매개변수는 다음과 같습니다:

- **`mode`**:
  - `'real-time'` 또는 `'file'` 값 중 하나를 선택할 수 있습니다.
  - 필수 입력 값입니다.

- **`file_path`**:
  - `mode='file'`일 때 필요한 파일 경로입니다.
  - `nullable=True`로 설정해 `mode='real-time'`일 경우 생략 가능합니다.

- **`port`**:
  - `mode='real-time'`일 때 사용되는 직렬 포트 경로입니다.
  - 기본값은 `/dev/ttyUSB0`입니다.
  - `nullable=True`로 설정해 `mode='file'`일 경우 생략 가능합니다.

- **`baudrate`**:
  - 직렬 통신의 속도를 설정하며, 기본값은 `9600`입니다.
  - `nullable=True`로 설정해 `mode='file'`일 경우 생략 가능합니다.

#### **Required**
- **`mode`**는 필수 입력 값으로 지정되었습니다.







## Function Calling Definition: 조도 센서 모듈
Python Function 정의. jupyter notebook에서 실행.

**함수명 : light_sensor_info**

```bash
import pandas as pd
import serial
import time

def light_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    조도 센서 데이터를 처리하고 반환하는 함수.

    Args:
        mode (str): 'real-time'이면 실시간으로 데이터를 읽고,
                    'file'이면 엑셀 파일에서 데이터를 읽음.
        file_path (str): 엑셀 파일 경로 (mode가 'file'일 때 필수).
        port (str): 아두이노 직렬 포트 (실시간 데이터 수집용).
        baudrate (int): 직렬 통신 속도.

    Returns:
        str: 조도 센서 정보 또는 오류 메시지를 문자열로 반환.
    """
    if mode == 'real-time':
        try:
            # 직렬 통신 설정
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)  # 포트 안정화
            if arduino.in_waiting > 0:
                data = arduino.readline().decode("utf-8").strip()  # 데이터 읽기
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")    # 현재 시간
                arduino.close()
                return str({'timestamp': timestamp, 'brightness': f"{data.split(': ')[1]}%"})
            else:
                return str({'error': 'No data available from sensor.'})
        except Exception as e:
            return str({'error': str(e)})

    elif mode == 'file':
        try:
            if not file_path:
                return str({'error': 'File path is required in file mode.'})
            # 엑셀 파일 읽기
            df = pd.read_excel(file_path)
            if df.empty:
                return str({'error': 'No data available in the file.'})
            # 가장 최근 데이터 반환
            latest_entry = df.iloc[-1].to_dict()
            return str({
                'timestamp': latest_entry['Timestamp'],
                'brightness': f"{latest_entry['Brightness (%)']}%"
            })
        except Exception as e:
            return str({'error': str(e)})

    else:
        return str({'error': 'Invalid mode. Use "real-time" or "file".'})
```

## use_functions 정의
함수 호출을 JSON 형식으로 설명하여 함수 호출 메타데이터를 제공함.

```bash
use_functions = [
    {
        "type": "function",
        "function": {
            "name": "light_sensor_info",
            "description": "Retrieves light sensor data either in real-time from a sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the sensor or 'file' for data from an Excel file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the Excel file containing brightness data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time'). Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    }
]
```

### 함수 이름
- light_sensor_info : 조도 센서 데이터를 처리하는 함수.
### 매개변수
- mode: 센서 데이터를 실시간(real-time)으로 읽을지, 파일(file)에서 읽을지를 결정.
- file_path: 파일 모드일 때 필요한 파일 경로.
- port: 실시간 모드에서 아두이노 직렬 포트를 지정.
- baudrate: 실시간 모드에서 직렬 통신 속도를 설정.
### 반환 값
- JSON 형식으로 타임스탬프와 밝기 데이터 반환.
- 오류 발생 시 오류 메시지를 포함한 JSON 반환.



## 온습도 센서_ Function Calling Definition

```bash
def execute_function_call(function_data, args):
    """
    Executes a function based on provided metadata and arguments.

    Args:
        function_data (dict): Metadata for the function to call.
        args (dict): Arguments for the function call.

    Returns:
        str: The result of the function execution.
    """
    function_name = function_data["function"]["name"]
    
    if function_name == "dht11_sensor_info":
        return dht11_sensor_info(
            mode=args.get("mode"),
            file_path=args.get("file_path"),
            port=args.get("port", "/dev/ttyUSB0"),
            baudrate=args.get("baudrate", 9600)
        )
    else:
        return str({"error": f"Function {function_name} is not implemented."})
```

## 4. use_functions JSON 정의

```bash
use_functions = [
    {
        "type": "function",
        "function": {
            "name": "dht11_sensor_info",
            "description": "Retrieves temperature and humidity data either in real-time from a DHT11 sensor or from a file.",
            "parameters": {
                "type": "object",
                "properties": {
                    "mode": {
                        "type": "string",
                        "description": "'real-time' for live data from the DHT11 sensor or 'file' for data from an Excel file.",
                        "enum": ["real-time", "file"]
                    },
                    "file_path": {
                        "type": "string",
                        "description": "The path to the Excel file containing temperature and humidity data (required if mode is 'file').",
                        "nullable": True
                    },
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the sensor is connected (required if mode is 'real-time').",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    }
                },
                "required": ["mode"]
            }
        }
    }
]
```




