---
title: "Memory Profiling with Python"
date: 2024-02-14T23:00:00-05:00
draft: false
---

_Originally published on 2021-1-26 as an internal tech blog post when I was the Tech Lead for Wayfair's Experimentation Platform team. Internal links will not work._

### Where I learn about memory profiling in Python

We've observed that Simulations can fail when run in Kubernetes pods. We suspect the reason is that these pods are under-resourced and are running out of memory, then being killed/restarted by Kubernetes in order to keep the overall cluster healthy.

I found this [doc](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#how-pods-with-resource-limits-are-run) from Kubernetes that discusses what happens when a Container exceeds memory limits.

> If a Container exceeds its memory limit, it might be terminated. If it is restartable, the kubelet will restart it, as with any other type of runtime failure.

> If a Container exceeds its memory request, it is likely that its Pod will be evicted whenever the node runs out of memory.

> A Container might or might not be allowed to exceed its CPU limit for extended periods of time. However, it will not be killed for excessive CPU usage.

In the absence of logs, becasue the processes keep getting killed and Containers keep getting rolled over, if I'm able to figure out how much memory our Simulations use, we could be reasonably sure that exceeding those limits are what ends up killing the processs in prod.

@jliu2 from the Data Science team has been helping us build the Simulator tool and has the ability to run Simulations locally, so I asked her for the code to do that. It was in a Jupyter Notebook, so I copypasted that into a standard python file that could be run from the terminal because I knew that I wanted to wrap it in a profiler.

`sim_demo.py`

```
import numpy as np
import pandas as pd
import algorithm_picker_utils as utils
import matplotlib.pyplot as plt
import datetime
import time
import simplejson as json

historical_data = pd.read_csv("App_Interstitial.csv")

historical_data_list = historical_data.values.tolist()

print(historical_data_list)


def extract_traffic_history_and_control_performance(
    traffic_history_and_control_performance,
):
    """
    Args:
        traffic_history_and_control_performance (list): list of data shaped like [time, traffic_history, control performance]:
            [["2020-6-1 11:13:48", "5009", "0.2919"]]
    Returns:
        Tuple: (list: control_performance_history, list: traffic_history)
    """
    #         traffic_and_performance_list = json.loads(traffic_history_and_control_performance)
    traffic_and_performance_list = traffic_history_and_control_performance
    timestamp_list = []
    traffic_history_list = []
    control_performance_history_list = []
    for data in traffic_and_performance_list:
        time_as_date = pd.to_datetime(data[0], infer_datetime_format=True)
        timestamp_list.append(time_as_date)
        traffic_history_list.append(data[1])
        control_performance_history_list.append(data[2])
    traffic_history = pd.DataFrame(
        {"time": timestamp_list, "traffic_size": traffic_history_list}
    )
    control_performance_history = pd.DataFrame(
        {
            "time": timestamp_list,
            "control_performance": control_performance_history_list,
        }
    )
    return traffic_history, control_performance_history


(
    traffic_history,
    control_performance_history,
) = extract_traffic_history_and_control_performance(historical_data_list)

control_performance_history["control_performance"] = control_performance_history[
    "control_performance"
].astype(float)

traffic_history["traffic_size"] = traffic_history["traffic_size"].astype(int)

smallest_KPI_increase = 0.1
max_duration = 14
num_arm = 3

start_time = time.time()

result_dict = utils.algorithm_pick(
    control_performance_history,
    traffic_history,
    smallest_KPI_increase,
    max_duration,
    num_arm,
    test_arm_performance_dist="even",
    test_arm_performance_increase=None,
    epoch_type="day",
    traffic_perc=1.0,
    default_n_pure_exploration=-1,
    eg_explore_ratio=0.1,
    eg_explore_rate_type="decay",
    algorithm_list=[
        "ALGORITHM_AB",
        "ALGORITHM_EPSILON_GREEDY",
        "ALGORITHM_UPPER_CONFIDENCE_BOUND",
        "ALGORITHM_THOMPSON_SAMPLING",
    ],
    termination_significance=0.01,
    confidence_interval=0.95,
    param_num_sim=50,
    num_sim=100,
    right_clustered_alpha_beta=(3, 0.5),
    left_clustered_alpha_beta=(0.5, 3),
    two_end_clustered_alpha_beta=(0.3, 0.3),
    AA_test=False,
    param_converge_prob_threshold=0.9,
    sim_converge_prob_threshold=0.8,
    T0_explore_start_ratio=0.5,
    num_iteration=10,
)


print("---Total Execution Time: %s seconds ---" % (time.time() - start_time))
```

And the input data for historical control performance `App_Interstitial.csv` is:

```
time, traffic_size, control_performance
6/1/2020 11:13,47408,0.0437
6/2/2020 11:13,44469,0.0387
6/3/2020 11:13,39045,0.0426
6/4/2020 11:13,39572,0.0442
6/5/2020 11:13,51585,0.0422
6/6/2020 11:13,61115,0.0386
6/7/2020 11:13,47426,0.0411
6/8/2020 11:13,35489,0.0409
6/9/2020 11:13,37904,0.0366
6/10/2020 11:13,35804,0.0408
6/11/2020 11:13,31832,0.0428
6/12/2020 11:13,40610,0.0397
6/13/2020 11:13,53562,0.0432
6/14/2020 11:13,33323,0.0448
6/15/2020 11:13,22867,0.0429
6/16/2020 11:13,29738,0.0452
6/17/2020 11:13,28665,0.0432
6/18/2020 11:13,30365,0.0406
6/19/2020 11:13,44988,0.0396
6/20/2020 11:13,53480,0.0406
6/21/2020 11:13,47712,0.0433
6/22/2020 11:13,33760,0.0388
6/23/2020 11:13,34132,0.0411
6/24/2020 11:13,29499,0.0393
6/25/2020 11:13,35416,0.0413
6/26/2020 11:13,61542,0.0396
6/27/2020 11:13,78964,0.0364
6/28/2020 11:13,62616,0.0354
6/29/2020 11:13,51098,0.0379
6/30/2020 11:13,55058,0.0371
7/1/2020 11:13,48141,0.0368
```

I then searched for line-by-line memory profiling tools that work in python and found [memory-profiler](https://pypi.org/project/memory-profiler/). It looked easy to use and would give the memory used per line of application code.

`algorithm_picker_utils.algorithm_pick` is the core of the above code, where the core Data Science happens, so I placed the `@profile` memory-profile decorator on that function, since I have the source of that library.

Running it with profiling enabled:

`python -m memory_profiler sim_demo.py`

The program produces output in 306 seconds:

```
2021-01-26 08:58:54 [4842] [INFO] ds_mab_algos     Starting algorithm_pick()
Finished running algorithm_pick()
2021-01-26 09:04:01 [4842] [INFO] ds_mab_algos     Finished running algorithm_pick()
---Total Execution Time: 306.874724149704 seconds ---
```

And the memory profiler gives this output:

```
Line #    Mem usage    Increment  Occurences   Line Contents
============================================================
   698  113.914 MiB  113.914 MiB           1   @profile
   699                                         def algorithm_pick(
   700                                             control_performance_history,
   701                                             traffic_history,
   702                                             smallest_KPI_increase,
   703                                             max_duration,
   704                                             num_arm,
   705                                             test_arm_performance_dist="even",
   706                                             test_arm_performance_increase=None,
   707                                             epoch_type="day",
   708                                             traffic_perc=1.0,
   709                                             default_n_pure_exploration=-1,
   710                                             eg_explore_ratio=0.1,
   711                                             eg_explore_rate_type="decay",
   712                                             algorithm_list=[
   713                                                 "ALGORITHM_AB",
   714                                                 "ALGORITHM_EPSILON_GREEDY",
   715                                                 "ALGORITHM_UPPER_CONFIDENCE_BOUND",
   716                                                 "ALGORITHM_THOMPSON_SAMPLING",
   717                                             ],
   718                                             termination_significance=0.01,
   719                                             confidence_interval=0.95,
   720                                             param_num_sim=50,
   721                                             num_sim=100,
   722                                             right_clustered_alpha_beta=(3, 0.5),
   723                                             left_clustered_alpha_beta=(0.5, 3),
   724                                             two_end_clustered_alpha_beta=(0.3, 0.3),
   725                                             AA_test=False,
   726                                             param_converge_prob_threshold=0.9,
   727                                             sim_converge_prob_threshold=0.8,
   728                                             T0_explore_start_ratio=0.5,
   729                                             num_iteration=10,
   730                                         ):
   731                                             """
   732                                             Perform one round of:
   733                                                 Generate and return simulated performance for each method
   734
   735                                             Parameters:
   736                                                 control_performance_history (df): At least one month historical control performance,
   737                                                                                   columns = ['time', 'control_performance']
   738                                                 traffic_history (df): At least one month historical traffic size, columns = ['time', 'traffic_size']
   739                                                 smallest_KPI_increase (float): Smallest KPI increase expected to observe from testing
   740                                                 max_duration (int): Maximum number of days the test is allowed to run for
   741                                                 num_arm (int): Number of arms
   742                                                 test_arm_performance_dist (str): even, random, control_clustered, test_clustered, two_end_clustered, default = "even"
   743                                                 test_arm_performance_increase (array): Expected performance increase of each test arm compared to control arm,
   744                                                                                        not including control arm, default = None
   745                                                 epoch_type (str): Frequency of learning, default = "day"
   746                                                 traffic_perc (float): Percentage of traffic to be used in testing, default = 1.0
   747                                                 default_n_pure_exploration (int): default evenly split duration, if >= 0, then use this default duration for all methods
   748                                                 eg_explore_ratio (float): Epsilon value for Epsilon Greedy algorithm, default = 0.1
   749                                                 eg_explore_rate_type (str): Type of epsilon value decay for Epsilon Greedy algorithm, default = "decay"
   750                                                 algorithm_list (list): algorithms to run simulation for
   751                                                 termination_significance (float): Significance to determine test termination
   752                                                 confidence_interval (float): Confidence interval of performance from simulations, default = 0.95
   753                                                 param_num_sim (int): Number of simulations to find the optimal parameter for each method, default = 50
   754                                                 num_sim (int): Number of simulations to generate performance for each method, default = 100
   755                                                 right_clustered_alpha_beta (tuple): Beta distribution for right clustered, default = (3, 0.5)
   756                                                 left_clustered_alpha_beta (tuple): Beta distribution for left clustered, default = (0.5, 3)
   757                                                 two_end_clustered_alpha_beta (tuple): Beta distribution for two end clustered, default = (0.3, 0.3)
   758                                                 AA_test (Boolean): Perform AA test or not, default = False
   759                                                 param_converge_prob_threshold (float): Threshold for converge probability to find parameter, default = 0.9
   760                                                 sim_converge_prob_threshold (float): Threshold for converge probability to final simulation, default = 0.8
   761                                                 T0_explore_start_ratio (float): Ratio of AB test duration from where to start T0 trial for other MAB algorithms,
   762                                                                                 default = 0.5
   763                                                 num_iteration (int): Number of iterations (One iteration contains the number of simulations specified in num_sim)
   764
   765                                             Returns:
   766                                                 result_dict (dict): performance dict for each algorithm in the following format
   767                                                 {
   768                                                 4: {'converge_duration_mean': n1,
   769                                                 'converge_duration_confidence_interval': array([a1, b1]),
   770                                                 'converge_probability': m1,
   771                                                 'evenly_split_duration': z1},
   772                                                 2: {'converge_duration_mean': n2,
   773                                                 'converge_duration_confidence_interval': array([a2, b2]),
   774                                                 'converge_probability': m2,
   775                                                 'evenly_split_duration': z2},
   776                                                 3: {'converge_duration_mean': n3,
   777                                                 'converge_duration_confidence_interval': array([a3, b3]),
   778                                                 'converge_probability': m3,
   779                                                 'evenly_split_duration': z3},
   780                                                 1: {'converge_duration_mean': n4,
   781                                                 'converge_duration_confidence_interval': array([a4, b4]),
   782                                                 'converge_probability': m4,
   783                                                 'evenly_split_duration': z4}
   784                                                 }
   785                                                 random example:
   786                                                 {4: {'converge_duration_mean': 23.5254185191477,
   787                                                 'converge_duration_confidence_interval': array([21.72888813, 25.32194891]),
   788                                                 'converge_probability': 0.9999999999999999,
   789                                                 'evenly_split_duration': 0.0},
   790                                                 2: {'converge_duration_mean': 20.902511807636067,
   791                                                 'converge_duration_confidence_interval': array([19.10909269, 22.69593092]),
   792                                                 'converge_probability': 0.8979999999999998,
   793                                                 'evenly_split_duration': 14.000000000000002},
   794                                                 3: {'converge_duration_mean': 21.094068749884887,
   795                                                 'converge_duration_confidence_interval': array([19.14578034, 23.04235716]),
   796                                                 'converge_probability': 0.9999999999999999,
   797                                                 'evenly_split_duration': 10.000000000000002},
   798                                                 1: {'converge_duration_mean': 22.74253083272616,
   799                                                 'converge_duration_confidence_interval': array([20.44126719, 25.04379447]),
   800                                                 'converge_probability': 0.9609999999999999,
   801                                                 'evenly_split_duration': 12.400000000000002}}
   802                                             """
   803  113.918 MiB    0.004 MiB           1       try:
   804  114.055 MiB    0.137 MiB           1           logger.info("Starting algorithm_pick()")
   805
   806                                                 # Start simulation for each algorithm
   807  114.055 MiB    0.000 MiB           1           performance_list = []
   808  114.055 MiB    0.000 MiB           1           best_alg_ind_list = []
   809 1758.879 MiB    0.000 MiB          11           for i in range(num_iteration):
   810 1758.879 MiB 1644.820 MiB          20               performance, best_alg_ind = single_algorithm_pick(
   811 1680.086 MiB    0.000 MiB          10                   num_arm,
   812 1680.086 MiB    0.000 MiB          10                   max_duration,
   813 1680.086 MiB    0.000 MiB          10                   smallest_KPI_increase,
   814 1680.086 MiB    0.000 MiB          10                   control_performance_history,
   815 1680.086 MiB    0.000 MiB          10                   traffic_history,
   816 1680.086 MiB    0.000 MiB          10                   test_arm_performance_dist,
   817 1680.086 MiB    0.000 MiB          10                   test_arm_performance_increase,
   818 1680.086 MiB    0.000 MiB          10                   traffic_perc,
   819 1680.086 MiB    0.000 MiB          10                   algorithm_list,
   820 1680.086 MiB    0.000 MiB          10                   default_n_pure_exploration,
   821 1680.086 MiB    0.000 MiB          10                   eg_explore_ratio,
   822 1680.086 MiB    0.000 MiB          10                   eg_explore_rate_type,
   823 1680.086 MiB    0.000 MiB          10                   termination_significance,
   824 1680.086 MiB    0.000 MiB          10                   confidence_interval,
   825 1680.086 MiB    0.000 MiB          10                   param_num_sim,
   826 1680.086 MiB    0.000 MiB          10                   num_sim,
   827 1680.086 MiB    0.000 MiB          10                   right_clustered_alpha_beta,
   828 1680.086 MiB    0.000 MiB          10                   left_clustered_alpha_beta,
   829 1680.086 MiB    0.000 MiB          10                   two_end_clustered_alpha_beta,
   830 1680.086 MiB    0.004 MiB          10                   sim_converge_prob_threshold,
   831 1680.086 MiB    0.000 MiB          10                   param_converge_prob_threshold,
   832 1680.086 MiB    0.000 MiB          10                   T0_explore_start_ratio,
   833                                                     )
   834 1758.879 MiB    0.000 MiB          10               performance_list.append(performance)
   835 1758.879 MiB    0.000 MiB          10               best_alg_ind_list.append(best_alg_ind)
   836 1758.895 MiB    0.016 MiB           1           best_alg_ind = Counter(best_alg_ind_list).most_common(1)
   837
   838                                                 # Generate performance result for each algorithm
   839 1758.895 MiB    0.000 MiB           1           result_dict = {}
   840 1758.895 MiB    0.000 MiB           5           for i in range(len(algorithm_list)):
   841
   842 1758.895 MiB    0.000 MiB           4               temp_alg = ALGORITHM_ID_REFERENCE[algorithm_list[i]]
   843 1758.895 MiB    0.000 MiB           4               result_dict[temp_alg] = {
   844 1758.895 MiB    0.000 MiB           4                   "converge_duration_mean": 0,
   845 1758.895 MiB    0.000 MiB           4                   "converge_duration_confidence_interval": [0, 0],
   846 1758.895 MiB    0.000 MiB           4                   "converge_probability": 0,
   847 1758.895 MiB    0.000 MiB           4                   "evenly_split_duration": 0,
   848                                                     }
   849
   850 1758.895 MiB    0.000 MiB          23           valid_num_iteration = sum(x is not None for x in best_alg_ind_list)
   851 1758.898 MiB    0.000 MiB          11           for i in range(num_iteration):
   852 1758.898 MiB    0.000 MiB          10               temp_performance = performance_list[i]
   853 1758.898 MiB    0.000 MiB          10               best_alg = best_alg_ind_list[i]
   854 1758.898 MiB    0.000 MiB          10               if best_alg is not None:
   855 1758.898 MiB    0.000 MiB          50                   for j in range(len(algorithm_list)):
   856 1758.898 MiB    0.000 MiB          40                       temp_alg_name = algorithm_list[j]
   857 1758.898 MiB    0.000 MiB          40                       temp_alg_id = ALGORITHM_ID_REFERENCE[temp_alg_name]
   858 1758.898 MiB    0.000 MiB          40                       temp_alg = temp_performance[j]
   859 1758.898 MiB    0.000 MiB          40                       temp_duration_CI = np.array(temp_alg[0])
   860 1758.898 MiB    0.000 MiB          40                       temp_duration_mean = temp_alg[1]
   861 1758.898 MiB    0.000 MiB          40                       temp_prob = temp_alg[2]
   862 1758.898 MiB    0.000 MiB          40                       temp_even_traffic = temp_alg[6]
   863 1758.898 MiB    0.000 MiB          80                       result_dict[temp_alg_id]["converge_duration_mean"] += (
   864 1758.898 MiB    0.000 MiB          40                           temp_duration_mean / valid_num_iteration
   865                                                             )
   866 1758.898 MiB    0.000 MiB         120                       result_dict[temp_alg_id][
   867 1758.898 MiB    0.000 MiB          40                           "converge_duration_confidence_interval"
   868 1758.898 MiB    0.000 MiB          40                       ] += (temp_duration_CI / valid_num_iteration)
   869 1758.898 MiB    0.000 MiB          80                       result_dict[temp_alg_id]["converge_probability"] += (
   870 1758.898 MiB    0.000 MiB          40                           temp_prob / valid_num_iteration
   871                                                             )
   872 1758.898 MiB    0.004 MiB          80                       result_dict[temp_alg_id]["evenly_split_duration"] += (
   873 1758.898 MiB    0.000 MiB          40                           temp_even_traffic / valid_num_iteration
   874                                                             )
   875
   876 1754.914 MiB   -3.984 MiB           1           logger.info("Finished running algorithm_pick()", extra={"results": result_dict})
   877 1754.914 MiB    0.000 MiB           1           return result_dict
   878                                             except Exception as e:
   879                                                 error = str(e)
   880                                                 logger.info("Unexpected error: " + error)
   881                                             logger.info("Failed to run algorithm_pick()")
   882                                             return False
```

On line 810 we see the major spike of the application, in the `single_algorithm_pick` function. This isn't surprising, as this is the core work of the DataSci library producing output for the Simulator. What is surprising is that it's using ~1800 MB of memory. Another profiling run with more aggressive `smallest_KPI_increase = 0.02` takes 1463 seconds and uses over 2000 MB of memory. We've been provisioning our k8s instances a bit (8x !!!) under that:

```
resources:
requests:
    cpu: 200m
    memory: 256Mi
limits:
    cpu: 600m
    memory: 512Mi
```

Although we know that our application won't be halted by Kubernetes for pegging the CPU for a prolonged period of time, this seems like a reason that we _would_ get stopped.

Thankfully, the Wayfair Kubernetes team has published [Guardrails](https://docs.csnzoo.com/docker/kubernetes/development-resources/cluster-guardrails/#memory) that we are well under, so there's plenty of headroom to claim more resources. Given the above profiling, and allowing for the fact that this may not be a worst-case resource usage, I'm suggesting that we increase to something in the neighborhood of:

```
resources:
requests:
    cpu: 1
    memory: 2Gi
limits:
    cpu: 1
    memory: 3Gi
```

I previously wrote an [ADR](https://github.csnzoo.com/shared/gambit-adr/blob/master/doc/architecture/decisions/0005-Task-Queue.md) that will take us down the path of queueing (awesome word, because 5 vowels in a row!) long-running tasks such as Simulations. This new research supports the idea that we should indeed queue Simulations, because trying to run several of them at once will actually pretty quickly push us past the Kubernetes guardrails. A task queue allows us to pick one Simulation off at a time, complete it in an "atomic" memory space, cleanup, then perform the next Simulation. These aren't intended to be things that users actively wait for, so adding some delay is acceptable.
