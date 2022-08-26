# HPOaaS REST API
The HPOaaS REST API design is as follows

##  Start an Experiment
Start a new Experiment with HPOaaS. This requires a valid Search Space JSON to be passed in.

```
'POST /experiment_trials'
`Content-Type: application/json`

curl -H 'Content-Type: application/json' http://<URL>:<PORT>/experiment_trials -d 
'{
    "operation": "EXP_TRIAL_GENERATE_NEW",
    "search_space": '--See example search space JSON below--'
}'

Response:
Status code   Response body
200            trial_number
400            Corresponding error message for Bad request
404            Resource not found    
```

## Get a Trial JSON object
Get a Trial Configuration JSON filled with values for each tunable that is part of the Search Space for a given trial number.
```
'GET /experiment_trials?experiment_name={name}&trial_number={trial-number}'

curl -H 'Accept: application/json' 'http://<URL>:<PORT>/experiment_trials?experiment_name=name&trial_number=0'

Example Response:
[
    {
        "tunable_name": "cpu_request",
        “tunable_value”: 3.47
    },
    {
        "tunable_name": "memory_request",
        “tunable_value”: 728
    }
]

Response:
Status code   Response body
200            trial_configs
400            Corresponding error message for Bad request
404            Experiment/Resource not found   
```
## Send the Result of a Trial to HPOaaS.
Send the result obtained by running a trial with the previously provided Trial Configuration back to HPOaaS.

```
'POST /experiment_trials'
'Content-Type: application/json'

curl -H 'Content-Type: application/json' http://<URL>:<PORT>/experiment_trials -d 
‘{
    "experiment_name" : "name",
    "operation" : "EXP_TRIAL_RESULT",
    "trial_number": xyz,
    "trial_result": "success | failure | error",
    "result_value_type": "double",
    "result_value": abc
}’

success : The experiment trial runs successfully without any error.
failure : The experiment trial fails due to reason such as invalid tunable value in the search_space. 
          Trial will be skipped and experiment continues with the next trial. 
error : The experiment terminates due to reasons such as network error. 
   
Response:
Status code   Response body
200            Result Status
400            Corresponding error message for Bad request
404            Experiment/Resource not found 
```

## Continue the Experiment
Continue a previously started experiment and get the Next Trial Number.

```
'POST /experiment_trials'
'Content-Type: application/json'

curl -H 'Content-Type: application/json' http://<URL>:<PORT>/experiment_trials -d 
'{
    "operation": EXP_TRIAL_GENERATE_SUBSEQUENT",
    "experiment_name" : "name"
}'

Response:
Status code   Response body
200            trial_number
400            Corresponding error message for Bad request
404            Experiment/Resource not found
```

## Search Space JSON
Here is an example Search Space JSON
```
{
  "experiment_name": "petclinic-sample-2-75884c5549-npvgd",
  "total_trials": 100,
  "parallel_trials": 1,
  "hpo_algo_impl": "optuna_tpe",
  "objective_function": "transaction_response_time",
  "value_type": "double",
  "direction": "minimize",
  "tunables": [
    {
      "value_type": "double",
      "lower_bound": 150,
      "name": "memoryRequest",
      "upper_bound": 300,
      "step": 1
    },
    {
      "value_type": "double",
      "lower_bound": 1.0,
      "name": "cpuRequest",
      "upper_bound": 3.0,
      "step": 0.01
    }
  ]
}
```

## Stop experiment
Stop a running experiment before the experiment has finished

```
'POST /experiment_trials'
'Content-Type: application/json'

curl -H 'Content-Type: application/json' http://<URL>:<PORT>/experiment_trials -d 
'{
    "operation": EXP_STOP",
    "experiment_name" : "name"
}'

Response:
Status code   Response body
200            Result Status
400            Corresponding error message for Bad request
404            Experiment/Resource not found 
```

##  Health
Get the status of HPO.

```
Request
`GET /health`

`curl -H 'Accept: application/json' http://<URL>:<PORT>/health`

Response:
Status code   Response body
200            OK
503            Service Unavailable
```

## Plots
Generate various plots after an experiment is complete.
```
'GET /plot?experiment_name={name}&type={plot_type}'

curl -o tunable_importance.html 'http://<URL>:<PORT>/plot?experiment_name=name&type=tunable_importance'

Response:
Status code   Response body
200            html file containing the plot for the given type
400            Corresponding error message for Bad request
404            Experiment/Resource not found   

Supported plot type:
type                        Description
tunable_importance          Plot importance of all tunables
optimization_history        Plot optimization history of all trials
parallel_coordinate         Plot the high-dimensional tunable relationships
slice                       Plot the tunable relationship as slice
```
Note: In cases of a single trial experiment and no variance in objective function value, tunable_importance plot doesn't generate.