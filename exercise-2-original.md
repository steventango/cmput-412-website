# Exercise 2 - ROS Development and Kinematics


## Learnings

`sudo nmap -sS -v -n -p 22 192.168.1.33/24 --host-timeout 1 --max-retries 1 --min-parallelism 256 --min-rtt-timeout 1 --max-rtt-timeout 1 --initial-rtt-timeout 1`

Write your code to store your robot's hostname in a variable

```py
hostname = os.environ.get('VEHICLE_NAME')
```

Using `rqt_image_view` to view the camera image in my customized topic /`csc22902/`
![rqt_image_view](./images/rqt_image_view.png)

Screenshot of code for Part One Question 2
![my_image_node.py.png](./images/my_image_node.py.png)

**What is the relation between your initial robot frame and world frame?**

**How do you transform between them?**

## Challenges

### dts

I found that the `dts devel build -f -H csc22902.local` command failed intermittently with the following error:
```bash
docker.errors.APIError: 500 Server Error for http://192.168.1.33:2375/v1.40/auth: Internal Server Error ("Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)")
```

I found that this docker command modified from [RH2 Unit B-5.4-E17](https://docs.duckietown.org/daffy/duckietown-robotics-development/out/creating_docker_containers.html) is equivalent

```bash
docker -H csc22902.local build -t duckietown/my-ros-program:latest-arm64v8 .
docker -H csc22902.local build -t duckietown/waddle:latest-arm64v8 .
```

Need to mount `/data` to the container:

```bash
dts devel run -H csc22902.local -v /data:/data
```

Local build and run:

```bash
docker build -t duckietown/waddle:latest-arm64v8 .
docker run -v /data:/data  duckietown/waddle:latest-arm64v8
```

SPEED: 0.5
[DEBUG] [1675990852.515729]: kW: [-0.09360043  0.32902804  1.27478849]
44 cm

SPEED 0.8
[DEBUG] [1675990981.749347]: kW: [0.27339301 0.3296331  1.09718378]
23 cm
[DEBUG] [1675991057.426686]: kW: [0.29633999 0.33234411 1.36359084]
30 cm

SPEED 0.2
[DEBUG] [1675991282.317105]: kW: [0.16983162 0.34530541 1.30438927]
15 cm

SPEED 1
[DEBUG] [1675991362.801904]: kW: [0.5535115  0.38623376 1.6299979 ]
10 cm
