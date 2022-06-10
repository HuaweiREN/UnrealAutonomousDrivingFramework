# UnrealAutonomousDrivingFramework
The framework is to provide a straight forward and easy-to-use autonomous driving vehicle in unreal engine. The framework is based on UE chaos vehicle model, but not strictly limit to it, which means developers can easily replace the vehicle with their owns. The ADAS&APA control system and the vehicle model are de-coupled via several interfaces (throttle input, brake input, and steer input).  
Please note that the framework is developed under Unreal Engine ver.4.26.2. In theory, the framework should be compatible with any version later than 4.xx. Things might be a little tricky when coming to Unreal Engine 5 as the CHAOS vehicle model has been modified a lot in source code. There are some workaround provided in this framework. Should you be trapped into this condition, please refer to the section *Make framework alive in Unreal Engine 5*.


