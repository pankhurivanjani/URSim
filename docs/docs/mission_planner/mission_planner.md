---
layout: default
title: Mission Planner

nav_order: 4

has_children: false
permalink: docs/mission_planner
---

# **Mission Planner**

The mission planner used in the simulation system is a high level module based on the concept of finite state machines. Mission planning is the key to achieve intelligent behaviour in an autonomous system. It consists of an apparatus for top-down decomposition of states where each state represents a task that the robot needs to perform.<br>Real-world complex behaviors can be developed using hierarchical and concurrent state machines to perform tasks like motion in different degrees of freedom, object detection and obstacle avoidance.

The mission planner based on the above concept has been implemented using the SMACH package provided by ROS. The mission planner contains low level states for motion of the vehicle along or about each possible degree of freedom - Roll, Pitch, Yaw, Surge, Sway and Heave. Using combinations of these low level states, higher level sub-missions can be developed. The state hierarchy diagram depicts the tree structure for the developed mission.

![State Hierarchy](abstract.png)

Mission planner works internally as follows:

1. 	**mission_planner.py** 
	* *main()* calls *listener()*. 
	* *listener()* creates a new node called *mission_planner*. The node subscribes to a topic called */zsys* and calls the *callback()*.
	* The *callback()* calls *main()* where an object called *sm* of *StateMachine* is created with 3 outcomes - ['mission_complete', 'mission_failed', 'aborted']
	* *sm* performs 3 tasks - sink to a certain depth, align to a certain heading, move forward for some seconds.<br> 
	`Sink (sm, 'SINK1', 515, 'HEADING1')`<br>
    `Heading(sm, 'HEADING1', 75,'FORWARD1')`<br>
    `Forward(sm, 'FORWARD1', 6, 'FORWARD2')`<br>
	`Forward(sm, 'FORWARD2', 12, 'mission_complete')`<br>

2.  **Sink.py & Depth.py**<br>
	`Sink (<state machine object>, <name of state>, <pressure>, <next state>)`<br>
	* 	In *Sink.py* the *Sink* class's constructor is called which sets the initial pressure. 
	* 	A substate called *sm_sub* is created with outcomes as ['depth_success', 'aborted'].
	* 	*sm_sub* creates an object called *depthTask* of the class *Depth*.<br>
	`depthTask = Depth(INITIAL_PRESSURE, 'depth_success')`<br>The *Depth.py*'s constructor sets the values of pressure and task variables.<br>
	`depthTask.addDepthAction(smach_StateMachine)` calls the *addDepthAction(self, sm)* method of class *Depth* where a new *SimpleActionState* *DEPTH* is added to *sm* and *depthCallback()* is called, transitions being *'succeeded':self.TASK, 'preempted':'DEPTH', 'aborted':'aborted'*. 
	* 	In *depthCallback()*, the *depth_setpoint* is set to *PRESSURE*'s  value.'
	*	From *mission_planner.py*, *execute()* is called which creates a SimpleActionClient and sends the PRESSURE as the goal and returns 'DepthReached'/'aborted'.

2.  **Heading.py**<br>
	`Heading (<state machine object>, <name of state>, <heading>, <next state>)`<br>
	* 	In *Heading.py* the *Heading* class's constructor is called which sets the initial heading. 
	* A new *SimpleActionState* *heading1* is added to *sm* and *headingCallback()* is called.. 
	* 	In *headingCallback()*, the *heading_setpoint* is set to *HEADING*'s  value.'
	*	From *mission_planner.py*, *execute()* is called which creates a SimpleActionClient and sends the HEADING as the goal and returns 'HeadingReached'/'aborted'.

3. 	**Forward.py**<br>
	`Forward (<state machine object>, <name of state>, <time>, <next state>)`<br>
	*	Creates a SimpleActionState called *surgeServer*, calls *goalCallback()* to move the Underwater Vehicle forward for the time sent from *mission_planner.py*.
	


