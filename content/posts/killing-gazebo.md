---
type: post
title: "Killing Gazebo, Every Time"
date:   2017-10-09 10:05:55 -0600
categories: ROS
---

I do a lot of development with Gazebo.  A problem that has plagued me has been the fact that gazebo doesn't listen to the `Ctrl-C` command, and instead needs a `SIGTERM`.  This is super frustrating, because I have to wait something like 15 seconds before it finally dies, and when I'm trying to develop a controller, that time is valuable.

Here's my solution (it's a major hack).  All you do is modify the `nodeprocess.py` file (an internal ROS file) and reduce that 15 second wait before a `SIGTERM` to something a little shorter and less hair pulling.

Here's my new `/opt/ros/kinetic/lib/python2.7/dist-packages/roslaunch/nodeprocess.py:56`

```python
_TIMEOUT_SIGINT  = 1.0 #seconds (used to be 15.0)
_TIMEOUT_SIGTERM = 0.5 #seconds (used to be 2.0)

_counter = 0
def _next_counter():
    global _counter
    _counter += 1
    return _counter
```
