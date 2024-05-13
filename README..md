For more detials please refer to AGAMAS paper in: https://link.springer.com/chapter/10.1007/978-3-031-43264-4_25.</br>
AGAMAS is compatible with SUMO (Simulation of Urbam MObility: https://eclipse.dev/sumo/). AGAMAS acts like a middleware to integrate SUMO and JADE. </br>
Before running make sure to have jade and libtraci jar files compiled in your project. These files can be found in this repository.</br>
The Main.java file contains an example of simulating a simple accident. Before running new project, make sure to copy the following directories into your project: 1. AV_Cluster 2. SumoVehicle 3. SUMOMAS.</br>
Files in directory "Validation_03_02_14_30_17_Calibration_Congested_02_01_7_00_17_30_w_0_font_size_fixed" contains simulation files that are needed to run our example. Make sure to define simulation files along with agent vehicle type and the routes they are supposed to take. </br>
The architecture of the porposed framework is represented in figure below:
</br>


<img src="https://github.com/SEED-CAVS/AGAMAS/blob/main/AGAMAS.png?raw=true" 
     width="400" 
     height="500" style="text-align:center;display:block"/>

     
</br></br></br>
All the agents are created through SUMOMAS class. All the agents are considered as Clusters (group of vehicles traveling together). Clusters simply are the JADE agents in charge of handling one or many vehicles in the simulation. To understand this concept, below there is a figure, demonstraing this process:
</br>
     <img src="https://github.com/SEED-CAVS/AGAMAS/blob/main/AOTS.png?raw=true" 
     width="400" 
     height="500" style="text-align:center;display:block"/>
</br>
In your main file, take an instance of SUMOMAS class. A command can be like the following:</br>
<pre>
  <code>
   String[] command = new String[]{"sumo-gui", "--delay", "30.0", "-c", "osm.sumocfg file path"};
   SUMOMAS sm = new SUMOMAS(false, command, 3780000);
  </code>
</pre>
This will take three values to initialize the SUMO and JADE (in the second line of the code above). The first parameter is boolean of JADE_GUI. The second parameter is command string array for the simulation and the third one is simulation time horizon.
To run efficiently, it is better to set value of the delay in the first code line to at least 30, and the step-length value in the sumo config file (in this project called osm.sumocfg in AGAMAS/Validation_03_02_14_30_17_Calibration_Congested_02_01_7_00_17_30_w_0_font_size_fixed/iteration_1/sim_files/osm.sumocfg) to less or equal to 0.1 (0.1 meaning that every second in simulation is equivalent to 1 second of real time). Note that Sumo displays the real time, and not the simulation time steps. </br>

Clusters can be created using SUMOMAS as well. 
First of all, it is needed to define an array of ids for the vehicles, which will give every vehicle a unique id. Creating the array will set how many vehicles are in the cluster. For example:
<pre>
     <code>
     ArrayList<String> ids = new ArrayList<String>();
        ids.add("1");
        ids.add("2");
        ids.add("3");
        ids.add("4");
     </code>
</pre>
</br>

In the previous example, an arraylist "ids" was created and it contains 4 ids, so 4 vehicles. Multiple clusters require the same amount of ids arraylist, as each cluster will have its own arraylist.

To create a cluster, it is only needed to call createCluster function just like the following code:
<pre>
     <code>
     AV_Cluster avc = sm.createCluster(ids, "agent", "agent_route", "20", "2", "0", "max", "current", "max", "", "", "", "", 0, 0);
     </code>
</pre>
</br>
First we pass the IDs of the vehicles which is the arraylist "ids" that we created before.
The second one is the vehicle type of these vehicles that belong to the cluster. It is important to note that the vehicle type defined for agents is better to be different than normal vehicles. An example that we have used, is located in textRou.rou.xml file in sim_files of "Validation_03_02_14_30_17_Calibration_Congested_02_01_7_00_17_30_w_0_font_size_fixed":


   
     <vType accel="10.0" color="red" decel="10.5" id="agent" length="5" speed="30.0" maxSpeed="40.0" sigma="0.5" depart="0" tau="0.5" minap="0.5" lcPushy="1.0" lcSigma="1.0" lcCooperative="0.0"/>
     <route id="agent_route" color="1,1,0" edges="main_0 main_1 main_2 main_3 main_4 main_5"/>

</br>

 and the default route they have to take. The rest of the parameters are just like defining a vehicle in libtraci:
 the 4th parameter is the departure time of the cluster (in real time), the 5th is the departure lane of the departure edge of the trip (here main_0), the 6th is the departure position in the departure edge and the 7th is the departure speed... For more information please refer to: https://sumo.dlr.de/docs/Definition_of_Vehicles%2C_Vehicle_Types%2C_and_Routes.html </br>

Since the clusters are agents you can simply define behaviours in JADE. </br>
It's corresponding route is also demonstrated above. These definitions and the values that can be used are problem-specific and can vary based on the demands of the experiments. 

Here are some useful commands that can be given to the cluster "avc" defined above, to obey during the simulation:
<pre>
     <code>
    SUMOMAS.doNSimStep(2000);
    avc.setSpeed(10.0);
     </code>
</pre>
</br>
which sets the speed of the cluster to 10 m/s after 2000 seconds of simulation (200 seconds of real time).

<pre>
     <code>
    SUMOMAS.doNSimStep(3000);
    avc.changeLane(3,100);
     </code>
</pre>
</br>
which commands the cluster to change its current lane into the lane of index 3 (which is the 4th lane because lane indexes start from 0) after 3000 seconds of simulation (300 seconds of real time). The second parameter is the time window where the cluster is allowed to change lanes: in this example, the cluster has 100 seconds of simulation time to change its lane. Note that if the time window is short and it closes without the cluster being able to change lanes (or any other command that requires a time window), the cluster will remain on its previously designated lane. Another important notice is that if the lane which the id is given to the command doesn't exist in the section of the trip where the cluster will be after 2000 seconds of simulation, it won't change lanes.


It is required to call the function of runSim() that is already defined in SUMOMAS to start the simulation.</br>