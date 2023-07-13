# RTOS-Communicating-Tasks
This project involves several sender tasks (the ability to have to different priorities is available) periodically send messages to a fixed size queue containing the current time in ticks.
If the queue is full, the sending operation fails and increments a counter for blocked messages. 
Successful sending increments a counter for transmitted messages.
The receiver task periodically reads messages from the queue, increments a counter for received messages, and sleeps if no message is available.
Timers control task sleep/wake behaviour as each task has a corresponding timer that when fires it releases a dedicated semaphore upon which the task is blocked, so each task has a dedicated semaphore for blocking/unblocking. 
# Sender Task
A single sender task function is employed for multiple sender tasks by passing the sender task ID as a parameter. 
The function executes specific code within the task based on the ID, utilizing dedicated semaphores, counters, and timers stored in arrays with indexes similar to the task ID. 
And the following is the function pseudocode: 
```
function senderTask(senderTaskID):
ID = convertToInteger(senderTaskID)
    sendMsg = createCharArray(MAX_STRING_LENGTH)
    loop forever:
        If takeSemaphore(xSemaphoreSender[ID], portMAX_DELAY) is successful:
            formattedMsg = formatMessage("Time is %lu", getTickCount())
            If sendToQueue(gQueueHandle, formattedMsg, 0) is successful:
                increment(gNumOfTransmittedMessagesOfSender[ID])
            else:
                increment(gNumOfBlockedMessagesOfSender[ID])
```
# Sending time 
 It is to be noted that the timer period for each sender task is randomized from uniform distribution upon each sending between minimum and maximum bounds that vary from one iteration to another throughout the number of iterations.
 The following function is used to ensure a uniform random distribution of the randomized value:
 ```
static uint32_t generateRandomTsender (uint32_t minBound, uint32_t maxBound) {
	uint32_t range = maxBound - minBound + 1;
	uint32_t scaledRandom = (uint32_t)(((double)rand() / (double)RAND_MAX) * range);
	return scaledRandom + minBound;
}
```
# Enhancing adaptability
In order to enhance the code's adaptability, the initial section defines various parameters. 
These parameters include the time at which messages are received, the total number of senders,
the number of senders categorized as low priority (with the remaining senders considered higher priority), the number of iterations or periods for which the reciever will receive the number of revieved messages ,
the number of received messages per iteration, and the size of the queue.
```
#define Treceiver                                      100
#define NUM_OF_LOW_PRIORITY_SENDERS                    2
#define NUM_OF_RUNS                                    6
#define NUM_SENDERS                                    3
#define MAX_NUM_OF_RECEIVED_MESSAGES_PER_RUN           1000
#define QUEUE_SIZE                                     3
```
# An illustrative example 
 This [report](https://drive.google.com/file/d/1Ef3Onxk2SDorHa-o_zPlPkgDZ8OraW1C/view) include an example with three sender tasks (two with equal priority, one with higher priority).

