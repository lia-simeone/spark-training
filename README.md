# spark-training

* Difference between Hadoop and Spark
  * Hadoop pushes everything to disk and then reads it back out which is very slow for iterative work
  * Spark stores data in memory using a waterfall: memory first then disk if your data is larger than RAM
  * yields 10-100x speed benefits over Hadoop
  * Spark was originally written in scala, then wrappers were written for python (pyspark, but it's slower than scala)

* Resilent and distributed datasets (RDD)
  * datasets are partitioned into pieces
  * each partition is randomly assigned to a single worker
  * spark knows where each of the partitions are stored (like Teradata)
  * if one of the workers goes down, it will recompute data only for that lost partition
  * there are default partition sizes, but we should be aware of the RAM constraints
  * IMPORTANT: each partition must be smaller than memory for a single worker

* Transformations and Actions
  * Transformations are graphs of data manipulations on RDDs
    * called a Directed Acyclic Graph (DAG)
    * nothing will be actually run here
    * NOTE: for RDDs, the first part of it is called the "key"
    * E.g. map, filter, groupByKey
  * Actions trigger the data running through the DAG
    * code is actually compiled and run
    * results are taken from  worker nodes and pushed back to the driver/master node
    * E.g. collect, take, first, count

* Narrow and Wide transformations
  * Narrow
    * one-to-one mapping: each partiton of the parent RDD is used by at most one partition of the child RDD
    * no data moves across partitions so there are no network costs
    * e.g. map, filter, union
  * Wide
    * multiple child RDD partitions may depend on a single parent RDD partition
    * data gets moved across partitions
    * NOTE: wide transmations will incur network costs
    * e.g. groupByKey

* Building DAG
  * after you create a DAG and run an action, Spark looks for optimization opportunities
  * it looks for:
    1. Shuffle boundaries - these are dependency points where work cannot be run in parallel (see visual example)
  * then it runs data through DAG
    * when a stage is submitted, tasks are assigned
    * assigns one task per partition
    * think of the tasks as a single thread - they can be run in parallel
    
* Jobs, Stages, and Tasks
  * Jobs are triggered by Actions
    * composed of one of more Stages
  * Stages are composed of one or more Tasks
  
* Ways to run Spark
  * Static resource management
    * local, standalone
    * allocate CPU, RAM, worker numbers
    * once you assign a task, resources cannot be changed
  * Dyanmic resource management
    * YARN, Mesos
    * resources are modified based on submitted jobs
    * e.g. the first job could have 100% of resources, but if a second job is submitted each would get 50%
  
* Standalone mode
  * each machine is JVM (java virtual machine?)
  * one master - doesn't do any computation, doles out work to workers
  * three Worker JVMs report results back to Master and assign jobs
  * Executor JVM is below the worker JVM and actually run the jobs
  * NOTE: worker cores is the  number of tasks or threads that can be run at any time
    * Spark default is the same number of cores as machines on each Executor, but it makes more sense to use more (?)

* Spark DataFrames and Spark SQL
  * RDDs stores each record as tuples, but there's no schema within the data which makes it really hard to read
  * DataFrames stores each record as a row object where the schema is embedded
  * for Spark SQL, indicate the df is a table and then write SQL query as normal (yay!)
  * there's minimal performance difference between DF form and SQL, but the latter is easier to read
  
* Machine Learning on Spark
  * MLLib
    * inputs and outputs are all RDDs
    * Lots of APIs needed to get the RDDs in the right format 
  * ML Pipeline
    * inputs and outputs are all DFs
    * builds off of MLLib
    * plug and chug, much more reasonable, less fighting with code
    * high level steps: Transformer, Estimator, Pipeline
  
* Spark tuning - persistence and caching
  * caching avoids rerunning steps repeatedly
    1. disk only - always serialized
    2. spill to disk when memory is full
    3. memory only - same as calling .cache
    4. off heap - tachyon storage system, experiemental project
  * default splitting of Executor JVM
    1. Execution 20% 
    2. Shuffle 20%
    3. Peristing and Cache 60%

* reduceByKey vs. groupbyKey
 * huge differences in computational requirements
 * try to always use the former
 * add more details here?
