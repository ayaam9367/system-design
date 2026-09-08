# Latency vs Throughput
You should strive for max throughput with acceptable latency (not in the low latency world though)

Latency is the time required to perform some action or to produce some result.
Throughput is the number of such actions executed or results produced per unit of time. 
memory bandwidth - throughput of memory systems.

## Practical 
A designer is given the task to create hardware for a communications device that has the following characteristics:
 - Clock frequency: 100MHz
 - Time available to perform the computation: 1000ns
 - Throughput of the device: 640 Mbits / second
 - Word width of each output: 64 bits

Translation : 
clock freq = 100MHz = 10^5 Hz
1 clock period = 10^-5 s
time for compute = 1000 * 10^-6 = 10^-3 s = 10^-3/10^-5 clock periods = 10^2 clock periods
time of compute = 100 clock periods

Throughput = 64 * 10^4 bits / second = 10^4 words / second = 0.1 word per clock period

Higher throughput means the app is able to serve more data in lesser time - thus eventually serving more users.
This is important - https://youtu.be/FqR5vESuKe0?si=TmagsrA5mnisvi-W

# Little's Law /\ -> lambda
It applies when the server is operating at its capacity. 
L = /\ * W
L -> number of req currently being handler, /\ -> throughput, W -> avg latency
