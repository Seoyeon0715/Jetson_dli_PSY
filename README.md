# 아두이노 우노에서 실행되는 코드 

```bash
#include <DHT.h>

#define DHTPIN 2     // DHT11 센서가 연결된 핀
#define DHTTYPE DHT11   // DHT11 타입의 센서를 사용

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600); 
  dht.begin();
}

void loop() {
  float humidity = dht.readHumidity();  // 습도값 읽기
  float temperature = dht.readTemperature();  // 섭씨 온도 읽기

  // 센서가 제대로 읽히지 않을 경우
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" *C");

  // 상태 판단 (온도와 습도를 기준으로)
  if (humidity < 30) {
    Serial.println("Warning: Low Humidity! Consider humidifying.");
  } else if (humidity > 70) {
    Serial.println("Warning: High Humidity! Consider dehumidifying.");
  } else {
    Serial.println("Humidity is in the optimal range.");
  }

  if (temperature < 18) {
    Serial.println("Temperature is too low! Consider heating.");
  } else if (temperature > 30) {
    Serial.println("Temperature is too high! Consider cooling.");
  } else {
    Serial.println("Temperature is in the optimal range.");
  }

  delay(2000); // 2초 대기
}              
```

# 설명 
라이브러리 및 핀 설정 : DHT.h디지털을 사용하여 DHT11 센서를 제어하며, 데이터 핀은 2번 핀에 연결됩니다.
setup()함수 : 직렬 접속을 통해 센서를 시작합니다.
loop(): 인증 된 센서의 테스트( humidity)와 온도( temperature) 값을 반환하고, 값을 반환합니다.
경고 메시지 출력 : 온도가 측정된 값이 범위를 벗어나면 경고 메시지를 출력합니다.

-------------------------------------------------------------------------------------------
#아두이노에 연결된 접속수분 센서에서 데이터 수집하기 & 아두이노 데이터 수집 및 저장

```bash
import serial
import csv
import time

# 아두이노와 연결된 시리얼 포트 설정
SERIAL_PORT = '/dev/ttyUSB0'  # 젯슨 나노에 연결된 아두이노의 포트 (예: '/dev/ttyUSB0')
BAUD_RATE = 9600  # 아두이노의 시리얼 통신 속도

----------------------------참고----------------------------------
# 가상환경 활성화 및 패키지 설치 지침
print("### 1. 터미널에서 가상환경을 활성화하는 명령어 실행")
print("source myenv/bin/activate")
print("# 활성화되면 터미널 프롬프트에 (myenv2)가 추가로 표시됩니다")
print("(myenv2) dcrc@jetson-nano:~$")

print("\n### 2. 필요한 패키지 확인")
print("# 활성화된 가상환경에서 필요한 Python 패키지가 설치되었는지 확인합니다:")
print("pip list")
print("# 설치되어야 하는 패키지 : jupyter, pandas, pyserial, matplotlib, openpyxl")

print("\n### 만일 누락된 패키지가 있음?")
print("sudo apt update")
print("sudo apt install python3-pip")
print("pip3 install jupyter pandas pyserial")
print("pip install jupyter pandas pyserial matplotlib openpyxl")

print("\n# 4. Jupyter Notebook 실행한다")
print("jupyter notebook")
print("터미널에 표시된 URL(예: http://localhost:8888/)을 브라우저에 입력하여 Jupyter Notebook에 접속합니다.")
------------------------------------------------------

# CSV 파일 설정
CSV_FILE = 'dht11_data.csv'

# 아두이노 데이터를 살펴보고 저장하는 코드를 작성했습니다. 아래는 Jupyter Notebook 셀에 입력할 Python 코드입니다:
print("\n코드: 아두이노 데이터 수집 및 저장")

# 시리얼 포트 연결
try:
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print(f"Connected to {SERIAL_PORT}")
except serial.SerialException:
    print(f"Failed to connect on {SERIAL_PORT}")
    exit()

# CSV 파일 작성 및 헤더 추가
with open(CSV_FILE, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(['Timestamp', 'Temperature (C)', 'Humidity (%)'])

print("Starting data collection...")

# 데이터 수집 루프
try:
    while True:
        if ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').strip()
            
            # 아두이노로부터 데이터 형식: "Temperature: xx.x *C, Humidity: xx.x %"
            if line.startswith("Temperature"):
                try:
                    temp_part, hum_part = line.split(', ')
                    temperature = temp_part.split(': ')[1].replace(' *C', '')
                    humidity = hum_part.split(': ')[1].replace(' %', '')
                    
                    # 현재 시간 가져오기
                    timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
                    
                    # 데이터 출력 및 CSV 저장
                    print(f"{timestamp} - Temperature: {temperature} *C, Humidity: {humidity} %")
                    with open(CSV_FILE, mode='a', newline='') as file:
                        writer = csv.writer(file)
                        writer.writerow([timestamp, temperature, humidity])
                except ValueError:
                    print("Failed to parse the data from Arduino.")

        # 1초 간격으로 데이터 수집
        time.sleep(1)

except KeyboardInterrupt:
    print("\nData collection stopped.")
    ser.close()
```

# 설명
"
아래는 아두이노에서 수집한 온도와 습도 데이터를 젯슨 나노에 연결된 Python 프로그램으로 수집하고 이를 CSV 파일로 저장하는 코드입니다.
이 코드에서는 Python에서 시리얼 통신을 사용하여 아두이노의 데이터를 읽고, 이를 저장하는 방식으로 구현했습니다.
"
# 코드 설명:
시리얼 통신 설정: 아두이노와의 시리얼 통신을 설정합니다. /dev/ttyUSB0를 사용하여 아두이노가 젯슨 나노와 연결되어 있고, 보드레이트(BAUD_RATE)는 아두이노와 일치하는 9600으로 설정합니다.
CSV 파일 생성: 데이터 수집을 시작하기 전에 CSV 파일을 만들고, 헤더를 추가합니다.
데이터 수집 루프: 시리얼 포트에서 데이터를 읽고, 데이터 형식이 적절한지 확인한 후 온도와 습도 데이터를 추출하여 파일에 저장합니다.
키보드 인터럽트 처리: Ctrl+C를 통해 수집을 중단할 수 있으며, 이 경우 시리얼 포트를 안전하게 닫습니다.

터미널에서 실행 방법:
이 코드를 실행하기 위해서는 Python 환경이 필요하며, pyserial 라이브러리가 설치되어 있어야 합니다. 
다음 명령어를 터미널에서 실행하여 설치할 수 있습니다:

pip install pyserial

```bash 
# 아두이노 코드
아두이노에서 접속수분 센서 값을 읽고 시리얼 모니터에 출력하는 코드입니다.

// 접속수분 센서가 연결된 아날로그 핀
#define SOIL_MOISTURE_PIN A0

void setup() {
  Serial.begin(9600); // 시리얼 통신 시작
  pinMode(SOIL_MOISTURE_PIN, INPUT); // 접속수분 센서 핀을 입력으로 설정
}

void loop() {
  // 아날로그 값을 읽음 (0 ~ 1023)
  int sensorValue = analogRead(SOIL_MOISTURE_PIN);

  // 값을 습도 퍼센트로 변환 (0 ~ 100%)
  float moisturePercent = map(sensorValue, 1023, 0, 0, 100);

  // 값 출력
  Serial.print("Soil Moisture: ");
  Serial.print(moisturePercent);
  Serial.println(" %");

  delay(1000); // 1초 대기
}
```
-----------------------------------
파이썬 코드
아두이노에서 데이터를 수신하여 CSV 파일에 저장하는 파이썬 코드입니다.
```bash 
import serial
import csv
import time

# 아두이노와 연결된 시리얼 포트 설정
SERIAL_PORT = '/dev/ttyUSB0'  # 아두이노와 연결된 포트를 입력
BAUD_RATE = 9600  # 아두이노 시리얼 통신 속도

# CSV 파일 설정
CSV_FILE = 'soil_moisture_data.csv'

# 시리얼 포트 연결
try:
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print(f"Connected to {SERIAL_PORT}")
except serial.SerialException:
    print(f"Failed to connect on {SERIAL_PORT}")
    exit()

# CSV 파일 작성 및 헤더 추가
with open(CSV_FILE, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(['Timestamp', 'Soil Moisture (%)'])

print("Starting data collection...")

# 데이터 수집 루프
try:
    while True:
        if ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').strip()

            # 아두이노로부터 데이터 형식: "Soil Moisture: xx.x %"
            if line.startswith("Soil Moisture"):
                try:
                    moisture_percent = line.split(': ')[1].replace(' %', '')

                    # 현재 시간 가져오기
                    timestamp = time.strftime('%Y-%m-%d %H:%M:%S')

                    # 데이터 출력 및 CSV 저장
                    print(f"{timestamp} - Soil Moisture: {moisture_percent} %")
                    with open(CSV_FILE, mode='a', newline='') as file:
                        writer = csv.writer(file)
                        writer.writerow([timestamp, moisture_percent])
                except ValueError:
                    print("Failed to parse the data from Arduino.")

        # 1초 간격으로 데이터 수집
        time.sleep(1)

except KeyboardInterrupt:
    print("\nData collection stopped.")
    ser.close()
``` 
---------------------------------------------
# 사용 방법

1) 아두이노에 접속수분 센서를 연결합니다. 센서의 출력 핀은 아날로그 핀(A0)에 연결하세요.
2) 아두이노 코드를 업로드합니다.
3) 파이썬 환경에서 필요한 라이브러리를 설치하세요

pip install pyserial
```

#참고 사항
센서가 출력하는 값은 환경에 따라 달라질 수 있으므로, 정확한 습도 퍼센트를 얻으려면 실험적으로 map() 함수의 범위를 조정해야 할 수 있습니다.
데이터 수집 중 키보드 인터럽트(Ctrl + C)로 종료할 수 있습니다.

#
사용 방법
Jupyter Notebook 환경에서 실행
해당 코드를 Jupyter Notebook 셀에 복사하여 실행합니다.
시리얼 포트 설정
SERIAL_PORT에 실제 아두이노가 연결된 포트를 입력합니다. 예: /dev/ttyUSB0 (Linux/Jetson Nano) 또는 COM3 (Windows).
데이터 수집 중단
데이터 수집은 무한 루프를 통해 진행됩니다. Ctrl + C를 눌러 중단합니다.
CSV 파일 확인
실행 디렉토리에 soil_moisture_data.csv 파일이 생성되며, 시간별 센서 데이터가 저장됩니다.

---------------------------------------
# 아두이노 코드 업로드 확인
```bash
import serial
import time

def check_arduino_connection(port, baud_rate, test_duration=10):
    """
    아두이노가 올바르게 동작하는지 확인하는 함수.
    
    Parameters:
        port (str): 아두이노가 연결된 시리얼 포트 (예: '/dev/ttyUSB0', 'COM3').
        baud_rate (int): 시리얼 통신 속도.
        test_duration (int): 테스트 실행 시간 (초).
        
    Returns:
        bool: True if the connection and data are valid, False otherwise.
    """
    try:
        # 시리얼 포트 연결
        ser = serial.Serial(port, baud_rate, timeout=1)
        print(f"Connected to {port}")
        start_time = time.time()
        valid_data_received = False

        # 테스트 시간 동안 데이터 확인
        print(f"Checking Arduino output for {test_duration} seconds...")
        while time.time() - start_time < test_duration:
            if ser.in_waiting > 0:
                line = ser.readline().decode('utf-8').strip()
                print(f"Received: {line}")
                
                # 예상 데이터 형식 확인 (예: "Temperature: xx.x *C, Humidity: xx.x %")
                if line.startswith("Temperature") and "Humidity" in line:
                    valid_data_received = True
                    break

            time.sleep(0.1)  # 약간의 대기 시간 추가

        ser.close()

        if valid_data_received:
            print("Arduino is functioning correctly!")
            return True
        else:
            print("No valid data received. Check Arduino code and connection.")
            return False

    except serial.SerialException as e:
        print(f"Error: {e}")
        return False

# 메인 실행
if __name__ == "__main__":
    SERIAL_PORT = '/dev/ttyUSB0'  # 아두이노 연결 포트
    BAUD_RATE = 9600  # 아두이노 코드에서 설정한 시리얼 통신 속도

    # 연결 확인
    is_arduino_working = check_arduino_connection(SERIAL_PORT, BAUD_RATE)
    if is_arduino_working:
        print("Arduino is ready for data collection.")
    else:
        print("Fix the issues and try again.")
```
-----------------------------
# 함수 설명
connect_serial(port, baud_rate)

시리얼 포트를 연결하고, 연결 객체를 반환합니다.
연결 실패 시 None을 반환하여 예외 처리 가능.
collect_data(ser, csv_file, duration)

아두이노에서 데이터를 읽고, 데이터를 CSV 파일에 저장합니다.
데이터는 시간(Timestamp), 온도(Temperature), 습도(Humidity)로 저장됩니다.
수집 시간(duration)은 초 단위로 설정 가능.
main()

프로그램의 주요 흐름을 정의.
시리얼 포트를 연결하고, 데이터를 수집한 뒤 종료.
```bash 
# use_functions = [
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
    },
    {
        "type": "function",
        "function": {
            "name": "temperature_humidity_sensor_info",
            "description": "Retrieves temperature and humidity data in real-time from a DHT11 sensor.",
            "parameters": {
                "type": "object",
                "properties": {
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the DHT11 sensor is connected. Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the sensor. Default is 9600.",
                        "nullable": True
                    },
                    "duration": {
                        "type": "integer",
                        "description": "The duration in seconds for data collection. Default is 60 seconds.",
                        "nullable": True
                    }
                },
                "required": []
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "check_arduino_status",
            "description": "Verifies whether the Arduino is properly connected and sending expected data.",
            "parameters": {
                "type": "object",
                "properties": {
                    "port": {
                        "type": "string",
                        "description": "The serial port to which the Arduino is connected. Default is '/dev/ttyUSB0'.",
                        "nullable": True
                    },
                    "baudrate": {
                        "type": "integer",
                        "description": "The baud rate for serial communication with the Arduino. Default is 9600.",
                        "nullable": True
                    },
                    "test_duration": {
                        "type": "integer",
                        "description": "The duration in seconds for the Arduino status test. Default is 10 seconds.",
                        "nullable": True
                    }
                },
                "required": []
            }
        }
    }
]
```

# 각 함수의 설명
moisture_sensor_info

mode: 실시간(real-time) 센서 데이터를 읽거나, 파일(file)에서 데이터를 읽습니다.
file_path: 파일 모드에서 필요한 Excel 파일 경로.
port: 센서가 연결된 시리얼 포트. 실시간 모드에서만 필요합니다.
baudrate: 센서와의 통신 속도. 기본값은 9600.
temperature_humidity_sensor_info

온습도 데이터를 실시간으로 수집합니다.
port: 센서가 연결된 포트.
baudrate: 센서와의 통신 속도.
duration: 데이터 수집 지속 시간.
check_arduino_status
아두이노가 정상적으로 작동하는지 확인합니다.
port: 아두이노가 연결된 포트.
baudrate: 아두이노와의 통신 속도.
test_duration: 테스트 실행 시간



아두이노 코드: 조도 센서 데이터 전송
Jetson Nano에서 Python 코드 작성
Function Calling Definition
use_functions JSON 정의


https://github.com/adafruit/DHT-sensor-library

https://github.com/adafruit/Adafruit_Sensor

arduino-cli lib install "DHT sensor library"
arduino-cli lib install "Adafruit Unified Sensor"
arduino-cli compile --fqbn arduino:avr:uno YourSketch.ino
arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:uno YourSketch.ino




## 0. Python 라이브러리 설치
Jetson Nano에서 DHT11 센서를 사용하려면 Adafruit_DHT 라이브러리가 필요합니다. 터미널에서 다음 명령을 실행하세요:
**pip install Adafruit_DHT**


## 1. 아두이노 코드: 온습도 센서(DHT11) 데이터 전송

```bash
#include "DHT.h"

#define DHTPIN 2     // DHT 센서가 연결된 핀
#define DHTTYPE DHT11  // DHT11 센서 유형

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);  // 직렬 통신 시작
  dht.begin();         // DHT 센서 초기화
}

void loop() {
  float humidity = dht.readHumidity();    // 습도 읽기
  float temperature = dht.readTemperature();  // 온도 읽기

  // 센서 읽기 오류 처리
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // 데이터를 직렬 통신으로 전송
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print("°C, ");
  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println("%");

  delay(2000);  // 2초 대기
}
```

## 2. Jetson Nano에서 Python 코드 작성

```bash
import serial
import time
import pandas as pd

def dht11_sensor_info(mode='real-time', file_path=None, port='/dev/ttyUSB0', baudrate=9600):
    """
    DHT11 센서 데이터를 처리하는 함수.

    Args:
        mode (str): 'real-time'이면 센서에서 데이터를 읽고, 'file'이면 엑셀 파일에서 데이터를 읽음.
        file_path (str): 파일 경로 (mode='file'일 때 필수).
        port (str): 아두이노가 연결된 직렬 포트.
        baudrate (int): 직렬 통신 속도.

    Returns:
        str: 온습도 데이터 정보 또는 오류 메시지.
    """
    if mode == 'real-time':
        try:
            # 아두이노와 직렬 통신 설정
            arduino = serial.Serial(port, baudrate, timeout=1)
            time.sleep(2)  # 아두이노 초기화 대기
            
            if arduino.in_waiting > 0:
                line = arduino.readline().decode('utf-8').strip()  # 데이터 읽기 및 디코딩
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")    # 현재 시간 기록
                arduino.close()
                return str({'timestamp': timestamp, 'data': line})
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
                'temperature': f"{latest_entry['Temperature']}°C",
                'humidity': f"{latest_entry['Humidity']}%"
            })
        except Exception as e:
            return str({'error': str(e)})

    else:
        return str({'error': 'Invalid mode. Use "real-time" or "file".'})
```

## 3. Function Calling Definition

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
