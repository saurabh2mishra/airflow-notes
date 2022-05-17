[<img src="https://upload.wikimedia.org/wikipedia/commons/d/de/AirflowLogo.png" align="right">](https://airflow.apache.org/)


This content I have created for my own learning. I have tried to put all jargons in simple word to make our understanding clear and precise. 
Please feel free to contribute any items that you think is missing or misleading.

## Contents
- [Introduction](#Introduction)
- [Airflow Architecture](#airflow-architecture)
- [Installing Airflow](#installing-airflow)
- [Fundamentals of Airflow](#fundamentals-of-airflow)
     - [Airflow's module structure](#airflow-module-strcutrure)
     - [Workloads](#workloads)
        - [Operators](#operators)
        - [Scheduler](#scheduler)
        - [Executors](#executors)
        - [hooks](#hooks)
        - [sensors](#sensors)
- [Best Practices](#best-practices)
- [Where to go from here?](#where-to-go-from-here)
     
---
## Introduction
[Airflow](https://airflow.apache.org/) is a batch-oriented framework for creating data pipelines.

It uses [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) to create data processing networks or pipelines.

- DAG stands for -> Direct Acyclic Graph. It flows in one direction. You can't come back to same point i.e. acyclic.

- In many data processing environments, a series of computations are run on the data to prepare it for one or more ultimate destinations. This type of data processing flow is often referred to as a data pipeline.
- A DAG or data processing flows can have multiple paths in the flow and that is also called branching.

A simplest DAG could be like this

`task-read-data-from-some-endpoint --> task-write-to-storage` 


where 
- `read-data-from-some-endpoint` & `write-to-storage`  - reprsent a task (unit of work)
- Arrow `-->` represents processing direction and depencies to check on what basis next action will be triggered.


### **Ok, so why should we use Airflow?**

- If you like *`Everything As Code`* and **everything** mean everything including your configurations 
. This helps to create any complex level pipeline to solve the problem.
- If you like open source because mostly everything you can get as as inbuilt operator or executors.
- `Backfilling` features. It enables you to reprocess historical data.

### **And, why shouldn't you use Airflow?**
- If you want to build a streaming data pipeline. 

---
## Airflow Architecture

So, as of know we have atleast an idea that Airflow is created to build the data pipelines. Below we can see the different componenets of Airflow and their internal connections.

![Airflow Architecture](/imgs/airflow_arch.png)

We can see above components when we do install Airflow and implicitly Airflow installs them to facilitate execution of pipelines. These componenets are 

- `Scheduler`, which parses DAGS, check their schedule interval, and starts scheduling DAGs tasks fir execution by pasing them to airflow workers.
- `Workers`, Responsible for doing real work. It picks up tasks and excute them.
- `Websever`, which presents a handy user interface to inspect, trigger and debug the behaviour of DAGs and tasks.
- `DAG directory`, to keep all dag in place to be read by scheduler and executor.
- `Metadata Database`, used by scheduler, executor, and webseerver to store state, so that all of them can communicate and take decisions.

The heart of Airflow is arguably the scheduler, as this is where most of the magic happens that determines when and how your pipelines are executed. At a high level, the scheduler runs through the following steps.

1. Once users have written their workflows as DAGs, the files containing these DAGs are read by the scheduler to extract the corresponding tasks, dependencies, and schedule interval of each DAG.

2. For each DAG, the scheduler then checks whether the schedule interval for the DAG has passed since the last time it was read. If so, the tasks in the DAG are scheduled for execution.

3. For each scheduled task, the scheduler then checks whether the dependencies (= upstream tasks) of the task have been completed. If so, the task is added to the execution queue.

4. The scheduler waits for several moments before starting a new loop by jumping back to step 1.

For now, ite enough on architecture. Let's move to next part.

---
## Installing Airflow

Airflow provides many options for installations. You can read all the options in the [official airflow documentation](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html), and then decide which option suits your need. However, to keep it simple and experimental, I will go ahead with Docker way of installation.

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

`docker-compose up --build`

Above command starts a docker environment and runbelow services as well
- `Webserver`
- `Scheduler`
- `Postgres database for metastore`

After few seconds, when everything is up then the webserver is available at: http://localhost:8080. The default account has the login `airflow` and the password `airflow`.

From terminal, you can also run `docker ps ` to check the processes which are up and running.

## Cleaning up

To stop and delete containers, delete volumes with database data and download images, run:

`docker-compose down --volumes --rmi all`

---

## Fundamentals of Airflow 

We have installed Airflow and know at high level what it stands for but we are yet to discover how to build our pipeline. We will roughly touch few more concepts and then we will create a full fledged project using these concepts.

So, let's refresh our memory one more time. Airflow works on the principle of **`DAG`** and DAG is acyclic graph. 
We saw this example `read-data-from-some-endpoint --> write-to-storage` 

So, create an airflow DAG, we will write it like this

- Step 1

Create a DAG. It accepts unique name, when to start it, and what could be the interval of running. There are many more parameters which it accepts but for now let's stick with these three.

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

Let's create our operators. We have python functions which need be attached to some Operator. The last argument it accept a DAG and here we need to tell operator that which dag it is going to consider.

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



## How to create a bit complex tasks flow?

Let's take this example.

![example_dag](/imgs/dag_example.png)

we see there are two color code have been used in above image.

light red  - shows branch flow (two or more flows) i.e. **branch_1, branch_2**

light green - normal task for different purpose. i.e. **false_1, false_2, true_2 etc.**

Now, without worrying about code, let's create the task flow to represneting above structure.

1- Lower workflow from **branch_1**

 `branch_1 >> true_1 >> join_1`

2- Upper workflow from **branch_1**

- upper flows has two section. First part goes till `branch_2`
    
`branch_1 >> false_1 >> branch_2`
- and then at branch_2, two parallel execution happens and goes till false_3
 
 `branch_2 >> false_2 >> join_2 >> false_3`

`branch_2 >> true_2 >> join_2 >> false_3`

Since false_2 and true_2 is happening in parallel, so we can merge them (put them in a list) in this way 

`branch_2 >>` **[true_2, false_2]** `>> join_2 >> false_3`

and finally we can merge above steps like this 

`branch_1 >> false_1 >> branch_2 >> [true_2, false_2] >> join_2 >> false_3 >> join_1`

**So we have got these two from step 1 and step 2**

`branch_1 >> true_1 >> join_1`
    

`branch_1 >> false_1 >> branch_2 >> [true_2, false_2] >> join_2 >> false_3 >> join_1`
 
and that's represent the execution of task or a DAG.

### **How bit shift operator (>> or <<) defines task dependency?**
The __ rshift __ and __ lshift __ methods of the BaseOperator class implements the Python right shift logical operator in the context of setting a task or a DAG downstream of another.
See the implementation [here](https://github.com/apache/airflow/blob/5355909b5f4ef0366e38f21141db5c95baf443ad/airflow/models.py#L2569).

So, **`bit shift`** been used as syntactic sugar for  `set_upstream` (<<) and `set_downstream` (>>) tasks.

For example 
`task1 >> task2` is same as `task2 << task1` is same as `task1.set_downstream(task2)` is same as  `task1.set_upstream(task2)`

**`bit shift`** plays crucial roles to build relationships among the tasks.

### **Effective Task Design**

The created task should follow

### 1- Atomicity 

Means `either all occur or nothing occurs.` So each task should do only one activity and if not the case then split the functionality into individual task.

### 2- Idempotency

`
An Airflow task is said to be idempotent if calling the same task multiple times with the same inputs has no additional effect. This means that rerunning a task without changing the inputs should not change the overall output.
`

**for data load** : It can be make  idempotent by checking for existing results or making sure that previous results are overwritten by the task.

**for database load** : `upsert` can be used to overwrite  or update previous work done on the tables.


### 3-  Back Filling the previous task

The DAG class can be initiated with property `catchup`

if `catchup=False` ->  Airflow starts processing from the `current` interval.

if `catchup=True` -> This is default property and Airflow starts processing from the `past` interval.

## Templating tasks using the Airflow context

All operators load `context` a pre-loaded variable to supply most used variables during DAG run. A python examples can be shown here 

```python
from urllib import request
 
import airflow
from airflow import DAG
from airflow.operators.python import PythonOperator
 
dag = DAG(
    dag_id="showconext",
    start_date=airflow.utils.dates.days_ago(1),
    schedule_interval="@hourly",
)
 
def _show_context(**context):
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
 
 
show_context = PythonOperator(
   task_id="show_context", 
   python_callable=_show_context, 
   dag=dag
)
```
 

But now you might be thinking from where we got `PythonOperator`, `DAG` etc. To understand it we will see the important `modules ` which is provided by Airflow.

-----


## Airflow's Module Structure

Airflow has standard module structure. It has all it's [important packages](https://airflow.apache.org/docs/apache-airflow/2.0.0/_modules/index.html) under airflow. Few of the important module structures are here

- `airflow` - For DAG and other base API.
- `airflow.executors` : For all inbuilt executors.
- `airflow.operators` : For all inbuilt operators.
- `airflow.models` : For DAG, log error, pool, xcom (cross communication) etc.
- `airflow.sensors` : Different sensors (in simple word it is either time interval, or file watcher to meet some criteria for task executions)
- `airflow.hooks` : Provides different module to connect external API services or databases.


So, we looking at above module, now we can easily determine that to get `PythonOperator` or any other Operator we need to import 
them from `airflow.operators`. Similar way an `exeuctor` can be imported from `airflow.executors` and so on.

Apart from that, many different packges providers including vendors and third party enhance the capabilty of Airflow. All provideres follow `apache-airflow-providers` nominclatures for the package build.
Providers can contain operators, hooks, sensor, and transfer operators to communicate with a multitude of external systems, but they can also extend Airflow core with new capabilities.

This is the list of providers - [providers list](https://airflow.apache.org/docs/#providers-packages-docs-apache-airflow-providers-index-html)

-----
# Workloads

## **`Operators`**

Operators are helpful to run your function or any executble program.


### `Types of Operators`

There are many operators which help us to map our code. Few of them are
- `PythonOperator` - To wrap a python callables/functions inside it.
- `BashOperator` - To call your bash script or command. Within BashOperator we can also call any executable program. 
- `DummyOperator` - to show a dummy task
- `DockerOperator` - To write and execute docker images.
- `EmailOperator` - To send an email (using SMTP configuration)

*and there many more operators do exits.* See the full [operators list](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html) in official documentation.

## **`Scheduler`**

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

## **`Executors`**

It hepls to run the task instance (task instances are function which we have wrapped under operator)


### `Types of Executors`

There are two types of executors

**`Local Executors`**

- [Debug Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/debug.html)
- [Local Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/local.html)
- [Sequential Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/sequential.html)

**`Remote Executors`**

- [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) - for scaling the workers. Helpful in parallelization.
- [CeleryKubernetes Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery_kubernetes.html)
- [Dask Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/dask.html)
- [Kubernetes Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/kubernetes.html)
- [LocalKubernetes Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/local_kubernetes.html)

## **`Hooks`**

A high level interface to establish a connection with databases or any other external services.

[List of different avaible hooks](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/hooks/index.html?highlight=hooks#module-airflow.hooks)

## **`Sensors`**

A special types of operators whose purpose is to wait for an event to happen to start the execution.
For instance, 
    
- `ExternalTaskSensor` waits on another task (in a different DAG) to complete execution.
- `S3KeySensor` S3 Key sensors are used to wait for a specific file or directory to be available on an S3 bucket.


## *What if something which I'm interested is not present in any of the module?*

You didn't found right operator, executors, sensors or hooks? No worries, you can write your own custom stuffs.
Airflow provides Base classes which we can inherit to write our own custom classes.

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

# Best Practices

 - Write clean DAG and stick with one principle to create your DAG. Generally, in two way we create our DAG
    - with context manager 
    - without context manager
- Keep computation code and DAG definition separate. Everytime DAG loads it recompute, hence more time it took to load.
- Don't hardocde or leave your senstive connection information in the code. Manage it at central level in secure way.
- Create the tag and use it for quick look to groups the tasks in monitoring.
- Always search for existing inbuilt airflow operators, hooks or sensors before creating your own cutom stuff.
- Data Quality and Testing is often gets overlooked. So make sure you use a standard for your code base.
- Follow load strategies - incremental, scd types in your code to unnecessary data load.
- If possible, create a framework for DAG generations. A meta wrapper. Checkout this [repo](https://github.com/ajbosco/dag-factory).
- Specify configuration details once : The place where SQL templates are is configured as an Airflow Variable and looked up as a global parameter when the DAG is instantiated.
- Pool your resources : All task instances in the DAG use a pooled connection to the DWH by specifying the pool parameter. 
- Manage login details in one place : Connection settings are maintained in the Admin menu.
- Sense when to start a task : The processing of dimensions and facts have external task sensors which wait until all processing of external DAGs have finished up to the required day.


check this website to generate cron expression - https://www.freeformatter.com/cron-expression-generator-quartz.html


## Running docker images as a root

`docker run --rm -it -u root --entrypoint bash apache/airflow:2.2.4`


## Running docker images as a normal user

`docker exec -it <image_id> bash`

*sudo apt-get update && sudo apt-get install tk*


## Reference
- [Data Pipelines with Apache Airflow ](https://www.manning.com/books/data-pipelines-with-apache-airflow)(Highly recommended)
- [Source code](https://github.com/apache/airflow/)
- [Documentation](https://airflow.apache.org/) (official website)
- [Confluence page](https://cwiki.apache.org/confluence/display/AIRFLOW/Airflow+Home)
- [![Twitter Follow](https://img.shields.io/twitter/follow/apacheairflow?style=social)](https://twitter.com/ApacheAirflow)
- [Slack workspace](https://apache-airflow-slack.herokuapp.com/)
