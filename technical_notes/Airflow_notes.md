---
date: 2025-02-09
---
# **🔹 Airflow: Custom Operators, Dynamic DAGs & Testing**

## **📌 Custom Airflow Operator**
Learn how to create a custom Airflow operator that transfers data from MySQL to PostgreSQL.  
🔗 [Read More](https://medium.com/data-folks-indonesia/airflow-create-custom-operator-from-mysql-to-postgresql-a69d95a55c03)

---

## **🔹 Dynamic DAGs in Airflow**
### **1️⃣ Using Nested Operators**
Achieve a dynamic workflow in Airflow using nested operators.  
🔗 [Read More](https://medium.com/@jw_ng/using-nested-operators-to-achieve-a-dynamic-airflow-workflow-ab9d14e136c1)

### **2️⃣ Advanced Airflow Task Dependencies**
Explore techniques for managing dependencies across DAGs using sensors and triggers.  
🔗 [Read More](https://blog.damavis.com/en/advanced-airflow-cross-dag-task-and-sensor-dependencies/)

---

## **🔹 Testing Airflow DAGs**
Best practices for writing testable Airflow DAGs, including mocking dependencies and unit testing.  
🔗 [Read More](https://medium.com/limejump-tech-blog/how-we-test-our-airflow-code-at-limejump-46492fdc95ac)

---

## **🔹 Custom Operator Hints**
- Printing unique messages in an Airflow Operator:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/52144108/how-to-print-a-unique-message-in-airflow-operator)
- Dynamic inheritance in Python using decorators:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/42862493/dynamic-inheritance-in-python-through-a-decorator)
- Understanding Python decorators for dynamic customization:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/5764922/python-dynamic-decorators-why-so-many-wraps)

---

## **🔹 Useful Resources**
### **1️⃣ Dynamically Creating Classes in Python**
- Dynamically create derived classes from a base class:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/15247075/how-can-i-dynamically-create-derived-classes-from-a-base-class)
- Python dynamic class creation techniques:  
  🔗 [Geeks for Geeks](https://www.geeksforgeeks.org/create-classes-dynamically-in-python/)

### **2️⃣ Airflow Metadata & Execution Tracking**
- Using Airflow Cluster Policies and Task Callbacks to track metadata:  
  🔗 [Read More](https://medium.com/apache-airflow/how-to-track-metadata-with-airflow-cluster-policies-and-task-callbacks-f80d42db9895)
- Running a block of code at the start of every task in Airflow:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/62216429/is-there-any-way-to-specify-a-block-of-code-that-should-run-at-the-start-of-ever)
- Printing unique messages inside an Airflow Operator:  
  🔗 [Stack Overflow](https://stackoverflow.com/questions/52144108/how-to-print-a-unique-message-in-airflow-operator)

---
# **🔹 Must-Read: Airflow & XCom for Inter-Task Communication**
Understand how XCom enables communication between tasks in Airflow with real-world use cases.  
🔗 [Read More](https://precocityllc.com/blog/airflow-and-xcom-inter-task-communication-use-cases/)

---

# **🔹 Airflow Use Cases**
### **1️⃣ Adding a Form Interface for DAGs**
Learn how to create an input form for DAGs in Airflow, making workflows more interactive.  
🔗 [Read More](https://medium.com/maisonsdumonde/road-to-add-form-for-airflows-dag-1dcf2e7583ef)

---
# **🔹 Airflow Blog: Triggering a DAG from Another DAG**
Learn how to trigger an Airflow DAG from within another DAG with best practices and examples.  
🔗 [Read More](https://www.mikulskibartosz.name/trigger-an-airflow-dag-from-another-dag/)

---

# **🔹 To-Do in Airflow**
### **📌 Building a Gmail Data Pipeline in Airflow**
Step-by-step guide on setting up a Gmail data pipeline using Apache Airflow.  
🔗 [Read More](https://towardsdatascience.com/data-engineering-how-to-build-a-gmail-data-pipeline-on-apache-airflow-ce2cfd1f9282)

---

# **🔹 SLA Management in Airflow**
Learn how to implement and monitor Service Level Agreements (SLAs) for your Airflow DAGs.  
🔗 [Read More](https://www.cloudwalker.io/2020/12/15/airflow-sla-management/)

---
# **🔹 Airflow Basics**

## **🛠️ Components of Airflow**
- **Webserver**  
- **Scheduler**  
  - The scheduler doesn’t run tasks directly but hands them over to the Executor.  
  - In a default Airflow installation, it runs everything inside the scheduler, but production-ready executors push task execution to workers.  

- **Executors**  
  - Executors handle resource utilization and efficiently distribute work.  
  - Example: If a DAG has six tasks, the Airflow scheduler assigns each task separately to the Executor. Whether tasks run in parallel or sequentially depends on the executor type.  

- **Queue**  
  - Executors allocate resources and place tasks into the queue.  
  - When a worker becomes available, it picks up tasks from the queue for execution.  

- **Workers**  
  - Workers are nodes/processors responsible for running tasks.  

---

## **🔄 Life Cycle of a Task (Scheduler to Executor)**
1. Before execution, executor resources remain idle or unavailable.  
2. The scheduler signals the executor when the scheduled time arrives.  
3. The executor allocates resources, queues tasks, and waits for available workers.  
4. The scheduler continuously monitors tasks via heartbeats and updates the backend database.  
5. When tasks complete, the executor releases allocated resources.  

---

## **⚙️ Different Types of Executors**
### **1️⃣ SequentialExecutor**
- Runs tasks sequentially, even with a branching operator.  
- Example: Tasks execute in the order `'branch_a' → 'branch_b' → 'branch_c' → 'branch_d'`.  

### **2️⃣ LocalExecutor**
- Runs tasks on the same node as the Airflow scheduler but on different processors.  
- Supports multi-processing.  
- Requires **MySQL/PostgreSQL** for metadata storage.  

### **3️⃣ CeleryExecutor**
- Distributed execution using Celery.  
- Requires **Celery and a backend (Redis/RabbitMQ)** for task management.  
- Preferred for scalability and widely used in production environments.  

### **4️⃣ DebugExecutor**
- Introduced in **Airflow 1.10.8** for debugging within IDEs.  
- Works similarly to SequentialExecutor.  

---

## **📌 When to Use Which Executor?**
### **🏢 Production**
- **CeleryExecutor** or **KubernetesExecutor** for scalable environments.  
- **LocalExecutor** is suitable if the workload is light or primarily runs in cloud services like AWS/Azure.  

### **🛠️ Development/Testing**
- **SequentialExecutor** or **DebugExecutor** are preferred.  

---
# **🔹 Key Features of Apache Airflow & When to Use Them**

## **📌 DAGs (Directed Acyclic Graphs)**
- Defines workflows in Airflow.  
- DAGs contain **Tasks** that are executed in a defined order.  

---

## **📌 Tasks**
### **🔹 Task Instances**
- Similar to how a **DAG Run** instantiates a DAG, tasks are instantiated as **Task Instances** for a given execution date.  

### **🔹 Relationship Terminology**
- **Upstream & Downstream** → Execution date remains the same.  
- **Previous & Next** → Different execution dates for the same task.  

### **🔹 Timeout vs. SLA**
| Feature  | Purpose  | Behavior  |
|----------|---------|----------|
| **Timeout** | Sets a max runtime for a task. | If the timeout is reached, the task **fails** with a timeout exception. |
| **SLA (Service Level Agreement)** | Monitors task runtime. | Sends a **notification** if runtime exceeds SLA but does not fail the task. |

---

## **📌 Types of Tasks**
1️⃣ **Operators** → Predefined task templates in Airflow.  
- When an operator is instantiated, it represents a **task** in the DAG.  
- Example: **BashOperator, PythonOperator, PostgresOperator**.  

2️⃣ **Control Flow Modifications**  
- Tasks run only when all **upstream (parent) tasks** succeed.  
- **Control flow** can be modified using branching and dependencies.  

---

## **🔄 Task Communication (XComs)**
**Reference:** [Airflow XCom](https://marclamberti.com/blog/airflow-xcom/#:~:text=XCom%20stands%20for%20%E2%80%9Ccross-communication%E2%80%9D%20and%20allows%20to%20exchange,The%20key%20is%20the%20identifier%20of%20your%20XCom.)

- XCom = "**Cross-Communication**" between tasks.  
- **Local to a DAG** (i.e., cannot be shared across DAGs).  
- Used to pass data between tasks.  

🔹 **Disabling XCom in Operators:**  
Some operators (e.g., **BashOperator**) automatically push outputs to XCom. To disable this:  
```python
downloading_data = BashOperator(
    task_id='downloading_data',
    bash_command='sleep 3',
    do_xcom_push=False
)
```
## Pushing  add to XCOM 
```python
def _training_model(**kwargs):
    ti = kwargs['ti']
    ti.xcom_push(key='model_accuracy', value=accuracy)
```
## Pulling data from XCOM 
```python
def _choose_best_model(**kwargs):
    fetched_accuracies = kwargs['ti'].xcom_pull(
        key='model_accuracy',
        task_ids=['training_model_A', 'training_model_B', 'training_model_C']
    )
```
## **🔄 XCom Limitations**
- ❌ **Cannot store large data** (e.g., Pandas DataFrames).  
- ✅ Used only for **small values** (like strings, numbers).  

---

## **📌 Variables & Hooks**
### **🔹 Variables**
- **Global in scope** (accessible across multiple DAGs).  

### **🔹 Hooks**
- Interfaces for **external services** (e.g., PostgreSQL, MySQL, S3).  

---

## **📌 Connections**
- Stores **login credentials** for external services.  
- Returns a **connection ID** that can be referenced in DAGs.  

---

## **📌 Types of Operators**
| **Operator Type**        | **Purpose**                   | **Example**             |
|-------------------------|-----------------------------|-------------------------|
| **Transfer Operators**   | Move data between sources.  | `MySQLToS3Operator`    |
| **Sensor Operators**     | Wait for external conditions. | `S3Sensor`             |
| **Action Operators**     | Execute specific actions.   | `BashOperator`, `PythonOperator` |

---

## **📌 Plugins**
- **Lazy-loaded** extensions that add custom functionality.  





# Overriding the Airflow operators 
```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator as _DummyOperator
from airflow.operators.python import PythonOperator as _PythonOperator

from airflow.models.baseoperator import BaseOperator as _bp
from datetime import datetime, timedelta
#from airflow.operators.util.AuditMixin import PrintMixin


class PrintMixin:
    def execute(self, context):
        self.logger.info('Inside task {task_id}'.format(task_id=context['task_id']))
        super().execute(context)

    def pre_executor(self,context):
        print("Operator is {0}".format(str(context)))
        print("Dictionary is {0}".format(str(self.__dict__)))

'''class LogDummyOperator(_DummyOperator):
    pass'''
class LogDummyOperator(_bp,PrintMixin):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

    def execute(self, context):
        #pass
        self.pre_executor(context)
        #'''
class DummyOperator(PrintMixin,_DummyOperator):
    def execute(self, context):
        #self.logger.info('Inside task {task_id}'.format(task_id=context['task_id']))
        self.pre_executor(context)
        #super().execute(context)

print(DummyOperator.__dict__)
class PythonOperator(_PythonOperator, PrintMixin):
    def execute(self, context):
        #self.logger.info('Inside task {task_id}'.format(task_id=context['task_id']))
        self.pre_executor(context)
        super().execute(context)

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2021, 8, 29, 16, 00),
    'max_active_runs': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG('inher_source_a',
          schedule_interval=None,
          default_args=default_args,
          catchup=False,
          tags=["dev_dag"]
          )


def my_processing_func(**kwargs):
    print("*************Completed Ingestion *****************")


t1 = DummyOperator(
    task_id='Start',
    dag=dag
)

t2 = LogDummyOperator(
    task_id='End',
    dag=dag
)



ingestion = PythonOperator(
    task_id='Ingestion_to_Target_A',
    python_callable=my_processing_func,
    dag=dag)



t1 >> ingestion >> t2
```
---
## **🚀 Scaling Airflow**
🔗 **Reference:** [Scaling Workers in Airflow](https://www.astronomer.io/guides/airflow-scaling-workers)  

---

## **📌 Airflow Pools**
🔗 **Reference:** [Understanding Airflow Pools](https://guptakumartanuj.wordpress.com/2020/05/09/understanding-of-airflow-pools/)  

### **🔹 What are Airflow Pools?**
- Configurable via the **Airflow UI**.  
- Used to **limit parallelism** for a specific set of tasks.  

### **🔹 Why Use Airflow Pools?**
✅ **Prioritize Tasks** – Assign priority to certain tasks over others.  
✅ **Control Execution Limits** – Avoid overwhelming third-party APIs with rate limits.  

## **📌 DAG (Directed Acyclic Graph)**
### **🔹 Key Characteristics**
- A DAG **does not manage** what happens inside tasks.
- It is responsible for:
  - **Execution order** of tasks.
  - **Retries** in case of failures.
  - **Timeouts** and overall task lifecycle.

---

### **🔹 Declaring a DAG**
#### **1️⃣ Using a Context Manager**
```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator

with DAG("my_dag_name") as dag:
    op = DummyOperator(task_id="task")
```
#### **2 Using a Standard Constructor**
```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator

my_dag = DAG("my_dag_name")
op = DummyOperator(task_id="task", dag=my_dag)
```
## **📌 Task Dependencies in Airflow**
### **🔹 Standard Task Dependency**
```python
first_task >> [second_task, third_task]
# OR
first_task.set_downstream(second_task, third_task)
```
### **🔹 Chain Dependencies**
```python
from airflow.models.baseoperator import chain

# Equivalent to: op1 >> op2 >> op3 >> op4
chain(op1, op2, op3, op4)
```
### **🔹 Dynamically Creating Dependencies**
```python
from airflow.operators.dummy import DummyOperator

chain([DummyOperator(task_id='op' + str(i)) for i in range(1, 6)])
```
---
## **📌 Loading DAGs**
- Airflow loads DAGs from Python files by detecting objects at the top level that are DAG instances.

### **🔹 Example**
```python
dag_1 = DAG('this_dag_will_be_discovered')

def my_function():
    dag_2 = DAG('but_this_dag_will_not')

my_function()
```
**Note: dag_2 will not be loaded because it is inside a function.**
## Running DAGs
DAGs run in two ways:
1. Manually triggered (via UI or API)
2. Scheduled execution (defined within the DAG)
```python 
with DAG("my_daily_dag", schedule_interval="@daily"):
```
## **📌 Default Arguments**
- By default, a Task in a DAG runs **only when all upstream tasks are successful**.
- This behavior can be modified using different **Trigger Rules**.

---

## **📌 Control Flow in DAGs**
### **🔹 Branching**
- Branching allows a DAG to **dynamically choose execution paths**.
- **`BranchPythonOperator`** returns a `task_id` or a list of `task_ids` to determine which path should execute.

---

### **🔹 Trigger Rules**
Use the `trigger_rule` argument in a task to define execution conditions. There are **9 trigger rules**, including:

| **Trigger Rule**             | **Behavior** |
|------------------------------|-------------|
| `all_success`                | Runs if **all** upstream tasks succeed. |
| `all_failure`                | Runs if **all** upstream tasks fail. |
| `all_done`                   | Runs whether upstream tasks **fail or succeed**. |
| `one_failed`                 | Runs if **at least one** upstream task fails. *(e.g., Send alert email if one task fails)* |
| `none_failed`                | Runs if **none of the upstream tasks fail** (even if skipped). |
| `none_failed_or_skipped`     | Runs if **no upstream task failed**, and at least one succeeded. |

📌 **Best Practice:**  
🚫 **Never use `all_success` or `all_failed` downstream of a Branching operation!**

---

### **🔹 Latest Only**
- Uses the **`LatestOnlyOperator`** to execute tasks **only in the most recent DAG run**.
### Depends on Past
Ensures a task runs only if the same task in the previous DAG run succeeded.
```python
task = PythonOperator(
    task_id="my_task",
    python_callable=my_function,
    depends_on_past=True
)
```
## **📌 Task Groups**
- **Task Groups** visually organize tasks into **hierarchical groups** in **Graph View**.
- They are **only for UI grouping** and **do not affect execution order**.

---

## **📌 SubDAGs**
Use **SubDAGs** when:
- A set of tasks is **reused** in multiple DAGs.
- You want to **group many tasks into a single logical unit**.

📌 **Created using the `SubDagOperator`**.
```python
from airflow.operators.subdag import SubDagOperator

subdag = SubDagOperator(
    task_id="my_subdag",
    subdag=create_subdag_function()
)
```
## **📌 Parameters**
### **🔹 provide_context**
- Used along with **operators**.
- When set to `True`, it allows passing **Airflow Context variables** to tasks.
## **📌 Things Needed for Creating a DAG**
### **🔹 Essential Components**
- **default_args**: Define properties common to all tasks in the DAG.
- **DAG Definition**: 
  ```python
  DAG('dag_id', schedule)
  ```
## **📌 Connecting to JDBC Sources**
- **Create a Connection ID** in the Airflow **Web UI**.
- Use the **Connection ID** in the operator.

---

## **📌 Sensors**
### **🔹 HttpSensor**
- Specify the **Connection ID** in the operator.
- Add the **URL information** under the connection.
- Define the **endpoint** (REST API endpoint).

---

## **📌 Example DAG Flow**
1️⃣ **Create Table** ➝ 2️⃣ **Check API Availability** ➝ 3️⃣ **Extract User** ➝ 4️⃣ **Process User** ➝ 5️⃣ **Store User**

---

## **📌 Scheduling a DAG**
### **🔹 Key Parameters:**
- `start_date`
- `schedule_interval`

### **🔹 Example:**
```python
schedule_interval = "10 minutes"
```
If **start_date = 10 AM**, the DAG will trigger at **10:10 AM**.  
**Execution Date** = start_date (10 AM).  

---

## **📌 Backfilling and Catchup**
- A **non-triggered DAG** executes from the **starting execution date**, **NOT** the `start_date`.
- To **avoid running old DAG instances**, set: `catchup = False`
- ### **🔹 Catchup Configuration:**
	- **DAG Level**: Use `catchup=False` in the DAG definition.
	- **Scheduler Level**: Modify in `airflow.cfg` file.
# **📌 Sensors in Apache Airflow**  
**Reference:** [Marc Lamberti's Blog](https://marclamberti.com/blog/airflow-sensors/)  

A **Sensor** is an operator that checks at a given time interval if a condition is met.  
- If **true**, it **succeeds**.  
- If **false**, it **retries** until it times out.  

## **🔹 Use Cases**
Sensors help trigger a DAG based on conditions such as:  
1️⃣ Waiting for a file.  
2️⃣ Checking if an SQL entry exists.  
3️⃣ Delaying DAG execution.  

---

## **🔹 Types of Sensors**
| **Sensor Type**            | **Description** |
|----------------------------|----------------|
| **FileSensor**             | Waits for a file/folder to appear. |
| **S3KeySensor**           | Waits for a key in an S3 bucket. |
| **SqlSensor**             | Runs an SQL query until a condition is met. |
| **HivePartitionSensor**   | Waits for a partition in Hive. |
| **ExternalTaskSensor**    | Waits for another DAG/task to complete. |
| **DateTimeSensor**       | Waits until a specific datetime. |
| **TimeDeltaSensor**       | Waits for a timedelta after execution_date + schedule_interval. |

---

## **🔹 Key Parameters**
### **🔸 `poke_interval`**
- Defines how often the sensor checks for a condition (e.g., every **30 seconds**).  
- Considerations:  
  - Frequent checks may create **new connections**, leading to **network latency**.

### **🔸 `timeout`**
- Maximum wait time before the sensor fails (**default: 7 days**).  
- Prevents **deadlocks** (e.g., if a file never arrives, the sensor won’t hold up a task indefinitely).  
- Example:  `timeout = 60 * 30 # Waits for 30 minutes`

### **🔸 `reschedule`**
- Two sensor modes:
1️⃣ **Poke Mode**:  
    - The sensor **holds a slot** and **sleeps** between pokes.  
    - The task remains active until completion.  
2️⃣ **Reschedule Mode**:  
    - The sensor **releases the slot** if the condition isn’t met.  
    - The task is marked **"up_for_reschedule"** until the next check.  
    - Other tasks can execute while waiting.

---

## **🔹 Handling Failures**
### **🔸 `on_failure_callback`**
- Defines how to handle sensor **timeouts**.  
- The **context object** contains useful DAG/task metadata.  

### **🔸 `soft_fail`**
- Determines if a **timeout** should be marked as **failed** or **skipped**.  
- Example use case:
- If a DAG waits for multiple files and **one file is missing**, the DAG should **continue processing the others**.  
- `soft_fail=True` marks the sensor as **skipped** instead of failed.
- **Note:**  
- **Skipped tasks** will **skip** their downstream tasks **unless** their trigger rule is modified.  

### **🔸 `exponential_backoff`**
- Allows for **progressively longer wait times** between retries using an **exponential backoff algorithm**.

---

## **🔹 How `poke_interval`, `retry`, and `timeout` Work**
**Reference:** [Cloudwalker Blog](https://www.cloudwalker.io/2021/07/24/apache-airflow-sensors/)  

Example settings:  
- `retries = 2`  
- `poke_interval = 60 seconds`  
- `timeout = 180 seconds`  

👉 Airflow **retries the sensor task twice**.  
👉 In each attempt, it **pokes** at:  
 **0s → 60s → 120s → 180s**  

---

## **🔹 ExternalTaskSensor**
**Use case:**  
- Triggers a task in another DAG.  

**Note:**  
- Both DAGs **must have the same start date** (but can have different schedules).  

### **🔸 `execution_delta`**
- Defines the **time difference** between DAG runs.  
- Example:  
- **Master DAG** triggers at **8:48 PM**.  
- **Slave DAG** triggers at **8:46 PM**.  
- If `execution_delta = timedelta(2)`, the **Master DAG checks if the Slave DAG completed at 8:46 PM**.  

🔗 **Example:** [Medium Blog on ExternalTaskSensor](https://medium.com/@fninsiima/sensing-the-completion-of-external-airflow-tasks-827344d03142)  
# **📌 TriggerDagRunOperator**
### **🔹 Overview**
- Introduced in **Airflow 2.0**, this operator can:
  1️⃣ **Trigger an external DAG**.  
  2️⃣ **Wait for its completion** before proceeding to the next step.  

- **Before Airflow 2.0**, this behavior was achieved using:
  - `TriggerDagRunOperator` **+** `ExternalTaskSensor`.  

### **🔹 Key Parameters**
- `wait_for_completion` → Waits for the external DAG to complete.  
- `poke_interval` → Defines how often to check for completion.  
- `failed_states` → Specifies which states indicate a failure.  

---

# **📌 BranchPythonOperator**
### **🔹 Overview**
- Allows **dynamic execution paths** based on a Python function’s return value.  
- Returns **one or more `task_id`s**, skipping the others.  

### **🔹 Best Practices**
1️⃣ **Avoid excessive XComs:**  
   - By default, `BranchPythonOperator` **creates many XComs**.  
   - Disable them using `do_xcom_push=False`.  

2️⃣ **Handling skipped tasks:**  
   - Any task **following** a `BranchPythonOperator` is **skipped** if not selected.  
   - Airflow expects **all parent tasks to succeed**.  
   - To prevent issues, set:  
     ```python
     trigger_rule="none_failed_or_skipped"
     ```
# **📌 SubDAGs**
### **🔹 Problems with SubDAGs**
- ❌ **Can create deadlocks**.  
- ❌ **Not recommended for production**.  
- ❌ **Uses its own executor** (default: **SequentialExecutor**), limiting parallelism.  

### **✅ Best Practice: Use Task Groups Instead**
- **Task Groups** provide **UI-based grouping** without execution limitations.  

### **🔹 Example: Using `TaskGroup`**
```python
from airflow.utils.task_group import TaskGroup
from airflow.operators.bash import BashOperator

with TaskGroup("task_group_id") as processing_task:
    task1 = BashOperator(task_id="task1", bash_command="echo 'Task 1'")

dag_level_task >> processing_task
```
# 📌 Airflow Commands

```
# 🔹 List all DAGs
airflow list_dags

# 🔹 Trigger a DAG with a specific execution date
airflow trigger_dag -e 2021-01-01 xcom_dag

# 🔹 Test a specific task in a DAG without affecting production
# Usage: airflow test <dag_id> <task_id> <some_past_date>
airflow test my_dag task_id 2021-01-01

# 🔹 Test an entire DAG run for a past date
airflow dags test my_dag 2021-01-01
```
# 📌 Miscellaneous

## 🔹 Airflow Ignore Files
- Airflow uses `.airflowignore` to ignore specified files and directories.
- This file should be placed inside the DAG folder.
- Any file or directory listed in `.airflowignore` will be ignored by Airflow.

## 🔹 Zombie Tasks
- Airflow automatically kills tasks if the worker does not respond within the scheduler heartbeat interval.
# 📌 Gotchas

1️⃣ **Retries Configuration**  
- By default, retry settings are configured at the DAG level.  
- However, retries can be configured at the task level by passing the `retries` parameter in the task definition.  

🔗 **Reference:** [Different Retry Delay for Every Airflow Task](https://www.mikulskibartosz.name/different-retry-delay-for-every-airflow-task/)
---
# 📌 Tuning Airflow Performance

### **Concurrency Parameters**
- **`parallelism` (airflow.cfg, default = 32)**  
  - Defines the number of parallel tasks that can run across the entire Airflow instance.

- **`dag_concurrency` or `concurrency` (airflow.cfg or DAG parameter)**  
  - Defines the number of parallel tasks that can run within a single DAG instance.

- **`max_active_runs` (airflow.cfg or DAG parameter)**  
  - Specifies the maximum number of active DAG runs allowed simultaneously.

### **Celery Executor**
- **`worker_concurrency` (airflow.cfg)**  
  - Defines the number of tasks a worker can execute concurrently.

---

# 📌 Best Practices and Tips

### **1️⃣ Idempotent DAGs**
🔗 **Reference:** [Apache Airflow Tips & Best Practices](https://towardsdatascience.com/apache-airflow-tips-and-best-practices-ff64ce92ef8)

#### **Scenario**
- A DAG runs a Python function daily to retrieve marketing ad performance data from an API and loads it into a database.
- The DAG uses the **current timestamp** to determine the date.
- If the DAG's `start_date` is **2019-12-01** and the Airflow scheduler starts on **2019-12-08**, then **seven past DAG runs** will execute on **2019-12-08**.
- Since the API function dynamically retrieves "yesterday's" date, **all backfilled DAG runs will fetch data for 2019-12-07** and duplicate it in the database.

#### **Solution: Make the DAG Idempotent**
✔ **Use DELETE before INSERT**  
✔ **Compute the date based on Execution Date**  
  - Instead of using the `datetime` library, use **`{{ ds }}`**, one of Airflow’s built-in template variables, to get the execution date of the DAG run.
  - This makes the date independent of the DAG's actual run date.

---

### **2️⃣ Completely Removing a DAG**
- To fully delete a DAG:
  1. **Remove the DAG file** from the Airflow DAGs directory.
  2. **Delete the DAG metadata** from the Airflow database.
  3. **Use the Airflow UI "Delete" button** or run the CLI command:
     ```bash
     airflow delete_dag <dag_id>
     ```

---

### **3️⃣ Avoid Renaming Existing DAGs**
- **Do NOT rename an existing DAG** unless absolutely necessary.
- Renaming a DAG creates a **new DAG entry** in the metadata database, without deleting the old one.
- This leads to:
  - **Loss of DAG run history**  
  - **Unintentional backfilling** (if `catchup=True`)  

**❗Important Note:**  
> Changing the DAG ID is equivalent to creating a brand new DAG. Airflow will register a new entry in the metadata database without deleting the old one. This can cause issues if catchup is enabled, as Airflow will attempt to backfill all past DAG runs.

# 📌 Airflow Refresher
🔗 **Reference:** [Build Data Pipelines with Apache Airflow](https://towardsdatascience.com/https-medium-com-xinran-waibel-build-data-pipelines-with-apache-airflow-808a4de79047)

---

# 📌 Dynamic DAGs in Airflow
🔗 **Reference:** [Dynamic Workflows in Airflow](https://www.linkedin.com/pulse/dynamic-workflows-airflow-kyle-bridenstine/)

# 📌 Dynamic DAGs - Basic (Global Concept)
🔗 **Reference:** [Airflow Dynamic DAGs using Python Globals](https://galea.medium.com/airflow-dynamic-dags-python-globals-4f40905d314a)

## 🔹 Dynamically Creating DAGs in Airflow
To dynamically create DAGs, two key steps are required:
1. Run a function that instantiates an `airflow.DAG` object.
2. Pass that object back to the **global namespace** of the DAG file.

### 🔹 Example:
```python
from airflow import DAG

def create_dag(symbol):
    with DAG(...):
        pass
    return dag
```
### 🔹 Need for Global Variable
* Airflow loads any DAG object it can import from a DAG file.
* The DAG must appear in globals() for Airflow to recognize it.
* Python's globals() is a built-in function that returns a dictionary of global variables.
	```
	>>> globals()["my_name"] = "Alex"
	>>> print(my_name)
	Alex
	```
	
### Using globals() to Register DAGs
```python 
for symbol in ("BTC", "ETH", "LTC", "XLM"):
    dag = create_dag(symbol=symbol)
    globals()["{}_dag".format(symbol.lower())] = dag
```
# 📌 Airflow Schedule - Start Date vs. Execution Date
🔗 **Reference:** [Airflow Schedule Interval 101](https://towardsdatascience.com/airflow-schedule-interval-101-bbdda31cc463)

## 🔹 Key Concepts

### 1️⃣ DAG Trigger Mechanism
- A DAG will be triggered **when both the `start_date` and `schedule_interval` have elapsed**.
- **Example:**
  ```plaintext
  start_date = 01/01/2020 10:00 AM
  schedule_interval = 10 minutes
  DAG's first run: 01/01/2020 10:10 AM
	```
	# 📌 Airflow Doesn't Trigger DAGs in Real Time

- Airflow **monitors all DAGs at specific intervals** based on the `scheduler_heartbeat_sec` setting.
- **For production**, this value should be **greater than 60 seconds** to prevent excessive load on the scheduler and database.

## 🔹 `schedule_interval` Options
1. `@once` → DAG runs **only once**.
2. `None` → DAG must be **triggered manually**.

## 🔹 `start_date`
- Defines **when the DAG becomes eligible to be triggered**.

## 🔹 `execution_date`
- Represents **the last successfully completed schedule date**.

### 📌 Example:
```plaintext
Schedule: 0 2 * * 4,5,6 (Runs at 2:00 AM on Thursday, Friday, Saturday)
DAG Start Date: Thursday, 2-April
Execution Date: 28-March
```
### Why?

* The execution window is from 2-April 2:00 AM to 3-April 2:00 AM.
* When the DAG runs on Friday (3-April), the execution date will be 2-April.

# 📌 Airflow - Macros, Templates, and Variables

🔗 **Reference:** [Templates & Macros in Apache Airflow](https://marclamberti.com/blog/templates-macros-apache-airflow/)

## 🔹 Use Cases:
- Retrieve the **execution date** of the DAG.
- Access the **DAG ID** dynamically.
- Use the **execution date** in SQL queries.

## 🔹 Macros:
- Airflow provides **predefined macros** that can be used within templates.
- Some macro values are **objects** rather than **literal values**.

### 📌 Examples:
- `{{ dag.dag_id }}` → Retrieves the **DAG ID**.
- `{{ var.value.var_key }}` → Fetches the **Airflow variable value**.
# 🚀 Airflow 2.0: Key Enhancements  
---
## 🔹 1) High Availability Scheduler  
- In earlier versions, the **scheduler** was a **single point of failure**.  
- Airflow 2.0 allows **multiple instances** of the scheduler to prevent failures.  

## 🔹 2) DAG Versioning  
- Previously, adding a new task to an existing DAG required **creating a new DAG**.  
- Airflow 2.0 introduces **DAG Versioning**, enabling updates without duplication.  

## 🔹 3) DAG Serialization  
- Airflow scans the **DAG folder**, **parses DAGs**, and **stores them in the database**.  
- DAGs are **lazy-loaded**, meaning they appear in the Airflow UI **only when needed**.  
# 🔹 Use Cases: Handling Historical Data in Airflow  

## 📌 Case 1: Managing Backfills with Limited Data Retention  
**Reference:** [The Zen of Python and Apache Airflow](https://godatadriven.com/blog/the-zen-of-python-and-apache-airflow/)  

### 🛠️ Option 1: Using `ShortCircuitOperator`  
📌 **Scenario:**  
- You download a `.zip` file from an **SFTP server** daily.  
- The server retains **one week of history**.  
- If you try to **backfill beyond one week**, it **fails** because the old data no longer exists.  

🔹 **Solution:**  
- Use **`ShortCircuitOperator`** to check if the execution date is older than **today - 7 days**.  
- If `False`, all **downstream tasks are skipped**.  

### 🛠️ Option 2: Using `PythonOperator` with `AirflowSkipException`  
📌 **Alternative Approach:**  
- Use a **Python function** to check the execution date.  
- Raise **`AirflowSkipException`** if the date is **too old**, skipping the task **and all downstream tasks** automatically.