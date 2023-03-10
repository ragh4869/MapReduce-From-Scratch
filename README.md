<h1 align="center">
MapReduce-From-Scratch
</h1>

### Objective:

The main goal of this project is to locally build a distributed MapReduce system with Word Count and Inverted Index applications from scratch.

### Functionalities:
* **Nodes/Tasks:** Nodes and tasks are implemented as conventional OS processes. Each task is communicated and executed through network sockets.
* **Data Storage:** The data storage is using the simple key-value store (KV-store) where the input data, intermediate data and output data are accessed through the master node, mapper and reducer for any file manipulations. The files being generated and stored are being replicated as the distributed file system such as HDFS. In addition to the set_data and get_data functions, the del_file and write_file functions are defined to handle the deletion of files and writing of files.
* **Master Node:** The master node is spawned which spawns the map and reduce tasks. Additionally, it controls and coordinates all the other processes.
* **User Interaction:** Inputs are taken in and run through the usage of configuration file stored in the form of json file, thus, the user does not need to input anything from their end. If any changes are required, it can be changed in the configuration file.
* **Fault Tolerance:** The code also implements fault tolerance to survive process failures. This is done by restarting only the failed processes. A max_tries variable is defined to set the number of times it tries to rectify the failures.

### How to run? :

* Open a command prompt window and run the server.py
* Open an another command prompt window and run the main.py

### Applications:

#### **Word-count:** 

Find the frequency of the words in a given text file

**Output:** word1: frequency1, word2: frequency2

#### **Inverted-index:** 

Find the frequency of the words with respect to each of the given files.

**Output:** word1: {doc1: frequency_word1, doc2: frequency_word1, doc3: frequency_word1}

### Design:

#### Master Node –

The master node is first spawned and connected with the server through which the Key-Value store is accessed and the intermediary file which is going to hold all the input and the log files are generated by invoking the set function. It then splits the files to match the number of mappers (m) and spawns ‘m’ processes with the file key as input. After this, the hashmap is implemented through which the get function is invoked to get all the respective files, thus, using these files, the hashmap maps the keys to match the number of reducers (r). These file keys are then sent by the master node to the reducers once ‘r’ processes are spawned to perform the reducer function. After the mapper and reducer have finished their process respectively, a check is made after which it moves on. Finally, the reducer outputs are combined and written in a json file as the output along with the deletion of the intermediary file. The workflow below show the process flow –

![MapReduce-System](https://user-images.githubusercontent.com/96961381/210404949-e3494396-e6d0-4b2a-b914-1e9328959ddf.JPG)

![MapReduce-Flow](https://user-images.githubusercontent.com/96961381/210404936-9f02d482-4d07-440a-ad11-1d03ce03bbcd.jpeg)

#### Mapper:

The mapper is spawned from the master node and is given the input - file keys which are the keys used to access the Key-Value store through the get function associated with their respective data. The input files are the files split for each mapper. The mapper retrieves the data through get and performs the operation of associating for the applications as follow – 
*	Word Count - word:1 and separated from others by the delimiter ‘,’
*	Inverted Index – doc_id@word:1 and separated from others by the delimiter ‘,’

Each mapper produces this intermediary output for each associated word. This output is given as input to the Distributed Shuffle/GroupBy. The code is well commented and explained.

#### Distributed Shuffle/GroupBy:

This process is implemented in the master node using the Hashmap. The data is initially gathered from all the mapper outputs through the KV-store using the get function. After this, the hash value of each key is checked and the key is sent to the respective reducer files. These reducer files are updated in the KV-store and ready for use by the reducers. The code is well commented and explained.

![Distributed GroupBy](https://user-images.githubusercontent.com/96961381/210405517-b617e3ef-5dfe-4e80-9ff8-bc12a7de390f.jpeg)

#### Reducer:

The reducer is spawned from the master node and is given the input - file keys which are the keys used to access the Key-Value store through the get function associated with their respective data. The input files are the files having the respective keys based on their hash values. The reducer retrieves the data through get and performs the operation of associating for the applications as follow – 
*	Word Count - word:total_count and separated from others by the delimiter ‘,’
*	Inverted Index – doc_id@word:total_count and separated from others by the delimiter ‘,’

Each reducer produces this final output for each associated word. This output is given as input to the write function which writes a json file using the reducer outputs. The code is well commented and explained.

#### Distributed Barrier:

It is very essential to check if all the processes have run successfully whether it is a mapper and reducer. To do this, it is essential to make sure to perform the check after all the processes have finished running. This is achieved with the implementation of the distributed barrier on mapper and reducer. The barrier ensures that the check is not done till all the mappers or all the reducers process are completed as shown below –

![Mapper Barrier](https://user-images.githubusercontent.com/96961381/210406067-c5625c8c-c866-4a0e-afce-43ac453cb86e.jpeg)

![Reducer Barrier](https://user-images.githubusercontent.com/96961381/210406071-3acb2c97-a87c-49f6-a42b-79e8313733ad.jpeg)

To create this, I used the multiprocessing package and extracted the Barrier module. The module is initialized with the number of process to look out for till which no other function can be run. The Barrier object is sent into the mapper or reducer process and the wait function is called. This enables it to wait even though a mapper/reducer has finished the computation.

Example: For 5 reducer processes, the Barrier is given to check for 6 processes and each Barrier object is sent through the reducer process which makes it wait till all reducer processes are done.

#### Fault Tolerance:

In any system, the fault tolerance is the most important factor which could hold the system from crashing or lose important data. In this MapReduce system, I have implemented the same using key checkpoints and re-running the mappers or reducers that have not successfully run. The below images shows the implementation for the same – 

![Fault Check](https://user-images.githubusercontent.com/96961381/210406832-14f0bc4c-7209-4491-82c8-b689535a9676.jpeg)

![Fault Mapper](https://user-images.githubusercontent.com/96961381/210406833-af5f68bf-6d4b-4418-b7b3-606327b62270.jpeg)

![Fault Reducer](https://user-images.githubusercontent.com/96961381/210406837-e83ed9c6-1ab3-431f-a106-4f41e0d587f9.jpeg)

### Applications Implemented:

The applications implemented for this MapReduce system are – 
*	Word Count
*	Inverted Index

Both these applications are utilized in all the code files where special functions are defined for each application. The code is well commented and explained.

### Effective Use of key-value store (KV-store):

The KV-store which is implemented has four functions – 
*	get_data – Gets the data for a given key
*	set_data – Sets the data for a given key in the intermediary file
*	del_file – Deletes the intermediary file when invoked
*	write_file – Creates a final json output file taking the reducer outputs

Out of these four functions, the main functions being used are the set and get functions through which almost all the operations are being handled. The set and get functions store or retrieve the data value which is a huge string with all the required data in it. The data is stored or retrieved from a intermediary text file. This makes it much easier and faster to store than in json file as the json file would require lot of set or get operations making the operations very slow. I initially implemented the json file but later moved on to storing and retrieving from a text file.

### Testing and ease of running:

The testing is very easy to do as I have created a configuration file (json file) to read from. The configuration file is read in the main.py and run for each configuration. To test the code, the server.py must run first to create a server instance running in the background. Then the main.py must run after which all the operations in the configuration file is run and updated. Below shows the configuration file –

![Test-Run](https://user-images.githubusercontent.com/96961381/210408065-ea945e31-2186-414d-a215-d7346bc3222a.jpeg)

The output and log files for each task are stored in folders with the same name as the task. 

