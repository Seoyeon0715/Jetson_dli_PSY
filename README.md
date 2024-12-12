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




# 아두이노에서 실행해서 되었던 코드들도...




# 1. Soil Moisture Sensor Code
```bash
// Soil Moisture Sensor Code for Arduino to A1 Communication
// 토양 수분 센서 Arduino-A1 통신 코드

const int sensorPin = A1;  // Analog pin for soil moisture sensor
const int dryValue = 520;  // Calibrated dry soil value
const int wetValue = 260;  // Calibrated wet soil value

void setup() {
  Serial.begin(9600);  // Initialize serial communication
  // A1과의 통신을 위해 시리얼 통신 초기화
  
  pinMode(sensorPin, INPUT);  // Set sensor pin as input
  // 센서 핀을 입력 모드로 설정
}

void loop() {
  // Read and average sensor value to reduce noise
  // 노이즈 감소를 위해 센서 값 평균 계산
  int sensorValue = getAverageSensorValue(sensorPin, 10);
  
  // Map sensor value to moisture percentage
  // 센서 값을 0-100% 수분 퍼센트로 변환
  int moisturePercent = map(sensorValue, dryValue, wetValue, 0, 100);
  
  // Constrain moisture percentage to 0-100 range
  // 수분 퍼센트를 0-100 범위로 제한
  moisturePercent = constrain(moisturePercent, 0, 100);
  
  // Prepare JSON-like message for A1 device
  // A1 장치를 위한 JSON 형식 메시지 준비
  String message = "{\"sensor\":\"soil_moisture\",\"value\":" + 
                   String(moisturePercent) + 
                   ",\"status\":\"";
  
  // Determine moisture status
  // 수분 상태 판단
  if (moisturePercent < 30) {
    message += "critical_low\"}";
    Serial.println(message);
  } else if (moisturePercent < 40) {
    message += "low\"}";
    Serial.println(message);
  } else if (moisturePercent >= 40 && moisturePercent <= 60) {
    message += "optimal\"}";
    Serial.println(message);
  } else if (moisturePercent > 60 && moisturePercent <= 90) {
    message += "high\"}";
    Serial.println(message);
  } else {
    message += "flooded\"}";
    Serial.println(message);
  }
  
  // Wait for 5 seconds between readings
  // 5초마다 readings 수행
  delay(5000);
}

// Function to get average sensor value
// 평균 센서 값을 얻는 함수
int getAverageSensorValue(int pin, int samples) {
  long total = 0;
  for (int i = 0; i < samples; i++) {
    total += analogRead(pin);
    delay(10);  // Small delay between readings
  }
  return total / samples;
}
```



# 2. Light Intensity Sensor Code
```bash
const int lightSensorPin = 8; // Light sensor connected to analog pin

// Number of samples for averaging
const int NUM_SAMPLES = 10;

void setup() {
  Serial.begin(9600); // Initialize serial communication
  pinMode(lightSensorPin, INPUT); // Set light sensor pin as input
}

// Function to get stable and averaged sensor reading
int getStableLightReading() {
  long total = 0;
  int samples[NUM_SAMPLES];
  
  // Collect samples
  for (int i = 0; i < NUM_SAMPLES; i++) {
    samples[i] = analogRead(lightSensorPin);
    delay(10); // Small delay between readings
  }
  
  // Sort samples
  for (int i = 0; i < NUM_SAMPLES - 1; i++) {
    for (int j = i + 1; j < NUM_SAMPLES; j++) {
      if (samples[i] > samples[j]) {
        // Swap values
        int temp = samples[i];
        samples[i] = samples[j];
        samples[j] = temp;
      }
    }
  }
  
  // Remove extreme values (lowest and highest)
  long sum = 0;
  for (int i = 2; i < NUM_SAMPLES - 2; i++) {
    sum += samples[i];
  }
  
  // Calculate average
  return sum / (NUM_SAMPLES - 4);
}

void loop() {
  // Get stable light reading
  int sensorValue = getStableLightReading();
  
  // Convert to percentage with more precise mapping
  int brightnessPercent = map(sensorValue, 0, 1023, 0, 100);
  
  // Additional interpretation
  String lightIntensity;
  if (brightnessPercent < 20) {
    lightIntensity = "Dark";
  } else if (brightnessPercent < 40) {
    lightIntensity = "Low Light";
  } else if (brightnessPercent < 60) {
    lightIntensity = "Moderate Light";
  } else if (brightnessPercent < 80) {
    lightIntensity = "Bright";
  } else {
    lightIntensity = "Very Bright";
  }
  
  // Send data via serial communication
  Serial.print("Raw Value: ");
  Serial.print(sensorValue);
  Serial.print(" | Brightness: ");
  Serial.print(brightnessPercent);
  Serial.print("% | Condition: ");
  Serial.println(lightIntensity);
  
  delay(1000); // Wait for 1 second between readings
}
```

# 3. dht11 temp & humidity Sensor Code
```bash
#include <SimpleDHT.h>

// Pin number where the DHT data pin is connected
int pinDHT = 2;

// Initialize the DHT11 sensor using the SimpleDHT library
SimpleDHT11 dht11;

void setup() {
  Serial.begin(9600);
  Serial.println("Starting DHT11 Test!");
}

void loop() {
  // Variables to store temperature and humidity data
  byte temperature = 0;
  byte humidity = 0;

  // Read data from the DHT11 sensor
  int err = dht11.read(pinDHT, &temperature, &humidity, NULL);
  if (err != SimpleDHTErrSuccess) {
    Serial.print("Read error: ");
    Serial.println(err);
    delay(1000);
    return;
  }

  // Print humidity and temperature values
  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print("% ");
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println("C");

  delay(2000); // Wait for 2 seconds
}
```


#4. 토양수분센서 csv 파일로 불러오는 작업 코드 (jupyter notebook 가상환경에서 작업함)


import serial
import time
import pandas as pd

# 직렬 통신 설정
port = "/dev/ttyUSB0"  # 아두이노의 직렬 포트
baudrate = 9600        # 아두이노와 동일한 보드레이트
arduino = serial.Serial(port, baudrate, timeout=1)

# 데이터 저장용 리스트
data = []

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # 데이터가 직렬 포트에 들어왔는지 확인
            line = arduino.readline().decode("utf-8").strip()  # 데이터 읽기 및 디코딩
            timestamp = time.strftime("%Y-%m-%d %H:%M:%S")    # 현재 시간 생성
            print(f"{timestamp}, Moisture: {line}%")
            
            # 데이터 저장
            data.append({"Timestamp": timestamp, "Moisture (%)": line})

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # 데이터프레임 생성
    df = pd.DataFrame(data)

    # 데이터프레임을 CSV 파일로 저장
    df.to_csv("sensor_data.csv", index=False, encoding="utf-8")
    print("Data saved to sensor_data.csv")

    
    # 포트 닫기
    arduino.close()


#5. 토양수분센서+조도센서 csv 파일로 불러오는 작업 코드 (jupyter notebook 가상환경에서 작업함)

import serial
import time
import pandas as pd

# 직렬 통신 설정
port = "/dev/ttyUSB0"  # 아두이노의 직렬 포트
baudrate = 9600        # 아두이노와 동일한 보드레이트
arduino = serial.Serial(port, baudrate, timeout=1)

# 데이터 저장용 리스트
data = []

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # 데이터가 직렬 포트에 들어왔는지 확인
            line = arduino.readline().decode("utf-8").strip()  # 데이터 읽기 및 디코딩
            values = line.split(",")  # 아두이노에서 데이터가 'Moisture,Light' 형식으로 들어온다고 가정
            if len(values) == 2:  # 데이터가 제대로 들어왔는지 확인
                moisture = values[0].strip()  # 토양 수분
                light = values[1].strip()    # 조도 센서 값
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # 현재 시간 생성
                print(f"{timestamp}, Moisture: {moisture}%, Light: {light} lux")
                
                # 데이터 저장
                data.append({"Timestamp": timestamp, "Moisture (%)": moisture, "Light (lux)": light})

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # 데이터프레임 생성
    df = pd.DataFrame(data)

    # 데이터프레임을 CSV 파일로 저장
    df.to_csv("sensor_data.csv", index=False, encoding="utf-8")
    print("Data saved to sensor_data.csv")

    # 포트 닫기
    arduino.close()




