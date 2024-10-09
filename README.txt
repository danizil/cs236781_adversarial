Main function is run_attacks.py. 
================================
Training, testing and evaluation are done in "run_attacks_train()". The clean input is first calculated on all datasets and saved if the --save flags are called. After, the selected attack's "perturb" method is called in which a perturbation is trained on the dataset at each iteration and updates the perturbation if it performs better on the evaluation set. If an evaluation set is not chosen, it will be done on the test set. The optimized perturbation is then tested against the test set, and the optimization results and perturbed flow and pose data are saved if the --save flags are called.

Attacks folder 
==============
contains all of the different attacks tried. Most relevant attacks are in pgd.py and apgd.py. to use an attack use 
--attack pgd/apgd --alpha 0.05. for apgd also define --momentum. to set the number of iterations use --attack_k

loss.py 
=======
contains the loss criterion class VOCriterion. The default run will call mean_partial_rms loss on the training set and rms of the evaluation. A list of available flow and rotation terms can be found in VOCriterion __init__ function. There is a possibility to chose a factor by which the flow and rotation criterions are multiplied which defaults to 1.
to use, call e.g.
--attack_flow_crit div --attack_flow_factor 10. for rotation criterion call --attack_rot_crit.

                                            Saving output images flow and pose 
                                            ==================================
can be done with --save-best-pert --save-last-pert --save-flow --save-pose --save_csv. The program also saves the flow difference between clean and adversarial. To avoid saving the clean flow and pose use --dont-save-clean-flow-pose

                                            Testing, training, evaluation and overfit
                                            =========================================
The dataset class is defined in tartanTrajFlowDataset.py and inherits from pytorch's Dataset class. The datasets are defined at the beginning of the run and made into dataloaders which load the data for optimization and testing.
--test-folder (default -1) --eval-folder (default -1)
chose the relevant folder on which to test and evaluate. if -1 is chosen there will be no eval/train set. the folders that aren't eval or test folders will be used for training.

--overfit
=========
Will run overfitting on a single trajectory. this is useful for testing the implementation of new algorithms.

There is also an option for stochastic optimization. In this option we shuffle the training data and take batches in the optimization process. To activate chose --stochastic --shuffle-train. 
To chose batch size use --dan-batch-size <batch_size>. If the size of the data is not divisable by <batch_size>, a warning will appear.


                                            Data
                                            ====
The raw data should be placed in a folder in the code/ directory. Should consist of numbered folders that consist of trajectories: a sequence of images. Each trajectory should contain I0 and I1 albedo images and "blender_pose_file.csv" for the camera intrinsics, "patch_blender_pose.csv" and "pose_file.csv" for the ground truth data and "patch_pose_VO.csv" and "mask_coords.csv" for the patch positions.
the name of the overall dataset should be given at runtime with --data-dir "VO_adv_project_train_dataset_8_frames".

The code runs a preprocessing scheme and saves the preprocessed data to the folder code/data/<given_dataset_name>_processed. To avoid preprocessing at every run after the preprocessed data has been saved, run "--preprocessed_data" 


                                            To reproduce the results run:
                                            =============================

for baseline:
=============
 srun -c 2 --gres=gpu:1 --pty python run_attacks.py --seed 42 --model-name tartanvo_1914.pkl --data-dir "VO_adv_project_train_dataset_8_frames" --preprocessed_data --max_traj_len 8 --batch-size 1 --worker-num 2 --save_csv --save_best_pert --save_last_pert --attack pgd --attack_k 200 --alpha 0.05 --test-folder -1 --eval-folder <eval_folder> --attack_t_crit mean_partial_rms --attack_rot_crit none --attack_flow_crit none --attack_flow_factor 0 --attack_rot_factor 0
 
 for flow criterion with pgd
 ===========================
srun -c 2 --gres=gpu:1 --pty python run_attacks.py --seed 42 --model-name tartanvo_1914.pkl --data-dir "VO_adv_project_train_dataset_8_frames" --preprocessed_data --max_traj_len 8 --batch-size 1 --worker-num 2 --save_csv --save_best_pert --save_last_pert --attack pgd --attack_k 200 --alpha 0.05 --test-folder -1 --eval-folder <eval_folder> --attack_t_crit mean_partial_rms --attack_rot_crit none --attack_flow_crit <flow criterion> --attack_flow_factor 10 --attack_rot_factor 0  

for apgd
========
 srun -c 2 --gres=gpu:1 --pty python run_attacks.py --seed 42 --model-name tartanvo_1914.pkl --data-dir "VO_adv_project_train_dataset_8_frames" --preprocessed_data --max_traj_len 8 --batch-size 1 --worker-num 2 --save_csv --save_best_pert --save_last_pert --attack apgd --attack_k 200 --alpha 0.05 --test-folder -1 --eval-folder <eval_folder> --attack_t_crit mean_partial_rms --attack_rot_crit none --attack_flow_crit none --attack_flow_factor 1 --attack_rot_factor 0 --momentum 0.1

 for div flow criterion with apgd
 ================================
 srun -c 2 --gres=gpu:1 --pty python run_attacks.py --seed 42 --model-name tartanvo_1914.pkl --data-dir "VO_adv_project_train_dataset_8_frames" --preprocessed_data --max_traj_len 8 --batch-size 1 --worker-num 2 --save_csv --save_best_pert --save_last_pert --attack apgd --attack_k 200 --alpha 0.05 --test-folder -1 --eval-folder <eval_folder> --attack_t_crit mean_partial_rms --attack_rot_crit none --attack_flow_crit div --attack_flow_factor 10 --attack_rot_factor 0 --momentum 0.05

For running without dataset 2:
==============================
Simply add --no-2 at the end of every run.

CONDA packages:
===============
added packages can be found in install_pytorch_cupy_3.txt