# dataMPC_parking
Collision Avoidance in Tightly-Constrained Environments without Coordination: a Hierarchical Control Approach

[Webpage](http://bit.ly/data-sg-control),  [arXiv](https://arxiv.org/abs/2011.00413)

Authors:
- Xu Shen, xu_shen@berkeley.edu
- Edward Zhu, edward_zhu@berkeley.edu
- Yvonne R. Stürz, y.stuerz@berkeley.edu

This repository contains all codes for MATLAB prototyping and simulation. The Python+ROS implementation on BARC car can be found in [`mpclab_strategy_obca`](https://github.com/MPC-Berkeley/mpclab_strategy_obca).

## Dependencies
### MATLAB:
1. MPT Toolbox and Yalmip (For solving and viz)
2. Parallel Computing Toolbox (Only for generating dataset)
3. Deep Learning Toolbox (For network training and predicting)
4. CasADi (For controller formulation)
5. Forces Pro (For real-time implementation)

### Python:
1. ROS Noetic
2. Packages for pip-installation are listed in `requirement.txt`

## File Structure:
1. `./bag_processing/`: The ipython notebooks for parsing experiment data and plot
   1. `bag_processing.ipynb` is to plot the static images
   2. `video_processing.ipynb` is to plot the video
2. `./extract_traj/`: The ipython notebooks for extracting data from CARLA data folder
3. `./learning/`: Different models to learn the mapping from relative configuration to startegies
   1. `strategy_learning/`: learn strategy classifers
      1. `strategy_datagen.m`: Generate strategy labels from offline rollouts
      2. `load_train_classify.m`: Load the strategy labels and offline rollouts and make dataset. Train different models.
      3. `predict_strategy_filter.m`: Apply a filter to the strategy prediction and visualize it along with offline rollouts
4. `./nominal_MPC/`: OBCA to solve the collision avoidance problem and produce rollouts offline
   1. `path_planning_tv_CFTOC.m`: Script to solve one individual scenario with time varying OBCA
   2. `path_planning_datagen_par.m`: Script to generate a bunch of rollouts with parallel process
5. `./online_MPC/`: SG and BL-OBCA onine
   1. `constraint_generation/`: Strategy Predictor Class and the functions to construct constraints
   2. `controllers/`: The controller classes
      1. `ebrake_controller.m`: The Emergency Break controller
      2. `hpp_obca_controller_FP.m`: The SG-OBCA controller
      3. `obca_controller_FP.m`: The BL-OBCA controller
      4. `safety_controller.m`: The safety controller
   3. `dynamics/`: Vehicle dynamics
   4. `experiments/`: The main experiments scripts. Note that the Baseline method are referred as "Naive" in all codes 
      1. `experiment.m`: **The script to pick scenario and run experiment**
      2. `datagen.m`: The script to solve all scenarios and generate statistics
      3. `FSM_HOBCA_naive_fp.m`: The function to use BL control framework (Set of policies: BL-OBCA, Safety, Emergency-Brake)
      4. `FSM_HOBCA_strat_fp_infalted.m`: The function to use SG control framework (Set of policies: SG-OBCA, Safety, Emergency-Brake)
   5. `plotting/`: Functions to plot
   6. `utils/`: Utils for running experiments
      1. `FiniteStateMachine.m`: The class definition to select different control policies

## Change log
### 10/01/2020
1. Reorganize the files. Now `FSM_HOBCA_naive_fp.m` and `FSM_HOBCA_strat_fp.m` are all functions. Use `experiment.m` or `datagen.m` to call them.
2. Added `Safe-Yield` state.

### 09/29/2020
1. Datagen with FSM

### 09/28/2020
1. Code up the Finite State Machine formulation (`./online_MPC/utils/FiniteStateMachine.m`, `./online_MPC/experiments/FSM_online_HOBCA_strat_fp.m`)
2. Changed the `plotExp.m` to accomodate data files generated by FSM.
3. Change some function name:
	1. `check_collision.m` -> `check_collision_point.m`
	2. `check_current_collision.m` -> `check_collision_poly.m`
4. Wrap the neural network and filter into `./online_MPC/constraint_generation/StrategyPredictor.m`

### 09/22/2020
1. Have another flow of control logic (`./online_MPC/experiments/BACK_*`):
	1. The score will influence the safety controller;
	2. If the HOBCA is feasible, follow the mode given by score;
	3. If HOBCA is infeasible, activate the safety controller too;
	4. In safety controller, if backing up still lead to collision, activate emergency break (which is equivalent to collision here).
2. Updated the input to `check_collision_poly.m` function and its usage.
3. Updated the `./online_MPC/experiments/datagen.m` to have simple evaluation too.

### 09/21/2020
1. Added up and bottom constraints;
2. Datagen code for `hobca_strat` and `hobca_naive` are finished.

### 09/16/2020
1. Fixed bug in `./online_MPC/dynamics/bike_dynamics_rk4.m`
2. Added forces pro implementations of naive receding horizon MPC and collision buffer MPC
3. A bunch of plotting updates in `./online_MPC/plotting/plotExp.m`
4. Standardized controller and experiment file formats
5. Forces pro generated files are dumped into `./online_MPC/experiments/forces_pro_gen`

### 09/11/2020
1. Added rate constriants to safety controller.
2. Always solve HOBCA in every timestep. If the threshold just comes above the threshold, the HOBCA is applied only if it is feasible.
3. Log more information: actual collision at every time step, the entire time every iteration.
4. (**TODO**) Add a slack to HOBCA distance constriants to encourage larger distance.

### 09/04/2020
1. Added Forces Pro for HppOBCA (Ed). But it will becomes infeasible at some time.
2. The exitflag = -6 bug was resolved.
3. Dump the plotting and movie generation into a function `./online_MPC/plotting/plotExp.m`
4. Log the console output into diary file.

### 08/30/2020
1. Added Forces Pro implementation. The class is defined in `./online_MPC/controllers/HPPControlFP.m` and the experiment script is `./online_MPC/experiments/online_HPP_force.m`
2. The Forces Pro generated files are all added into `.gitignore` with the format `FP_*`

### 08/27/2020
1. Added the CasADi implementation (Ed).
2. Reorgainzed the files. Now all the experiment scripts are in `./online_MPC/experiments`, using either class or CasADi formulation of controllers. All CFTOC-based scripts and functions are in `./online_MPC/exp_controllers_CFTOC`.

### 08/21/2020
1. Added the safety controller (Ed)
2. Rewrite all controllers in classdef. `MpcController` is the base class, and `HppController`, `NivController`, `HPPobcaController` are subclasses.
3. Added the up and bottom constraints into `MpcController`.

### 08/18/2020
1. Added `ref_v` for tracking. The score will only discount the `ref_v`, instead of `ref_x`.
2. Lock the strategy if there are already a certain steps colliding.

### 08/13/2020
1. Added `online_MPC/score_KF.m` for filtering strategy prediction, which helps to overcome sudden jump.

### 08/12/2020
1. The biased hyperplane seems to not perform well... Instead, we can bias the reference trajectory based on strategy prediction scores. The current logic is: if `max(score) < 0.5`, it will yield; if [0.5, 0.6] it will uses a discounted `v_ref`; if `max(score) > 0.6`, it will use the full `v_ref`. Still need to be verified across many test exps.
2. Fixed the annoying problem of dyn model frame. Now everything is in CoG, and the inputs are **[beta, a]**, where `beta = atan(lr/(lf+lr)*tan(delta))`. Since this function is also monotonic for `delta`, the real vehicle experiment can easily transform it back to `delta`.
3. Changed the input limit. The max acc is now `[-1, 1]` and beta is `[-0.35, 0.35]`.

### 08/11/2020
1. Encode the hyperplane constraints into OBCA formulation (`online_MPC/HPPobca_CFTOC.m`). The warm start can be the extended previous iteration. The waypoints to detect collision can also be the previous extend previous z_opt. (`online_MPC/online_HOBCA.m`)

### 08/10/2020
1. Generate the score-biased hyperplane (Edward)
2. Add the option of score biased hyperplane into `online_MPC/online_HOBCA.m` and `online_MPC/online_hppMPC.m`

### 08/07/2020
1. Moved `learning/models` into `./models`
2. Created folder `./online_MPC` and computed the hyperplane online (Edward)
3. Designed the online MPC controller based on the computed hyperplane, and compare it with the naive formulation. `online_MPC/online_hppMPC.m` compares the hyperplane constrained circular model with naive circular one; `online_MPC/online_HOBCA.m` compares the "hpp warm-started" HOBCA with naive circular version.

### 08/04/2020
1. Added NN classifier for strategy prediction (`learning/nn_clas.m`) and added one-hot label variables for it (`learning/load_train_classify.m`)
2. Modified predict and viz script to show strategy posterior probability at the same time (`learning/predict_vs_opt_strategy.m`)

### 08/03/2020
1. Added the strategy generation (`nominal_MPC/check_strategy_labels.m` and `nominal_MPC/generat_strategy_data.m`) and its parallel datagen version for constructing dataset (`learning/strategy_datagen.m`).
2. Changed `learning/loda_train_model.m` to `learning/load_train_regression.m` for the specific use of regression of hyperplane.
3. Added `load_train_classify.m` for constructing dataset for strategy classification.
4. Added classifiers: Gaussian SVM (`learning/gSVM.m`), bagged tree (`learning/bagTree.m`), and KNN (`learning/knn.m`)

### 07/31/2020
1. Small modification of saving and loading.
2. Added the movie recording function. (`learning/predict_vs_opt_midpoint.m`)

### 07/30/2020
1. Manually divided the \~80% training set and 20% validation set for NN and GP (`learning/load_train_model.m`)
2. Simplified the GP code, so that it only fits once, output compact model, and offer validation MSE. (`gp.m`)

### 07/29/2020
1. Added the GP training. For every element in the flattened label vector, train one GP for it, with all feature dimensions as the predictor. (`learning/gp.m`)
2. Renamed `learning/train_network.m` into `learning/load_train_model.m` so that user has the option to train either NN or GP. The NN training is also organized into `learning/nn.m`
3. Changed the prediction and viz script so that it can switch between GP and NN. (`learning/predict_vs_opt_midpoint.m`)

### 07/28/2020
1. Changed the `learning/gen_feature_label.m` so that it will switch between mid-point mode or free hyperplane mode
2. Added mid-point into `learning/train_network.m` and seperated the normalization process. (The normalized variable is not used for training now) Also added a new variable `reg_data` for regression toolbox.
3. Created a new script for prediction and visualization in mid-point mode. (`learning/predict_vs_opt_midepoint.m`)

### 07/27/2020
1. Customized the training code (`learning/train_network.m`)
2. Check the network prediction vs opt (`learning/predict_vs_opt.m`)
3. Reorganized the data folder:
	1. `./data/` is the folder to place trajectory examples
	2. `./hyperplane_dataset/` is the folder to place generated hyperplanes for training
	3. `./learning/models/` is the folder to place learned models
4. Added the hyperplane generation v2, where the hyperplanes are forced to go across the middle point of two sets, and slope is the free decision variable. (`nominal_MPC/generate_hyperplane_v2.m` and `learning/hyperplane_datagen_v2.m`)
5. ~~**TODO:** The slope will have the wrapping issue and jumps from pi/2 to -pi/2. Need to smooth it in training feature.~~

### 07/26/2020
1. Reorganized the folder tree, placed all learning related into `./learning/`
2. Generate hyperplanes from all scenarios and make the dataset (`learning/hyperplane_datagen.m`)
3. Train a shallow 2-layer FC network and save the model (`learning/train_network.m`)

### 07/25/2020
1. Generate optimal hyperplanes by solving a SVM problem: The current vehicle vertices are hard constraints, the future vertices are soft with slack var. (`nominal_MPC/generate_hyperplanes.m`, Author: Edward)
2. Generate hyperplane dataset by traversing all data files. (`learning/hyperplane_datagen.m`)

### 07/23/2020
1. Cleaned the code:
	1. Script for Time Invariant Obstacles: `nominal_MPC/path_planning_ti.m`
	2. Script for Time Varying Obstacles: `nominal_MPC/path_planning_tv_CFTOC.m` and `nominal_MPC/*_datagen*`
	3. Scipt for Data loading and visualization: `nominal_MPC/data_viz.m`
	4. Controller Functions: `nominal_MPC/OBCA.m`, `nominal_MPC/OBCA_tv`, `emergency_break.m`, `nominal_MPC/speed_controller.m`
	5. Warm Start Functions: `nominal_MPC/DealWultWS.m`, `nominal_MPC/DualMultWS_tv.m`, `nominal_MPC/hybrid_A_star.m`, `nominal_MPC/unicycleWS.m`
	6. Dynamics and other utils: `nominal_MPC/bikeFE.m`, `nominal_MPC/check_collision_*`, `nominal_MPC/plotCar.m`

### 07/22/2020
1. Added data loading and viz script (`data_viz.m`)

### 07/20/2020
1. Added the parallel computing version for faster dataset generation (`path_planning_datagen_par.m`)

### 07/13/2020
1. Path planning with the CFTOC formulation (`path_planning_tv_CFTOC.m`):
	1. Use the simple reference traj to check the collision. If no collsion, adjust the initial state to produce it.
	2. Try to solve the exact collision avoidance problem. Firstly, use the unicycle model as state WS, and then dual WS, and finally OBCA.
	3. If the exact problem cannot be solved, try to solve a speed profile on ref path for collision avoidance.
	4. If the speed profile cannot be solved too, solve a most conservative emergency break.
2. Iteratively run over all `exp_num`s and generate data with log file (`path_planning_datagen.m`)
3. Minor fix about constraints of the controller.

### 07/12/2020
1. HOBCA with time variant obstacle formulation. (But in the test, the obstacle remains static for now.)
2. Use the simplified vehicle model and collision avoidance constraints for WS, rather than Hybrid A\*. Because the Hybrid A\* may produce reverse motion due to the collision buffer.
3. Changed some function API.

### 07/06/2020
1. HOBCA with multiple obstacles added.
2. TODO: dynamic object.

### 07/02/2020
1. HOBCA deployed in MATLAB where only one static obstacle is present. The file is `path_planning.m`.
2. Workflow:
	1. Obstacle is firstly defined as Polyhedron;
	2. The figure is captured as a frame for Hybrid A* toolbox;
	3. The planned path is smoothened;
	4. Dual multipliers are firstly warm started;
	5. The warm starting states are also computed;
	6. The main optimization is solved
3. TODO: Add num of obstacles; Think about dynamic object.
