# Embedded Firmware Assignment solution 

### Problem Statement 1
In a vehicle control architecture, there are multiple Electronic Control Units (ECUs) connected together through a CAN protocol. Each ECU is programmed to take care of specific functionality. MCB is one such ECU in the architecture that performs below tasks:

- Measure 8 Bigital Inputs from multiple switches / sensors in a vehicle.
- Perform defined actions based on the status of digital inputs.
- Communication with other ECU's thorugh CAN protocol.
As part of this assignment, you have to write an Interrupt Service Routine (ISR) in C language, for samplin g the Digital inputs. The ISR should perform the below functionality:
- Read the real-time status of digital input pins from a global variable **g_ReadDIpinSts**.Here, each bit in g_ReadDIpinSts represents the status of one digital input pin.
- Update the digiral input staus to a global variable **g_AppDIpinSts**, maintaining the bit position of digital inputs.
- Condition for status update in **g_AppDIpinSts** is, "The pin state of a particular digital input has to be consistent for 10 consecutive ISR calls". Here, ISR is triggered every 100mSec once.

### Expected Output:
- Code uploaded in GitHub (share the link via Internshala chat). Use below function prototype.

   

      int ISR_DIsampling(){
            			//Write your code here
            	}

### Evaluation Criteria:
- Logic used
- Coding Standards
- Error free code

### Problem Statement 1 Solution 

### Logic used:

- For each pin, the function compares the current status with the previous status.
- If the status is consistent, the corresponding counter is incremented.
- If the counter reaches 10, the state is considered stable and g_AppDIpinSts is updated accordingly.
- If the state changes before reaching 10, the counter is reset.

### Code:
#include <stdint.h> \
#define NUM_INPUTS 8  // Number of digital inputs \
// Global variables representing the status of digital input pins \
volatile uint8_t g_ReadDIpinSts = 0;     // Real-time status of digital inputs \
volatile uint8_t g_AppDIpinSts = 0;      // Application-level status of digital inputs 

// Counter to keep track of consistent state occurrences \
uint8_t g_DIpinConsistencyCounter[NUM_INPUTS] = {0}; 

int ISR_DIsampling() {
    static uint8_t previousStatus = 0;  // To store the previous state of pins

    // Iterate over each digital input
    for (uint8_t i = 0; i < NUM_INPUTS; i++) {
        // Extract the current state of the ith pin
        uint8_t currentPinStatus = (g_ReadDIpinSts >> i) & 0x01;
        uint8_t previousPinStatus = (previousStatus >> i) & 0x01;

        // Check if the current state matches the previous state
        if (currentPinStatus == previousPinStatus) {
            // Increment the consistency counter
            if (g_DIpinConsistencyCounter[i] < 10) {
                g_DIpinConsistencyCounter[i]++;
            }

            // Update g_AppDIpinSts if the state has been consistent for 10 ISR calls
            if (g_DIpinConsistencyCounter[i] == 10) {
                // Set or clear the bit in g_AppDIpinSts based on current pin status
                if (currentPinStatus == 1) {
                    g_AppDIpinSts |= (1 << i);  // Set the bit
                } else {
                    g_AppDIpinSts &= ~(1 << i); // Clear the bit
                }
            }
        } else {
            // Reset the consistency counter
            g_DIpinConsistencyCounter[i] = 0;
        }
    }

    // Update previousStatus with the current state
    previousStatus = g_ReadDIpinSts;

    return 0;
}




### Problem Statement 2 

A Real Time Operating System is often required when an embedded system performs various event driven functionalities and FreeRTOS is one of the most ubiquitous Real Time Operating Systems for embedded systems. This assignment involves you showcasing the knowhow (or the ability to gather said knowhow) of the basic intricacies of a RTOS stack.

#### Consider the following Tasks and its corresponding Task Handle:

|  Task Prototype| Task Handle |
|--|--|
|  void ExampleTask1(void *pV);| TaskHandle_t TaskHandle_1; |
|  void ExampleTask2(void *pV);|TaskHandle_t TaskHandle_2;  |

#### Consider the following Queue and its properties:
| Property| Value|
|--|--|
| QueueHandle| QueueHandle_t Queue1;|
| Size of Queue | 5 |
| Data Type| Data_t|

#### Definition of Data_t

    typdef struct{
	    uint8_t dataID;
	    int32_t DataValue;
	    } Data_t;
### What is to be done:

- Create Tasks **ExampleTask1** and **ExampleTask2** as per above prototypes and handles. Priority will be the same and at your descrition.
- Create Queue **Queue1** with the above properties mentioned.
- **ExampleTask1** sends data to **Queue1** at a rate of once every 500ms. The delay between each send needs to be exact. The members of the structure are populated by global variables **G_DataID;** and **G_DataValue;** Assume that these variables are updated elsewhere.
- **ExampleTask2** takes data from **Queue2** whenever data is available, and applies the following logic to the data gathered:

| Condition | Action|
|--|--|
| if(dataID==0) |Delete ExampleTask2  |
| if(dataID==1) |Allow the processing of DataValue Member  |
| if(DataValue==0) |Increase the Priority of ExampleTask2 by 2 from the value given to it at creation  |
| if(DataValue==1) |Decrease the Priority of ExampleTask2 **if** previously increased |
| if(DataValue==2) |Delete ExampleTask2 |

At every evaluation, add a print statement to print out **dataID** and **DataValue**.


### Expected Output:
- Code uploaded in GitHub, same link for both problem statements, any other form of submission will not be accepted (google drive, direct file share, etc).(share the link using the submit assignment feature in Internshala).
### Evaluation Criteria:
- Logic used
- Coding Standards
- Error free code


### Problem Statement 2 Solution 

### Logic Uesd: 
### ExampleTask1:

- Function: This task is responsible for sending data to a queue (Queue1). It gathers data from global variables G_DataID and G_DataValue and sends this data structure (Data_t) to the queue at regular intervals of 500 milliseconds.
- Logic: It repeatedly creates a Data_t structure with the values of G_DataID and G_DataValue, then uses xQueueSend to place this data in the queue. A delay (vTaskDelay) ensures that this task waits 500 milliseconds before the next send, ensuring a consistent interval between data sends.

### ExampleTask2:

- Function: This task continuously waits to receive data from the queue (Queue1). Upon receiving data, it processes the data based on the value of dataID and DataValue.
- Logic:
When dataID is received:
If dataID == 0: The task deletes itself (vTaskDelete).
If dataID == 1: It allows further processing of DataValue.
Based on DataValue:
If DataValue == 0: It increases the task's priority by 2.
If DataValue == 1: It resets the task's priority to its original value.
If DataValue == 2: The task deletes itself (vTaskDelete).



### Includes and Definitions
#include "FreeRTOS.h"\
#include "task.h"\
#include "queue.h"\
#include <stdio.h>

// Data structure definition\
typedef struct {
    uint8_t dataID;
    int32_t DataValue;
} Data_t;

// Task handles\
TaskHandle_t TaskHandle_1;\
TaskHandle_t TaskHandle_2;

// Queue handle
QueueHandle_t Queue1;

// Queue size and other parameters
#define QUEUE_SIZE 5
#define QUEUE_WAIT_TICKS 100 / portTICK_PERIOD_MS

// Global variables
volatile uint8_t G_DataID;
volatile int32_t G_DataValue;

// Function prototypes
void ExampleTask1(void *pV);
void ExampleTask2(void *pV);

### ExampleTask1: Sends data to Queue1 every 500ms.
void ExampleTask1(void *pV) {
    Data_t dataToSend;\
    const TickType_t xDelay = pdMS_TO_TICKS(500); // 500 ms delay

    while (1) {
        dataToSend.dataID = G_DataID;
        dataToSend.DataValue = G_DataValue;

        if (xQueueSend(Queue1, &dataToSend, QUEUE_WAIT_TICKS) != pdPASS) {
            printf("Queue is full. Data not sent.\n");
        } else {
            printf("Sent data: ID = %d, Value = %d\n", dataToSend.dataID, dataToSend.DataValue);
        }

        vTaskDelay(xDelay); // Delay for 500 ms
    }
}

### ExampleTask2:  Receives data from Queue1 and processes it according to the conditions.

void ExampleTask2(void *pV) {
    Data_t receivedData;\
    UBaseType_t initialPriority = uxTaskPriorityGet(TaskHandle_2);

    while (1) {
        if (xQueueReceive(Queue1, &receivedData, portMAX_DELAY) == pdPASS) {
            printf("Received data: ID = %d, Value = %d\n", receivedData.dataID, receivedData.DataValue);

            switch (receivedData.dataID) {
                case 0:
                    vTaskDelete(TaskHandle_2);
                    break;
                case 1:
                    if (receivedData.DataValue == 0) {
                        vTaskPrioritySet(TaskHandle_2, initialPriority + 2);
                    } else if (receivedData.DataValue == 1) {
                        vTaskPrioritySet(TaskHandle_2, initialPriority);
                    } else if (receivedData.DataValue == 2) {
                        vTaskDelete(TaskHandle_2);
                    }
                    break;
                default:
                    printf("Unknown dataID: %d\n", receivedData.dataID);
                    break;
            }
        }
    }
}

### Main Function and Setup

int main(void){
    // Create Queue
    Queue1 = xQueueCreate(QUEUE_SIZE, sizeof(Data_t));\
    if (Queue1 == NULL) {
        printf("Queue creation failed!\n");\
        return 1;
    }

    // Create Tasks
    BaseType_t xReturned;
    xReturned = xTaskCreate(ExampleTask1, "Task1", configMINIMAL_STACK_SIZE, NULL, 1, &TaskHandle_1);
    if (xReturned != pdPASS) {
        printf("Task1 creation failed!\n");
        return 1;
    }

    xReturned = xTaskCreate(ExampleTask2, "Task2", configMINIMAL_STACK_SIZE, NULL, 1, &TaskHandle_2);
    if (xReturned != pdPASS) {
        printf("Task2 creation failed!\n");
        return 1;
    }

    // Start Scheduler
    vTaskStartScheduler();

    // Will never reach here
    for (;;);

    return 0;
}
