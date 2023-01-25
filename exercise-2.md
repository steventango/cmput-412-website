# Exercise 2 - ROS Development and Kinematics


## Learnings

### [ROS Concepts](http://wiki.ros.org/ROS/Concepts)

* Node: named modular computation process.
* Topic: named message bus that routes messages that nodes can publish and subscribe to.
* Service: named pair of request message from a client node and reply message from the providing node.
* Message: typed data structure used for internode communication.
* Bag: message data format

`sudo nmap -sS -v -n -p 22 192.168.1.33/24 --host-timeout 1 --max-retries 1 --min-parallelism 256 --min-rtt-timeout 1 --max-rtt-timeout 1 --initial-rtt-timeout 1`


Use this
```
docker -H csc22902.local build -t duckietown/my-ros-program .
```
not
```
dts devel build -f -H csc22902.local
```

Use this
```
docker -H csc22902.local run duckietown/my-ros-program .
```
not
```
dts devel run -H csc22902.local
```
