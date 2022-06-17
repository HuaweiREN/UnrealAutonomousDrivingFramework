# UnrealAutonomousDrivingFramework

The framework is to provide a straight forward and easy-to-use autonomous driving vehicle in unreal engine. The framework is based on UE chaos vehicle model, but not strictly limit to it, which means developers can easily replace the vehicle with their owns. The ADAS&APA control system and the vehicle model are de-coupled via several interfaces (throttle input, brake input, and steer input).

Please note that the framework is developed under Unreal Engine ver.4.26.2. In theory, the framework should be compatible with any version later than 4.xx. Things might be a little tricky when coming to Unreal Engine 5 as the CHAOS vehicle model has been modified a lot in source code. There are some workaround provided in this framework. Once you are trapped into this condition, please refer to the section *Make framework alive in Unreal Engine 5*.

Should you have any questions or other perference related to the framework itself, please feel free to contact me with the email address below.

COPYRIGHT: Huawei REN  
Contact: Ravidren@126.com

## What is Autonomous Driving (AD)?
Over the past few years, "how to make the vehicle drive with intelligence", becomes a hot topic among scientists and engineers. This is no longer a brainstorm concept, but a truely technology supported by in-market products. Tesla with their FSD,  GM with Super Cruise, Xpeng with NOP, are suddenly coming together onto audience choice list. Today, you may still use words like "beautiful design, large trunk space" to describe your preferences to a car, but we believe in future, "good autonomous driving ability" will also become an important attribute to "steal" money out of your wallet, of course, willingly.

### Level of AD
Fully Autonomous Driving is not a one-step feature. The exploration process is long and the target is difficult to catch. To compromise, ADAS (Advanced Driving Assistance System) concept is introduced.

From very basic ADAS to fully AD, researchers and vehicle engineers decided to use 5 levels to describe the steps human beings climbing to autonomous driving ear, which are:
- Level 1: vehicle can provide some alerts / reminders to drive for mitigating the 0
- Level 2: vehicle can provde some controls to the actuators with or without the confirm from vehicle driver. These automatic commands can bring some comfortness to the driver and occupants, and can mitigate unexpected collisions. However, L2 assistance is not 100 percent reliable to help road users to avoid any unexpected conditions. Drivers still should always to pay their attention on the road.
- Level 3: Based on L2 assistance ability. Drivers can now move their attentions to somewhere else, shortly. Long time abstraction is still not allowed even L3 assistance is provided. 
		- Huawei opinion: L3 assistance concept is a bit vague. There is no strict boundary between L2 and L3. OEMs sometime claim their new vehicle has reached L3 ADAS, but only limited to some special conditions.
- Level 4: Within a specific region, vehicle is able to fully drive safe automatically. Driver's attention on road is no longer needed.
- Level 5: Within all roads, vehicle is able to fully drive safe automatically. In other words, you can do whatever you want (under legal restriction) inside a car and there is no need to concern any safety problems.

 Here it comes with a question: what AD framework achieves? Is it able to achieve Level 5 assistance? The answer is NO. AD framework is still an infrastructure probing to L2+. Don't be upset. AD framework is providing you infinite potential solutions where you can define and explore on top of the infrastructures. At meantime, AD framework developers are also enhancing the performance and ability to fine-tune the published and under developing features.

## Classification of ADAS
There are multiple methodology to classify ADAS features. What we most favorite is to use the vehicle speed & control direction to classify.

Firstly, we can use if vehicle speed is high or low (thresold about 10 kph) to distinguish if a vehicle is under control of driving, or parking.
- Vehicle speed > 10 kph, vehicle is under driving control -> high speed ADAS features, aka. ADAS
- Vehicle speed < 10 kph, vehicle is under parking control -> low speed ADAS features, aka APA (Advanced Parking Assist)

Of course, the classification is not strictly suitable for any circumstance on road. For example, if the vehicle is approaching a static vehicle / or approaching a red traffic light, the vehicle should brake to stop to avoid collision / tickets. The state machine should still keep as high speed ADAS features, rather than jumping into APA features.

Secondly, we can use if the control direction is along the longitudinal direction or lateral direction to classify high speed ADAS features.

### Longitudinal direction control features:
- FDI: Forward Distance Indication
		- TODO: IMAGE NEEDED WITH FDI
		- This feature indicates how far the target vehicle is ahead of the host vehicle, in time space.
		- Usually we use TTC to describe the "how far" concept. TTC: Time to Collision, which is calculated from the division of the distance between host and target, and the speed difference between the host and target
- FCA: Forward Collision Alert
		- TODO: IMAGE NEEDED WITH FCA
		- This feature alerts the driver once TTC value is smaller than a thresold (can be calibrated)
- AEB: Automatic Emergency Brake
		- This feture sends brake control command to acuators once TTC value is smaller than a thresold (can be calibrated)
		- Usually we use 2 stages to send brake command to avoid over agressive brake control. For example, if the vehicle is lower than X kph, brake value would be 100%; if the vehicle is higher than X kph, brake value would be 40% at first several mili-seconds, and increase to 100% at the very last mili-seconds before the vehicle fully stops.
- ACC: Adaptive Cruise Control
		- This feature sends accelerate and brake (deceleratie) command to acuators once TTC value is larger / smaller than specific thresolds (can be calibrated)
		- This feature is a comfort feature where most people would use on highway in realistic life. The host vehicle (the vehicle driver drives) would accelerate and decelerate automatically according to the target vehicle action (in longitudinal direction). If the target vehicle is out of field of vision, the system would control the car at a constant speed (can be adjusted by drivers dynamically)
- RVB: Rear Virtual Bumper
		- This feature sends brake control command (at reverse gear, would not be effective when vehicle is driving forward) to acuators once TTC value is smaller than a thresold (can be calibrated)
- Other Longitudinal features will be released in later AD Framework version.

### Lateral direction control features
- LDW: Lane Depature Warning
		- This feature alerts the driver once the vehicle is deviated from the lane centering line. The alert timing is based on the distance (can be calibrated) between the vehicle center to the lane centering pivot point (w.r.t. lateral direction projection)
- LKA: Lane Keep Assist
		- This feature sends steering commands to acuators once the vehicle is deviated from the lane centering line. The commands would help the vehicle try to drive back to the centering line. The control command timing is based on the distance between the vehicle center to the lane centering pivot point.
		- Note that this feature is only an assist feature but not a comfort feature, which means the return-back steering will be aggressive. The vehicle would perform S curve if no driver intervention is added in between the steering triggers.
- LCC: Lane Centering Control
		- This feature sends steering commands to acuators continously to keep the vehicle driving at the lane center.
		- The concept HPP is the term we describe the lane center. In real world, HPP is a virtual line the camera / domain simulated according to the lanes (left and right)
		- Driving along HPP would bring benefits. First, HPP is a target line where we want the vehicle to drive along with; second, we can use the vehicle move trajectory to simulate a quasi-HPP for host vehicle to follow. In summary, using HPP would help LCC feature working normally with/without lanes detected.
- ILC: Instructed Lane Change
		- This feature sends steering commands to acuators once the driver/vehicle controller triggers the lane change command.
		- In theory, ILC feature would require either lane HPP, or target vehicle HPP is detected on the left side or right side of the host vehicle. If no HPP is detected at all, the feature should not be activated as the host vehicle would have no idea which HPP to follow after the vehicle depatures from the current HPP.
- SBZA: Side Blind Zone Alert
		- This feature alerts drivers when their is any risk target at the left/right side behind the host vehicle. The feature is mainly to help the driver to check if it is safe to change lane to the left or to the right. Of course, when ILC feature is activating, SBZA would also be helpful ILC for checking potential risks.
- Other Lateral features will be released in later AD Framework version.

### Parking features
- Parking features will be released in later AD Framework version.

## Classification of Sensors
Without eyes, we cannot see anything.  
Without sensors, vehicle cannot see anything. However, there are multiple sensor types among the market: camera, milimeter radars (long range and short range), ultrasonic sensors, lidars, etc.

- CAMERA: Camera is one of the most important, and probably the most widely used sensors among autonomous driving solutions. 
		- TODO: IMAGE NEEDED WITH REAL FCM
		- Camera uses their lens to send the image detected to the controller. The controller would then process the detected image and translate the image into semantic detection information, such as "sedan at 60m with 3deg azimuth, or pedestrian at 80m with -10deg azimuth". AD vehicle would require multiple cameras mounted around the vehicle to achieve wider range detection.
		- AD Framework initial release would include this sensor.

- LRR: Long Range Radar is also a commonly used sensors to detect the objects at max distance around 200m. 
		- TODO: IMAGE NEEDED WITH REAL LRR
		- The radar uses milimeter wave to detect objects. The nature of the milimeter wave brings LRR good performance to detect moving vehicles, and may perform poor when detecting static objects. Normally, a LRR would be mounted/packaged behind the front bumper.
		- AD Framework initial release would include this sensor.

- SRR: Short Range Radar is also a radar sensor to compensate the wakeness of the long range radar. 
		- TODO: IMAGE NEEDED WITH REAL SRR
		- Also, short range radar would bring cost down if the vehicle 360 deg information is required to be detected. Short range radar would also bring good performance to moving objects, but only limited to around 100m as their max distance.
		- AD Framework initial release would include this sensor.

- Lidar: Lidar is an advanced sensor with completely different technology theory than camera and normal milimeter radar. 
		- TODO: IMAGE NEEDED WITH REAL LIDAR
		- It uses lasers to detect objects. The nature brings the sensor a decent performance to detect not only moving objects, but also to static objects and other. It compensate most weakness over camera, LRR and SRR. However, the cost is also several times higher than camera and radar.
		- Lidar will be released in later AD Framework version.

- USS: Ultrasonic sensor is a good sensor when the controller needs low-speed object detection. 
		- TODO: IMAGE NEEDED WITH REAL USS
		- USS is good at serving for parking features where the vehicle is under a relative low speed (<10kph). USS sensor has a short detection range (max around 10m). It is cheap.
		- USS will be released in later AD Framework version.

- Other sensors: Sometimes AD vehicle would use other sensors to support advanced features, such as a capacitance transducer to detect if driver is puting their hands on the steering wheel, an IR camera (using infrared ray to detect) to detect if driver is abstracted or drowsy. These sensors are mostly packaged inside the cabin. This is more like a custom sensor class where customer/developer can design their own sensor style. AD Framework will provide a sensor basis (parent class) containing basic sensor functionality (such as the detection ability, and parameter adjust ability) for framework user finishing their further development.

## How AD framework helps your AD project?
AD framework seperates the whole AD technology into 2 parts: Sense the world, and Control the host. In AD framework initial release, we focus on the second part. The aim is to control the host vehicle properly and make it working as a "real" autonomous cars.

### Project setup
As chaos vehicle template has been largely modified from UE4 to UE5, the vehicle model base is not easy to migrate between the two versions. However, AD Framework is providing every project developers a workaround to make sure it works well in UE5.

- Project setup to versions prior to Unreal Engine 4.26.2
		- AD Framework is developed based on Unreal Engine V4.26.2 so we cannot guarantee full compatibility to versions before 4.26.2. If you are using those versions, please try to download the project and switch the version to the one you would like to use. If you, unfortunately found some bugs/defects reported in these early versions, please update your unreal engine version to at least 4.26.2.
- Project setup to Unreal Engine Version 4.26.2
		- Congrats! This is the version we were using to develop the framework. In theory you are good to go. If you still found some bugs/defects using 4.26.2, please report these information to *Ravidren@126.com*.
- Project setup to Unreal Engine Version 4.26.2+ and 4.27.x
		- As chaos vehicle template is not changed in these versions. In theory you are good to go as well. We have tested our framework in 4.27 version and found good compatibility. If you still found some bugs/defects using 4.26.2, please report these information to *Ravidren@126.com*.
- Project setup to Unreal Enginee Version 5.x.x
		- We would recommend you using UE4 to develop the AD vehicles. The reason is not because AD Framework is not fully compatible with UE5. There has been a few bugs/defects reported to Unreal Officials about the vehicle template and chaos vehicle itself. Unreal Engine 5 is still under early stage public development process. You may need to bear and solve unknown issues while you are developing your own vehicle projects.
		- If you were insisting to use Unreal Engine 5 with AD Framework, we are providing you a workaround. Please check the section below @*Project Modification to Unreal Engine 5*

### Project Modification to Unreal Engine 5
Note that in AD framework, the vehicle dynamics and the ADAS feature sensors and controllers are being seperated, which means chaos vehicle template is not a must needed as carrier to ADAS features. De-coupled ADAS feature sensors and controllers provide you the feasibility to migrate controllers and sensors individually into Unreal Engine 5. Please follow the steps below (or please check the video with link TBD).

1. Navigate to Content->AD->TBD, right click TBD, choose migrate, choose your Unreal Engine 5 project content folder.
2. Navigate to Content->AD->TBD, right click TBD (if you need Parking features as well), choose migrate, choose your Unreal Engine 5 project conent folder.
3. Navigate to content->AD->TBD, right click TBD, choose migrate, choose your Unreal Engine 5 project content folder.
4. Navigate to content->AD->TBD, right click TBD, choose migrate, choose your Unreal Engine 5 project content folder.
5. Navigate to content->AD->TBD, right click TBD, choose migrate, choose your Unreal Engine 5 project content folder.
6. Open your Unreal Engine 5 *.uproject* file, navigate to TBD, add child actors to the vehicle blueprint. In the details panel, choose the child actor class as the controllers/sensors migrated above.
		- For example, if you are looking forward to a 5R1V1D (5 radars, 1 vision camera, and 1 domain controller), you should add 7 child actors to the vehicle blueprint, in which choose 5 child actors class as radar, 1 as camera and 1 as domain controller)
7. Navigate to content->AD->UE5 Support, open actor blueprint TBD, in event graph, press ctrl+A, ctrl+C (copy all the custom events), and navigate to your Unreal Engine 5 vehicle blueprint, press ctrl+A, delete, and ctrl+B (paste all the copied custom events into the vehicle blueprint as the replacement to previous events)
		- the reason to do this is because the dependency on chaos vehicle between Unreal Engine 4 and Unreal Engine 5 are different. Old events are no longer valid in UE5.
8. Go to Project Settings, find input tab, click import from the top right of the editor window, navigate the path to content->AD->UE5 Support, choose *UE5 Input Setup.ini*.
		- the reason to do this is because the input name and input mapping (keyboard, mouse and controllers) are changed between UE4 and UE5. New mapping has been stored in the *UE5 Input Setup.ini*. This is only a convenience step, manual input is also ok.

### Project Layout
Core blueprints are stored in Content->AD folder.

#### Vehicle Blueprint
There are 3 vehicle blueprints in AD->Car path. Among them, BP_AD_Car is the one equipped with autonomous driving features. BP_RT_Car is the risk target template where developers can assign logics to them for behave as target. BP_Demo_Car is the demo car the framework used to display features in the demo room. BP_Demo_Car should not be used in the project AD framework users working on.

- BP_AD_Car
If you are just using AD framework as an AI car asset into your project (no vehicle physics such as transmissions and tyre parameters are required to change), feel free to create child class from BP_AD_Car and replace the static mesh with your owns. Otherwise, we advise you to use your own vehicle assets and add sensors & controllers onto it, instead of using BP_AD_Car directly.

Note: BP_AD_Car is currently set to be use Gear 1 all the time. Gear up and gear down actions are not activated. If in your project gear up and down is a must-have feature, you may need more fine tuning on the ADAS feature calibrations as gear up/down would consume the throttle/brake input no longer linearly.
 
**Parameters and Calibrations**

- AD Params->  
		- Car Size: controls the vehicle bounding box (X: length, Y: width and Z: height), unit in cm.
- Feature Status->  
		- controls the vehicle feature engagement (ON/OFF). Detailed feature explains please check the section *Classification of ADAS*
- ADAS Calibration Table->  
		- RT Lateral Distance Thresold: controls how far the risk target in lateral distance (RT) will be recognized. This array contains 10 elements describing the environments.  
		- TODO: IMAGE NEEDED  
			- RT0: risk target ahead of the host vehicle. RT0 is for deciding the HPP follow target.  
			- RT1 and RT2: risk target ahead of the host vehicle. RT1 is the nearest vehicle to the host, RT2 is the second nearest vehicle to the host vehicle.  
			- RT3 and RT5: risk target ahead of, but to the left of the host vehicle. RT3 is the nearest vehicle to the host on left, RT5 is the second nearest vehicle to the host on left.  
			- RT4 and RT6: risk target ahead of, but to the right of the host vehicle. RT4 is the nearest vehicle to the host on right, RT6 is the second nearest vehicle to the host on right.  
			- RT7: risk target behind of the host vehicle. RT7 is the nearest vehicle to the host behind the host.  
			- RT8: risk target behind of, but to the left of the host vehicle. RT8 is the nearest vehicle to the host behind on left.  
			- RT9: risk target behind of, but to the right of the host vehicle. RT9 is the nearest vehicle to the host behind on right.  
		- FCA & FDI Display Thresold: TBD  
		- FCA yellow TTC: the Time-To-Collision thresold to FCA yellow telltale alert. Yellow telltale indicates there is potetial risk to collide with the target vehicle. If the TTC time is smaller than yellow TTC (but larger than red TTC), FCA telltale would change from green to yellow.  
		- FCA red TTC: the TTC thresold to FCA red telltale alert. Red telltale indicates the collision risk is very high and the condition is urgent. If the TTC time is smaller than red TTC, FCA telltale would change from yellow to red.  
		- AEB Level1 Brake Axis Vlaue: how much of the brake value the AEB feature would request when alert level1 is triggered. Current vehicle template would clamp value to 1.0 if the request is higher than 1.0. If you want better brake performance (shorter brake distance or shorter brake-to-static time), please adjust the torque curve instead.  
		- AEB Level2 Brake Axis Value: how much of the brake value the AEB feature would request when alert level2 is triggred. To have Level2 brake would bring customer/vehicle user comfortness when braking-to-stop from a high speed value (larger than 40kph). In real feature implementation on autonomous driving, engineers would probably define more levels with different brake request value percent.  
		- ACC Default Set Speed: the target speed value when the user engage ACC for the first time. AD Vehicle would accelerate to this speed and stablize at this speed. User can adjust the set speed any time they want in the customer HMI panel. The user set value would override the default set speed value.  
				- Note: if you also turned on the LCC feature. LCC would also request to slow down the vehicle in urgent conditions like:  
					- vehicle is driving with S curve in the past few second. LCC would request to slow down the vehicle to keep the vehicle body driving stable along the prediction path.  
					- vehicle is going to drive along a high curve prediction path. LCC would request to slow down the vehicle in advance to keep the body driving stable along the prediction path.  
		- ACC TTC Value w/RT: the TTC value where ACC use as the thresold to brake and accelerate. If risk target is detected and is safe enough to follow, vehicle would use ACC system to keep the TTC as close as to this thresold. In this way, feature users would feel like the host vehicle is following the target vehicle constantly.  
		- ACC Control PID & EpochNum: this calibration class contains 3 sets of values, w/o RT, w/ RT and Brake2Stop. Each set contains 4 values, which are the parameters to a PID controller and the epoch number to the integration part.   
			- Note: PID controller is the simplest way to control a dynamic system with clear input and output. In AD framework, only propotion and integration values are used. Generally, A PI controller is enough to control the vehicle in AD framework. Sometimes, if you change the vehicle model into your own models, you may find overshooting is too obvious to work normally. If so, please also introduce the derivation part into PID controller.  
			- *what is PID controller?* Please refer to this ([PID Controller Explained • PID Explained](https://pidexplained.com/pid-controller-explained/)) where a normal PID controller is well-explained.   
