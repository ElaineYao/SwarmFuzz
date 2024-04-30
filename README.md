# SwarmFuzzBinary

SwarmFuzzBinary is a fuzzing framework to efficiently find Swarm Propagation Vulnerabilities(SPVs) in drone swarms. It uses observation-based seed scheduling and binary search to find the potential attack parameters. 


## Get started

### 1) Requirement
Matlab R2022b with the Statistics and Machine Learning Toolbox. 

### 2) Clone the repository
We use [Swarmlab](https://github.com/lis-epfl/swarmlab) as the simulator for drone swarms.
```
git clone https://github.com/lis-epfl/swarmlab.git
git clone https://github.com/DependableSystemsLab/SwarmFuzz.git
```
Suppose that the Swarmlab repository is under the path `swarmlab`, and the SwarmFuzz repository is under the path `SwarmFuzz`.


### 3) Change the Swarmlab accordingly
We have to copy SwarmFuzz into the Swarmlab folder and also change some settings in Swarmlab to record the swarm's state.

Run the following commands
```
cp -r SwarmFuzz/fuzz/ swarmlab
cp SwarmFuzz/changed_swarmlab/compute_vel_vasarhelyi.m swarmlab/@Swarm
cp SwarmFuzz/changed_swarmlab/param_swarm.m swarmlab/parameters/
cp SwarmFuzz/changed_swarmlab/example_vasarhelyi.m swarmlab/examples/examples_swarm/
cp SwarmFuzz/changed_swarmlab/Swarm.m swarmlab/@Swarm
```
### 4) Open Swarmlab
Under `swarmlab/`, open MATLAB, 

```
cd swarmlab
matlab
```
### 5) Run SwarmFuzz
In the command window in MATLAB, run the following command
```
cd fuzz/fuzz
swarmfuzzbinary(200, 201, 5, 5) 
```

The arguments for the `swarmfuzzbinary` function are `(seedStart, seedEnd, dev, nb)`
- `seedStart`: the seed for the first mission 
- `seedEnd`: the seed for the last mission
- `dev`: GPS spoofing deviation
- `nb`: number of drones in the swarm
Therefore, by running `swarmfuzzbinary(200, 201, 5, 5)`, we are fuzzing the 5-drone swarm missions with seed 200 and 201, with the 5m GPS spoofing deviation.

### 6) Interprete the result
- `fuzz/seed_generation/seedpool{seed}.csv`: Generated seedpool for each mission. Each row represents `[seed, deviation_direction, target_id, victim1_id, victim2_id, start_t, spoofing_time]`. The number of rows represents the number of potential attack-victim drone pairs in this mission.
- `fuzz/search/gpsParam{seed}.csv`: Attack results found by SwarmFuzz for each mission. Each row represents 
`[collision_or_not, seed, deviation_direction, target_id, victim1_id, victim2_id, spoofing_start_time, spooing_duration]`. The number of rows present the attack results for each seed.
- `fuzz/search/paramTmp{seed}.csv`: Results for each binary search iteration. The number of rows represents the overhead of the fuzzing, i.e., the number of iterations taken to find the attack.  


## Configurations
- The default number of drones in the swarm is 5, if you want to test the swarm of 10/15 drones, do the following:
    - Change the `nb` argument according when calling `swarmfuzz(seedStart, seedEnd, dev, nb)`
    - In the file `parameters/param_swarm.m`, change the variable `p_swarm.nb_agents`.
    - In the file `@Swarm/Swarm.m`, in the function `get_colors(self)`, change `colors` to be an array with 10/15 rows. You can specify the color as you like.

## Collision cases found by SwarmFuzzBinary

Some collision cases found by SwarmFuzzBinary are stored in folder `./results`. The results for two tested algorithms (i.e., [Olfati-Saber](https://www.sciencedirect.com/science/article/pii/S1474667015386651) and the [Vicsek](https://www.science.org/doi/full/10.1126/scirobotics.aat3536) algorithms) are stored in seperate folders under `./results`.  

An example of the result for Olfati-Saber algorithm in the 5m GPS spoofing setting looks like this:
```
- RQ1_Olfati_Saber
    - 5drones
        - 5m-dev
            - param
                - gpsParam_csv200.csv
            - seed
                - seedpool200.csv
            - tmp
                - paramTmp_csv200.csv
```
The most important information is in `gpsParam_csv{seed}.csv` as it contains the GPS spoofing parameters found by SwarmFuzzBinary. Each row follows the format of 

`[collision, seed, GPS spoofing deviation, target drone, victim drone1 (, victim drone2 ), GPS spoofing start time, GPS spoofing duration]`.

For example, `[1,200,5,1,5,4,0,0.185546875]` means that, *for the mission with seed 200, collision between drone 5 and drone 4 occurs when drone 1 is under the 5m GPS spoofing starting at 0s, with the duration of 0.185546875s*. 

`seedpool{seed}.csv` and `paramTmp_csv{seed}.csv` contain some intermediate results during fuzzing. Interested readers could refer to the [./fuzz/seed_generation/drone_candidates.m](https://github.com/DependableSystemsLab/SwarmFuzz/blob/swarmfuzzbinary/fuzz/seed_generation/drone_candidates.m#L218) for the interpretation.



## Contact
For questions about our paper or this code, please open an issue or contact Elaine Yao (elainedv111@gmail.com)