[<img src="https://upload.wikimedia.org/wikipedia/commons/d/de/AirflowLogo.png" align="right">](https://airflow.apache.org/)


This content I have created for my own learning. I have tried to put all jargons in simple word to make our understanding clear and precise. 
Please feel free to contribute any items that you think is missing or misleading.

## Contents
- [Introduction](#Introduction)
- [Airflow Architecture](#airflow-architecture)
- [Installing Airflow](#installing-airflow)
- [Fundamentals of Airflow](#fundamentals-of-airflow)
     - [Airflow's module structure](##airflow-module-strcutrure)
     - [Operators](##operators)
    
     
---
## Introduction
[Airflow](https://airflow.apache.org/) is a batch-oriented framework for creating data pipelines.

It uses [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) to create data processing networks or pipelines.

DAG stands for -> Direct Acyclic Graph. Meaning it flows in one direction. You can't come back to same point.

For Data prcoessing we can create our simplest DAG like this

`read-data-from-some-endpoint--> write-to-storage` 

where arrow `-->` represent dependencies which means on what basis next action will be triggered.

## Ok, so why should we use Airflow?

- If you like *`Everything As Code`* and **everything** mean everything including your configurations 😃
. EaC helps to create any complex level pipeline to solve the problem.
- If you like open source because mostly everything you can get as as inbuilt operator or executors.
- `Backfilling` features. It enables you to reprocess historical data.

## And, why shouldn't you use Airflow?
- If you want to build a streaming data pipeline. 

---
## Airflow Architecture

So, as of know we have atleast an idea that Airflow helps to create the data pipelines. Interanlly, Airflow installs below components to facilitate execution of pipelines. These componenets are 

- `Scheduler`, which parses DAGS, check their schedule interval, and starts scheduling DAGs tasks fir execution by pasing them to airflow workers.
- `Workers`, Responsible for doing real work. It picks up tasks and excute them.
- `Websever`, which presents a handy user interface to inspect, trigger and debug the behaviour of DAGs and tasks.
- `DAG directory`, to keep all dag in place to be read by scheduler and executor.
- `Metadata Database`, used by scheduler, executor, and webseerver to store state, so that all of them can communicate and take decisions.

![Airflow Architecture](/imgs/airflow_arch.png)

The heart of Airflow is arguably the scheduler, as this is where most of the magic happens that determines when and how your pipelines are executed. At a high level, the scheduler runs through the following steps.

1. Once users have written their workflows as DAGs, the files containing these DAGs are read by the scheduler to extract the corresponding tasks, dependencies, and schedule interval of each DAG.

2. For each DAG, the scheduler then checks whether the schedule interval for the DAG has passed since the last time it was read. If so, the tasks in the DAG are scheduled for execution.

3. For each scheduled task, the scheduler then checks whether the dependencies (= upstream tasks) of the task have been completed. If so, the task is added to the execution queue.

4. The scheduler waits for several moments before starting a new loop by jumping back to step 1.

For now, ite enough on architecture. Let's move to next part.

---
## Installing Airflow

Airflow provides many options for installations. You can read all the options in the [official airflow documentation](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html). and then decide which options suit your need. However, to keep it simple, I will go ahead with Docker

Installing Airflow with Docker is simple and intutive which helps us to understand typical features and working of Airflow. Below are the pre-requistes for running Airflow in Docker.
- Docker Community Edition installed in your machine. Check this link for [Windows](https://docs.docker.com/desktop/windows/) and [Mac](https://docs.docker.com/desktop/mac/). I followed this [blog](https://adamtheautomator.com/docker-for-mac/) for docker installation on Mac
- [Docker Compose](https://docs.docker.com/compose/install/) installation.

*Coveats* - You need atleast **4GB memory** for Docker engine.

![Docker memory setting](/imgs/docker_memory_settings.png)

### Installation Steps
1.  Create a file name as airflow_runner.sh. Copy below commands in the script. 

```
docker run --rm "debian:buster-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'

curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.2.4/docker-compose.yaml'

mkdir -p ./dags ./logs ./plugins

echo -e "AIRFLOW_UID=$(id -u)" > .env
```
2. Provide execute access to file. `chmod +x airflow_runner.sh`
3. Run `source airflow_runner.sh`
4. Once, the above steps get completed successfully, then run `docker-compose up airflow-init` to initialize database.

After initialization is complete, you should see a message like below.
```
airflow-init_1       | Upgrades done
airflow-init_1       | Admin user airflow created
airflow-init_1       | 2.3.0
start_airflow-init_1 exited with code 0
```

Now, we are ready to go for the next step.

---

## Starting Docker Airflow project

`docker-compose up`

After few seconds, when everything is up then the webserver is available at: http://localhost:8080. The default account has the login `airflow` and the password `airflow`.

## Cleaning up

To stop and delete containers, delete volumes with database data and download images, run:

`docker-compose down --volumes --rmi all`

---

## Fundamentals of Airflow 

We have comeup a long way and thanks for following till now. We have installed Airflow and know at high level what it stands for ;) but we are yet to discover how to build our pipeline. We will roughly touch few more concepts and then we will create a full fledged project using these concepts.

So, let's refresh our memory one more time. Airflow works on the principle of **`DAG`** and DAG is acyclic graph. 
We saw this example `read-data-from-some-endpoint --> write-to-storage` 

So, create an airflow DAG, we will write it like this

- Step 1
```python
dag = DAG(                                                     
   dag_id="my_first_dag",                          
   start_date=airflow.utils.dates.days_ago(2),                
   schedule_interval=None,                                     
)
```
- Step 2
 
 And now we need to create our two functions (I'm creating dummy functions) and will attach them to the Operator.
```python

def read_data_from_some_endpoint():
    pass

def write_to_storage():
    pass
 
```

- Step 3

Let's create our operator. We have python functions which need be attached to some Operator. 

```python

download_data = PythonOperator(. # This is our Airflow Operator.
    task_id="download_data", # unique name; it could be any name 
    python_callable=read_data_from_some_endpoint, # python function/callable
    dag = dag # Here we will attach our operator with the dag which we created at 1st step.
) 

persist_to_storage = PythonOperator(
    task_id = "persist_to_storage",  
    python_callable = write_to_storage,
    dag = dag
) 
 ```

- Step 4

Now, Lets create the execution order of our operators

```python

download_data >> persist_to_storage  # >> is bit shift operator in python which is overwritten in Airflow to indicate direction of task flow.

 ```

 That's it.  We have successfully created our first DAG.
 
But now you might be thinking from where we got `PythonOperator`, `DAG`, and how `>>` bit shift opertaor is working as task execution order. To understand it we will see the important `modules ` which is provided by Airflow.

-----


## Airflow's Module Structure

Airflow has standard module structure. It has all it's [important packages](https://airflow.apache.org/docs/apache-airflow/2.0.0/_modules/index.html) under airflow. Few of the important module structures are here

- `airflow` - For DAG and other base API.
- `airflow.executors` : For all inbuilt executors.
- `airflow.operators` : For all inbuilt operators.
- `airflow.models` : For DAG, log error, pool, xcom (cross communication) etc.
- `airflow.sensors` : Different sensors (in simple word it is either time interval, or file watcher to meet some criteria for task executions)
- `airflow.hooks` : Provides different module to connect external API services or databases.


So, we asked a question that from where we got our `Python Operators` and now when we know that above modules are there to provide these boilerplate code for us then where should we look for a `Python Operator` or any other Operators.
You guessed it right. It should be under `airflow.operators`. Similar way an `exeuctor` can be imported from `airflow.executors` and so on.

### and what if something which I'm interested is not present in any of the module?

If something is not there, then you can write your own custom opertor, sensor, executor, hook or anything which you want.
Airflow provides Base classes which anyone can inherit to write their own custom stuff. e.g. If I want to write a cutom operator, sensor, hook then this could be the template.

```python

from airflow.models import BaseOperator
from airflow.sensors.base import BaseSensorOperator
from airflow.hooks.base_hook import BaseHook
from airflow.utils.decorators import apply_defaults

class MyCustomOperator(BaseOperator):
    
    @apply_defaults # for default parameters from DAG
    def __init__(**kwargs):
        super(MyCustomOperator).__init__(**kwargs)
        pass
    def execute(self, conext): # we will cover more about context in next part.
        #your logic
        pass


class MyCustomSensor(BaseSensorOperator):
    
    @apply_defaults # for default parameters from DAG
    def __init__(**kwargs):
        super(MyCustomSensor).__init__(**kwargs)
        pass
    def poke(self, context): 
        #your logic
        pass

class MyCustomHook(BaseHook):
    
    @apply_defaults # for default parameters from DAG
    def __init__(**kwargs):
        super(MyCustomHook).__init__(**kwargs)
        pass
    def get_connection(self):
        #your logic
        pass

```

### How right shift operator (>>) defines task dependency?
The __ rshift __ method of the BaseOperator class implements the Python right shift logical operator in the context of setting a task or a DAG downstream of another.
See the implementation [here](https://github.com/apache/airflow/blob/5355909b5f4ef0366e38f21141db5c95baf443ad/airflow/models.py#L2569).

## Types of Operators

There are many operators which help us to map our code. Few of them are
- `PythonOperator`
- `BashOperator`
- `DummyOperator` - to show a dummy task
-  `DockerOperator` 
- `EmailOperator`

*and many more..* See the full [operators list](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html) in official documentation.


---

Satisfied principles (not listed are not applicable):

Load data incrementally : extracts only the newly created orders of the day before, not the whole table.

Process historic data : it’s possible to rerun the extract processes, but downstream DAGs have to be started manually.
Enforce the idempotency constraint : every DAG cleans out data if required and possible. Rerunning the same DAG multiple times has no undesirable side effects like duplication of the data.

Rest data between tasks : The data is in persistent storage before and after the operator.

Pool your resources : All task instances in the DAG use a pooled connection to the DWH by specifying the pool parameter.

Manage login details in one place : Connection settings are maintained in the Admin menu.

Develop your own workflow framework : A subdirectory in the DAG code repository contains a framework of operators that are reused between DAGs.

Sense when to start a task : The processing of dimensions and facts have external task sensors which wait until all processing of external DAGs have finished up to the required day.

Specify configuration details once : The place where SQL templates are is configured as an Airflow Variable and looked up as a global parameter when the DAG is instantiated.

## Cron based schedule

```
┌─────── minute (0 - 59)
│ ┌────── hour (0 - 23)
│ │ ┌───── day of the month (1 - 31)
│ │ │ ┌───── month (1 - 12)
│ │ │ │ ┌──── day of the week (0 - 6) (Sunday to Saturday;
│ │ │ │ │      7 is also Sunday on some systems)
* * * * *

```

check this website to generate cron expression - https://www.freeformatter.com/cron-expression-generator-quartz.html

# Best Practices for the task design

### 1- Atomicity 

`either all occur or nothing occurs.` So each task should do only one activity and if not the case then split the functionality into individual task.

### 2- Idempotency

`
Another important property to consider when writing Airflow tasks is idempotency. Tasks are said to be idempotent if calling the same task multiple times with the same inputs has no additional effect. This means that rerunning a task without changing the inputs should not change the overall output.
`

**for data load** : It can be make  idempotent by checking for existing results or making sure that previous results are overwritten by the task.

**for database load** : `upsert` can be used to overwrite  or update previous work done on the tables.


# Back Filling the previous task

The DAG class can be initiated with property `catchup`

if `catchup=False` ->  Airflow starts processing from the `current` interval.

if `catchup=True` -> This is default property and Airflow starts processing from the `past` interval.

# Templating tasks using the Airflow context

All operators load `context` a pre-loaded variable to supply most used variables during DAG run. A python examples can be shown here 

```python
from urllib import request
 
import airflow
from airflow import DAG
from airflow.operators.python import PythonOperator
 
dag = DAG(
    dag_id="stocksense",
    start_date=airflow.utils.dates.days_ago(1),
    schedule_interval="@hourly",
)
 
def _print_context(**context):
    """
    context here contains these preloaded items 
    to pass in dag during runtime.

    Airflow’s context dictionary can be found in the
    get_template_context method, in Airflow’s models.py.
    
    {
    'dag': task.dag,
    'ds': ds,
    'ds_nodash': ds_nodash,
    'ts': ts,
    'ts_nodash': ts_nodash,
    'yesterday_ds': yesterday_ds,
    'yesterday_ds_nodash': yesterday_ds_nodash,
    'tomorrow_ds': tomorrow_ds,
    'tomorrow_ds_nodash': tomorrow_ds_nodash,
    'END_DATE': ds,
    'end_date': ds,
    'dag_run': dag_run,
    'run_id': run_id,
    'execution_date': self.execution_date,
    'prev_execution_date': prev_execution_date,
    'next_execution_date': next_execution_date,
    'latest_date': ds,
    'macros': macros,
    'params': params,
    'tables': tables,
    'task': task,
    'task_instance': self,
    'ti': self,
    'task_instance_key_str': ti_key_str,
    'conf': configuration,
    'test_mode': self.test_mode,
    }
    """
   start = context["execution_date"]        
   end = context["next_execution_date"]
   print(f"Start: {start}, end: {end}")
 
 
print_context = PythonOperator(
   task_id="print_context", 
   python_callable=_print_context, 
   dag=dag
)
```

## Running docker images as a root

`docker run --rm -it -u root --entrypoint bash apache/airflow:2.2.4`


## Running docker images as a normal user

`docker exec -it <image_id> bash`

*sudo apt-get update && sudo apt-get install tk*


## Reference
- [Source code](https://github.com/apache/airflow/)
- [Documentation](https://airflow.apache.org/) (official website)
- [Confluence page](https://cwiki.apache.org/confluence/display/AIRFLOW/Airflow+Home)
- [![Twitter Follow](https://img.shields.io/twitter/follow/apacheairflow?style=social)](https://twitter.com/ApacheAirflow)
- [Slack workspace](https://apache-airflow-slack.herokuapp.com/)
