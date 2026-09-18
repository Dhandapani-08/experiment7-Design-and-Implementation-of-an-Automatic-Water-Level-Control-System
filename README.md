# Experiment: Design and Implementation of a Water Level Indicator Using STM32

## Aim

To design and implement a **Water Level Indicator using STM32 Nucleo-L031K6** that measures the water level using an analog input and displays the water level as **LOW, MEDIUM, or HIGH** on the Wokwi Serial Monitor.

---

## Components Required

- STM32 Nucleo-L031K6
- Potentiometer to simulate the water-level sensor
- Wokwi Simulator
- Virtual Serial Monitor
- Connecting wires
- Personal Computer/Laptop

---

## Software Required

- Wokwi Online Simulator
- STM32CubeIDE
- STM32 HAL Library
- C Programming Language

---

## Theory

A **Water Level Indicator** is used to monitor the amount of water present in a tank.

In this experiment, a **potentiometer** is used in Wokwi to simulate the output of a water-level sensor. The potentiometer produces an analog voltage between **0 V and 3.3 V** depending on its position.

The analog signal is connected to the **PA0 ADC input** of the STM32 Nucleo-L031K6.

The STM32 contains a **12-bit Analog-to-Digital Converter (ADC)**. Therefore, the analog input voltage is converted into a digital value between:

- **0** → Minimum water level
- **4095** → Maximum water level

The ADC value is converted into a percentage using:

`Water Level (%) = (ADC Value × 100) / 4095`

The water level is classified into three categories:

- **LOW** → 0–30%
- **MEDIUM** → 31–70%
- **HIGH** → 71–100%

The water-level information is displayed on the **Wokwi Serial Monitor** using UART communication.

---

## Principle of Operation

1. The potentiometer is used to simulate the water-level sensor.
2. The potentiometer produces an analog voltage between 0 V and 3.3 V.
3. The analog voltage is applied to the PA0 ADC input of the STM32.
4. The 12-bit ADC converts the analog voltage into a digital value from 0 to 4095.
5. The ADC value is converted into water-level percentage.
6. The percentage is compared with predefined threshold values.
7. The water level is classified as LOW, MEDIUM, or HIGH.
8. The ADC value, percentage, and status are transmitted through USART2.
9. The result is displayed on the Wokwi Serial Monitor.

---

## Pin Configuration

| Component | STM32 Pin | Function |
|---|---|---|
| Potentiometer SIG | PA0 | ADC Input |
| Potentiometer VCC | 3.3V | Power Supply |
| Potentiometer GND | GND | Ground |
| USART2 TX | PA2 | Serial Data Transmission |
| USART2 RX | PA15 | Serial Data Reception |

---

## STM32 Peripheral Configuration

| Parameter | Configuration |
|---|---|
| Microcontroller | STM32 Nucleo-L031K6 |
| ADC | ADC1 |
| ADC Input | PA0 |
| ADC Resolution | 12-bit |
| ADC Range | 0–4095 |
| ADC Mode | Single Conversion |
| UART | USART2 |
| Baud Rate | 115200 bps |
| Word Length | 8 Bits |
| Stop Bits | 1 |
| Parity | None |
| UART Mode | TX/RX |

---

## Block Diagram

~~~text
       Potentiometer
   (Water Level Sensor)
          |
          | Analog Signal
          v
       PA0 / ADC
          |
          v
+-----------------------+
| STM32 Nucleo-L031K6   |
|                       |
| ADC Conversion        |
|        |              |
|        v              |
| Water Level %         |
|        |              |
|        v              |
| LOW / MEDIUM / HIGH   |
+-----------+-----------+
            |
            | USART2
            v
      Serial Monitor
            |
            v
   Water Level Display
~~~

---

## Water Level Classification

| Water Level | Percentage | Approximate ADC Range | Indication |
|---|---:|---:|---|
| Low | 0–30% | 0–1229 | LOW |
| Medium | 31–70% | 1230–2866 | MEDIUM |
| High | 71–100% | 2867–4095 | HIGH |

---

## Algorithm

1. Start the program.
2. Initialize the STM32 HAL library.
3. Configure the system clock.
4. Configure PA0 as an analog input.
5. Initialize ADC1.
6. Configure ADC1 with 12-bit resolution.
7. Initialize USART2 for serial communication.
8. Configure USART2 with a baud rate of 115200 bps.
9. Start the ADC conversion.
10. Read the ADC value from the potentiometer.
11. Convert the ADC value into water-level percentage using:
    `Water Level (%) = (ADC Value × 100) / 4095`
12. Compare the calculated percentage with the predefined limits.
13. If the water level is less than or equal to 30%, display **LOW**.
14. If the water level is between 31% and 70%, display **MEDIUM**.
15. If the water level is greater than 70%, display **HIGH**.
16. Transmit the ADC value, water-level percentage, and status through USART2.
17. Display the result on the Serial Monitor.
18. Wait for one second.
19. Repeat the process continuously.
20. Stop.

---

## Program

```c
#include "main.h"
#include <stdio.h>
#include <string.h>

ADC_HandleTypeDef hadc1;
UART_HandleTypeDef huart2;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
static void MX_USART2_UART_Init(void);

int main(void)
{
    uint32_t adc_value;
    uint32_t water_level;
    char message[100];

    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_ADC1_Init();
    MX_USART2_UART_Init();

    while (1)
    {
        /* Start ADC Conversion */
        HAL_ADC_Start(&hadc1);

        /* Wait for ADC Conversion */
        HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);

        /* Read ADC Value */
        adc_value = HAL_ADC_GetValue(&hadc1);

        /* Convert ADC Value to Percentage */
        water_level = (adc_value * 100) / 4095;

        /* Determine Water Level */
        if (water_level <= 30)
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: LOW\r\n\r\n",
                    adc_value, water_level);
        }
        else if (water_level <= 70)
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: MEDIUM\r\n\r\n",
                    adc_value, water_level);
        }
        else
        {
            sprintf(message,
                    "ADC Value: %lu\r\nWater Level: %lu%%\r\nStatus: HIGH\r\n\r\n",
                    adc_value, water_level);
        }

        /* Send Result through UART */
        HAL_UART_Transmit(&huart2,
                          (uint8_t *)message,
                          strlen(message),
                          HAL_MAX_DELAY);

        /* Delay of 1 second */
        HAL_Delay(1000);
    }
}


/* ADC1 Initialization */
static void MX_ADC1_Init(void)
{
    ADC_ChannelConfTypeDef sConfig = {0};

    hadc1.Instance = ADC1;
    hadc1.Init.OversamplingMode = DISABLE;
    hadc1.Init.Resolution = ADC_RESOLUTION_12B;
    hadc1.Init.SamplingTime = ADC_SAMPLETIME_39CYCLES_5;
    hadc1.Init.ScanConvMode = ADC_SCAN_DIRECTION_FORWARD;
    hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
    hadc1.Init.ContinuousConvMode = DISABLE;
    hadc1.Init.DiscontinuousConvMode = DISABLE;
    hadc1.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
    hadc1.Init.EOCSelection = ADC_EOC_SINGLE_CONV;
    hadc1.Init.DMAContinuousRequests = DISABLE;
    hadc1.Init.Overrun = ADC_OVR_DATA_PRESERVED;

    HAL_ADC_Init(&hadc1);

    sConfig.Channel = ADC_CHANNEL_0;
    sConfig.Rank = ADC_RANK_CHANNEL_NUMBER;

    HAL_ADC_ConfigChannel(&hadc1, &sConfig);
}


/* USART2 Initialization */
static void MX_USART2_UART_Init(void)
{
    huart2.Instance = USART2;
    huart2.Init.BaudRate = 115200;
    huart2.Init.WordLength = UART_WORDLENGTH_8B;
    huart2.Init.StopBits = UART_STOPBITS_1;
    huart2.Init.Parity = UART_PARITY_NONE;
    huart2.Init.Mode = UART_MODE_TX_RX;
    huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
    huart2.Init.OverSampling = UART_OVERSAMPLING_16;

    HAL_UART_Init(&huart2);
}


/* GPIO Initialization */
static void MX_GPIO_Init(void)
{
    __HAL_RCC_GPIOA_CLK_ENABLE();
}


/* System Clock Configuration */
void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_MSI;
    RCC_OscInitStruct.MSIState = RCC_MSI_ON;
    RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_6;
    RCC_OscInitStruct.MSICalibrationValue = 0;

    HAL_RCC_OscConfig(&RCC_OscInitStruct);

    RCC_ClkInitStruct.ClockType =
        RCC_CLOCKTYPE_HCLK |
        RCC_CLOCKTYPE_SYSCLK |
        RCC_CLOCKTYPE_PCLK1;

    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_MSI;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;

    HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0);
}
```
Circuit Connections
Potentiometer
Potentiometer Pin	STM32 Connection
VCC	3.3V
GND	GND
SIG / Middle Pin	PA0
UART
UART Signal	STM32 Pin
TX	PA2
RX	PA15
GND	GND
Circuit Diagram
              Potentiometer
         (Water Level Sensor)
        +--------------------+
3.3V ---| VCC                |
GND  ---| GND                |
        | SIG                |
        +----+---------------+
             |
             |
            PA0
             |
             v
+---------------------------+
| STM32 Nucleo-L031K6       |
|                           |
| PA0  -> ADC Input         |
|                           |
| PA2  -> USART2 TX         |
| PA15 -> USART2 RX         |
+-------------+-------------+
              |
              | UART
              v
       Wokwi Serial Monitor
Procedure
Open the Wokwi Online Simulator.
Create a new STM32 project.
Select the STM32 Nucleo-L031K6 board.
Add a potentiometer to the circuit.
Connect the potentiometer VCC pin to 3.3V.
Connect the potentiometer GND pin to GND.
Connect the potentiometer SIG pin to PA0.
Configure PA0 as an ADC input.
Configure ADC1 for 12-bit resolution.
Configure USART2 for UART communication.
Set the baud rate to 115200 bps.
Enter the STM32 HAL program.
Compile the program.
Start the Wokwi simulation.
Open the Serial Monitor.
Rotate the potentiometer to simulate different water levels.
Observe the ADC value and water-level percentage.
Observe the corresponding LOW, MEDIUM, or HIGH status.
Verify the output using the manual calculations.
Formula

The STM32 ADC has a resolution of 12 bits.

Therefore:

Maximum ADC Value = 2^12 - 1

Maximum ADC Value = 4095

The water-level percentage is calculated as:

Water Level (%) = (ADC Value × 100) / 4095

Manual Calculations
Case 1: Low Water Level

Given:

ADC Value = 800

Calculation:

Water Level = (800 × 100) / 4095

Water Level = 19.53%

Approximately:

Water Level = 19%

Since:

19% <= 30%

Therefore:

Status = LOW

Case 2: Medium Water Level

Given:

ADC Value = 2200

Calculation:

Water Level = (2200 × 100) / 4095

Water Level = 53.72%

Approximately:

Water Level = 53%

Since:

31% <= 53% <= 70%

Therefore:

Status = MEDIUM

Case 3: High Water Level

Given:

ADC Value = 3500

Calculation:

Water Level = (3500 × 100) / 4095

Water Level = 85.47%

Approximately:

Water Level = 85%

Since:

85% > 70%

Therefore:

Status = HIGH

Expected Output
Low Water Level
ADC Value: 800
Water Level: 19%
Status: LOW
Medium Water Level
ADC Value: 2200
Water Level: 53%
Status: MEDIUM
High Water Level
ADC Value: 3500
Water Level: 85%
Status: HIGH
Output

The Wokwi Serial Monitor displays the water-level information continuously.

================================
     WATER LEVEL INDICATOR
================================

ADC Value: 800
Water Level: 19%
Status: LOW

ADC Value: 2200
Water Level: 53%
Status: MEDIUM

ADC Value: 3500
Water Level: 85%
Status: HIGH
Working

The potentiometer is used to simulate the water-level sensor. When the potentiometer is rotated, its output voltage changes between approximately 0 V and 3.3 V.

This voltage is applied to PA0, which is configured as an ADC input of the STM32.

The 12-bit ADC converts the analog voltage into a digital value between 0 and 4095.

The STM32 converts this ADC value into a percentage using:

Water Level (%) = (ADC Value × 100) / 4095

The calculated percentage is then compared with the predefined limits:

0–30% → LOW
31–70% → MEDIUM
71–100% → HIGH

The ADC value, percentage, and status are transmitted through USART2 and displayed on the Wokwi Serial Monitor.

Thus, rotating the potentiometer simulates an increase or decrease in the water level.

Result

Thus, the Water Level Indicator was successfully designed and implemented using the STM32 Nucleo-L031K6. The analog water-level signal was read through the ADC, converted into a percentage, classified as LOW, MEDIUM, or HIGH, and displayed successfully on the Wokwi Serial Monitor.
