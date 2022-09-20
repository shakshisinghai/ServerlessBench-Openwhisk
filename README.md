# [ServerlessBench](https://serverlessbench.systems/en-us/)

We are using various test cases to check characteristic metrics of serverless computing.

## Metrics

### Communication performance
* Complex serverless application consist of several interacting functions and other cloud services.
* There are different function-interacting models to initiate the inter-function communication.

###  Startup latency
* Serverless functions are initiated on-demand, thus, the processing latency of each request contains the startup overhead (including sandbox preparation, function loading and initialization).
* Snice these functions are short lived, second-level startup can be a considerable overhead.
* high concurrency in a serverless platform makes it harder to achieve low-latency startup.

###  Stateless execution.
* This compromise the application performance in below ways,
  * data transmission overhead to maintain the states needed by the function logic
  * loss of system states which contain useful information that can help improve performance
 
### Resource efficiency and performance isolation


## Tests in openwhisk:

### Composition Method :

* Sequence function chain 
* Nested function chain

### Test 1 : [Long function chain](https://github.com/SJTU-IPADS/ServerlessBench/tree/master/Testcase3-Long-function-chain)
**Array Sum** is a serverless application with a configurable number of chained functions(the chain length is set to 100 in our test).

Twp variants :
1. sequence function chain method :
   *  Each action delivers its result to another action and finish itself. 
   
   Return Value : The start time of each action, return time of each action
   
  Results:
  * The first invocation triggers a cold start for the first function in the chain, leading to higher   latency for the first request.
  * subsequent function in a sequence chain can reuse the container holding prior functions.
    
2. nested function chain method
  * The action invokes another action till n, and returns until the invoked action returns.
  * n should not be greater than the maximum allowed number of the concurrent runtime containers, which is limited by openwhisk.
  Return Value : The start time of each action, return time of each action, time of each action to invoke another action, except the last one and the result
  
  
  Results:
  *  First five requests fail with a timeout error as first function in a nested chain waits for all the following 99 functions to finish before it can return, so it easily exceeds the timeout value (default to 60s in OpenWhisk)
  * From the 6th request, there are enough warmed containers to allow the nested chain to finish request handling without a timeout, and the performance improves with more warmed functions We send the requests successively and record the processing latency of each request.
  
  Conclusion : nested function chain method faces a higher timeout risk than the sequence function chain
method, as the timeout limit of the first function is enforced on the whole chain.

![image](https://user-images.githubusercontent.com/37688219/190928535-fed89fa3-9991-4d5b-b040-e9d314cb3522.png)

### Test 2 : [Application breakdown](https://github.com/SJTU-IPADS/ServerlessBench/tree/master/Testcase4-Application-breakdown)
Real-world serverless applications with different composition patterns:

1. Image processing: Five functions in a sequence chain
  * The function is triggered when a request arrives. Request includes the name of an image in the cloud database (i.e., “CouchDB” in the figure). T
  * he function “Extract img metadata” is fired to extract the metadata from the image, and pass the result to the next function (“Transform metadata”) to validate the image format. 
  * After that, a “Handler” function is invoked to process the images, e.g., detecting objects in the image. 
  * The handled image, along with the extracted metadata, is saved to storage. Then a thumbnail is generated and packed in the function response to the client.

![image](https://user-images.githubusercontent.com/37688219/190931726-be76d3d5-1cee-49cf-a974-1c543f6d56b3.png)


2. Online compiling:
  * We implement the online compiling application based on gg framework, which leverages the auto-scaling nature of serverless computing to boost the compiling performance.
  * Online compiling relies on an external coordinator to schedule functions (sequence function chain).
  
![image](https://user-images.githubusercontent.com/37688219/190931959-d2562db9-79a6-461b-a1a8-fbaf375ee066.png)

Data analysis
  * This is triggered by a thirdparty event-source. The application begins with the insertion of personal wage data (implemented with nested function chain). 
  * Data checker and formatter are applied to the wage data before it is finally inserted into the database (i.e., CouchDB in the figure)
 * Then it detects the database change, and triggers a database analysis sequence function chain.
 *  Finally, the analysis results of the updated wage data set are written back into CouchDB.

![image](https://user-images.githubusercontent.com/37688219/190932138-56150da9-13d2-452d-ada8-783f145298ae.png)


### Test 3: [Data transfer costs](https://github.com/SJTU-IPADS/ServerlessBench/tree/master/Testcase5-Data-transfer-costs/OpenWhisk)
This test case presents a Node.js serverless application which transfers images with different sizes (the payload size) between two functions.
Sizesdic=(0 1024 5120 10240 15360 20480 25600 30720 35840 40960 46080 51200)

###  Test 4 :Startup breakdown
Detailed breakdown of the startup latency of four functions in ServerlessBench: PythonHello, Java-Hello, Python-Django and Java-ImageResize

Results :
* optimize the startup latency by holding or caching the finished sandboxes on the platform to avoid long latency incurred by cold start for every request. 
* However, the finished function instances can remain in hot or warm state only for a short period of time before they cool down, because the platform cannot predict when will a next request arrive, and keeping idle sandboxes wastes platform resources

### Concurrent startup
### Stateless costs


Other tests links:
https://github.com/i13tum/openwhisk-bench/tree/626c90d8b3f7deba06e233b3ed6013ae9f08f8cf
