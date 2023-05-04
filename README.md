Download Link: https://assignmentchef.com/product/solved-cse-486-assignment-4-replicated-key-value-storage
<br>
At this point, most of you are probably ready to understand and implement a Dynamo-style key-value storage; this assignment is about implementing a simplified version of Dynamo. (And you might argue that it’s not Dynamo any more &#x1f609; There are three main pieces you need to implement: 1) Partitioning, 2) Replication, and 3) Failure handling.

The main goal is to provide both availability and linearizability at the same time. In other words, your implementation should always perform read and write operations successfully even under failures. At the same time, a read operation should always return the most recent value. To accomplish this goal, this document gives you a guideline of the implementation. ​However, you have freedom to come up with your own design as long as you provide availability and linearizability at the same time (that is, to the extent that the tester can test)​. ​The exception is partitioning and replication, which should be done exactly the way Dynamo does.

This document assumes that you are already familiar with Dynamo. If you are not, that is your first step. There are many similarities between this assignment and the previous assignment for the most basic functionalities, and you are free to reuse your code from the previous assignment.

<h1>References</h1>

Before we discuss the requirements of this assignment, here are two references for the Dynamo design:

<ol>

 <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/lectures/26-dynamo.pdf">Lecture slides</a></u></li>

 <li><u><a href="http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf">Dynamo paper</a></u></li>

</ol>

The lecture slides give an overview, but do not discuss Dynamo in detail, so it should be a good reference to get an overall idea. The paper presents the detail, so it should be a good reference for actual implementation.

<h1>Step 0: Importing the project template</h1>

Just like the previous assignment, we have a project template you can import to Android Studio.

<ol>

 <li>Download<u>​</u><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/SimpleDynamo.zip"> the project template zip file</a></u><u>​</u> to a directory.</li>

 <li>Extract the template zip file and copy it to your Android Studio projects directory.</li>

 <li>Make sure that you copy the correct directory. After unzipping, the directory name should be “SimpleDynamo”, and right underneath, it should contain a number of directories and files such as “app”, “build”, “gradle”, “build.gradle”, “gradlew”, etc.</li>

 <li>After copying, delete the downloaded template zip file and unzipped directories and files. This is to make sure that you do not submit the template you just downloaded. (There were many people who did this before.)</li>

 <li>Open the project template you just copied in Android Studio.</li>

 <li>Use the project template for implementing all the components for this assignment.</li>

 <li>The template has the package name of “edu.buffalo.cse.cse486586.simpledynamo“. Please do not change this.</li>

 <li>The template also defines a content provider authority and class. Please use it to implement your Dynamo functionalities.</li>

 <li>We will use SHA-1 as our hash function to generate keys just as last time.</li>

 <li>The template is very minimal for this assignment. However, you can reuse any code from your previous submissions.</li>

 <li>You can add more to the main Activity in order to test your code. However, this is entirely optional and there is no grading component for your Activity.</li>

</ol>

<h1>Step 1: Writing the Content Provider</h1>

Just like the previous assignment, the content provider should implement all storage functionalities. For example, it should create server and client threads (if this is what you decide to implement), open sockets, and respond to incoming requests. When writing your system, you can make the following assumptions:

<ol>

 <li>Just like the previous assignment, you need to support insert/query/delete operations. Also, you need to support @ and * queries.</li>

 <li>There are always 5 nodes in the system. There is no need to implement adding/removing nodes from the system.</li>

 <li>However, there can be ​<em>at most 1 node failure at any given time</em>​. We will emulate a failure only by force closing an app instance. We will ​<strong><em>not</em></strong>​ emulate a failure by killing an entire emulator instance.</li>

 <li><em>All failures are temporary;</em>​ you can assume that a failed node will recover soon, i.e., it will not be permanently unavailable during a run.</li>

 <li><em>When a node recovers, it should copy all the object writes it missed during the failure. </em>This can be done by asking the right nodes and copy from them.</li>

 <li><em>Please focus on correctness rather than performance.</em>​ Once you handle failures correctly, if you still have time, you can improve your performance.</li>

 <li>Your content provider should support ​<em>concurrent read/write operations</em>​.</li>

 <li>Your content provider should handle ​<em>a failure happening at the same time with read/write operations</em>​.</li>

 <li><em>Replication should be done exactly the same way as Dynamo does.</em>​ In other words, a (key, value) pair should be replicated over three consecutive partitions, starting from the partition that the key belongs to.</li>

 <li>Unlike Dynamo, there are two things you do not need to implement.

  <ol>

   <li>Virtual nodes: Your implementation should use physical nodes rather than virtual nodes, i.e., all partitions are static and fixed.</li>

   <li>Hinted handoff: Your implementation do not need to implement hinted handoff.</li>

  </ol></li>

</ol>

This means that when there is a failure, it is OK to replicate on only two nodes.

<ol start="11">

 <li>All replicas should store the same value for each key. This is “per-key” consistency. There is no consistency guarantee you need to provide across keys. More formally, you need to implement ​<em>per-key linearizability</em>​.</li>

 <li>Each content provider instance should have a node id derived from its emulator port. This node id should be obtained by applying the above hash function (i.e., genHash()) to the emulator port. For example, the node id of the content provider instance running on emulator-5554 should be, <em>node_id = genHash(“5554”)</em>​ ​. This is necessary to find the correct position of each node in the Dynamo ring.</li>

 <li>Your content provider’s URI should be</li>

</ol>

“content://edu.buffalo.cse.cse486586.simpledynamo.provider”, which means that any app should be able to access your content provider using that URI. This is already defined in the template, so please don’t change this. Your content provider does not need to match/support any other URI pattern.

<ol start="14">

 <li>We have fixed the ports &amp; sockets.

  <ol start="10000">

   <li>Your app should open one server socket that listens on 10000.</li>

   <li>You need to use run_avd.py and set_redir.py to set up the testing environment.</li>

   <li>The grading will use 5 AVDs. The redirection ports are 11108, 11112, 11116, 11120, and 11124.</li>

   <li>You should just hard-code the above 5 ports and use them to set up connections.</li>

   <li>Please use the code snippet provided in PA1 on how to determine your local AVD.</li>

   <li>emulator-5554: “5554” ii. emulator-5556: “5556” iii.          emulator-5558: “5558” iv.          emulator-5560: “5560” v.          emulator-5562: “5562”</li>

  </ol></li>

 <li>Any app (not just your app) should be able to access (read and write) your content provider. As with the previous assignment, please do not include any permission to access your content provider.</li>

 <li>Please read the notes at the end of this document. You might run into certain problems, and the notes might give you some ideas about a couple of potential problems.</li>

</ol>




The following is a guideline for your content provider based on the design of Amazon Dynamo:

<h2>1. Membership</h2>

<ol>

 <li><em>Just as the original Dynamo, every node can know every other node.</em>​ This means that each node knows all other nodes in the system and also knows exactly which partition belongs to which node; any node can forward a request to the correct node without using a ring-based routing.</li>

</ol>

<h2>2. Request routing</h2>

<ol>

 <li>Unlike Chord, each Dynamo node knows all other nodes in the system and also knows exactly which partition belongs to which node.</li>

 <li>Under no failures, a request for a key is directly forwarded to the coordinator (i.e., the successor of the key), and the coordinator should be in charge of serving read/write operations.</li>

</ol>

<h2>3. Quorum replication</h2>

<ol>

 <li>For linearizability, you can implement a quorum-based replication used by Dynamo.</li>

 <li>Note that the original design does not provide linearizability. You need to adapt the design.</li>

 <li><em>The replication degree N should be 3.</em>​ This means that given a key, the key’s coordinator as well as the 2 successor nodes in the Dynamo ring should store the key.</li>

 <li><em>Both the reader quorum size R and the writer quorum size W should be 2. </em></li>

 <li>The coordinator for a get/put request should ​<em>always contact other two nodes</em>​ and get a vote from each (i.e., an acknowledgement for a write, or a value for a read).</li>

 <li>For write operations, all objects can be ​versioned​ in order to distinguish stale copies from the most recent copy.</li>

 <li>For read operations, if the readers in the reader quorum have different versions of the same object, the coordinator should pick the most recent version and return it.</li>

</ol>

<h2>4. Chain replication</h2>

<ol>

 <li>Another replication strategy you can implement is chain replication, which provides linearizability.</li>

 <li>If you are interested in more details, please take a look at the following paper: <u><a href="http://www.cs.cornell.edu/home/rvr/papers/osdi04.pdf">http://www.cs.cornell.edu/home/rvr/papers/osdi04.pdf</a></u></li>

 <li>In chain replication, a write operation always comes to the first partition; then it propagates to the next two partitions in sequence. The last partition returns the result of the write.</li>

 <li>A read operation always comes to the last partition and reads the value from the last partition.</li>

</ol>

<h2>5. Failure handling</h2>

<ol>

 <li>Handling failures should be done very carefully because there can be many corner cases to consider and cover.</li>

 <li>Just as the original Dynamo, each request can be used to detect a node failure.</li>

 <li><em>For this purpose, you can use a timeout for a socket read;</em>​ you can pick a reasonable timeout value, e.g., 100 ms, and if a node does not respond within the timeout, you can consider it a failure.</li>

 <li><strong><em>Do not rely on socket creation or connect status to determine if a node has failed.</em></strong>​ Due to the Android emulator networking setup, it is ​<strong>not</strong>​ safe to rely on socket creation or connect status to judge node failures. Please use an explicit method to test whether an app instance is running or not, e.g., using a socket read timeout as described above.</li>

 <li>When a coordinator for a request fails and it does not respond to the request, ​<em>its successor can be contacted next for the request.</em></li>

</ol>

<h1>Testing</h1>

We have testing programs to help you see how your code does with our grading criteria. There are 6 phases in testing. ​The first three phases will not take much time, so it’ll be better to finish them as quickly as possible. You will then be able to spend most of your time for the last three phases.

<ol>

 <li>Testing basic ops

  <ol>

   <li>This phase will test basic operations, i.e., insert, query, delete, @, and *. This will test if everything is correctly replicated. There is no concurrency in operations and there is no failure either.</li>

  </ol></li>

 <li>Testing concurrent ops with different keys

  <ol>

   <li>This phase will test if your implementation can handle concurrent operations under no failure.</li>

   <li>The tester will use independent (key, value) pairs inserted/queried concurrently on all the nodes.</li>

  </ol></li>

 <li>Testing concurrent ops with same keys

  <ol>

   <li>This phase will test if your implementation can handle concurrent operations with same keys under no failure.</li>

   <li>The tester will use the same set of (key, value) pairs inserted/queried concurrently on all the nodes.</li>

  </ol></li>

 <li>Testing one failure

  <ol>

   <li>This phase will test one failure with every operation.</li>

   <li>One node will crash before operations start. After all the operations are done, the node will recover.</li>

   <li>This will be repeated for each and every operation.</li>

  </ol></li>

 <li>Testing concurrent operations with one failure

  <ol>

   <li>This phase will execute operations concurrently and crash one node in the middle of the execution. After some time, the failed node will also recover in the middle of the execution.</li>

  </ol></li>

 <li>Testing concurrent operations with one consistent failure

  <ol>

   <li>This phase will crash one node at a time consistently, i.e., one node will crash then recover, and another node will crash and recover, etc.</li>

   <li>There will be a brief period of time in between the crash-recover sequence.</li>

  </ol></li>

</ol>




Each testing phase is quite intensive (i.e., it will take some time for each phase to finish), so the tester allows you to specify which testing phase you want to test. You won’t have to wait until everything is finished every time. However, you still need to make sure that you run the tester in its entirety before you submit. ​<em>We will not test individual testing phases separately in our grading.</em>

<ul>

 <li>You can specify which testing phase you want to test by providing ‘-p’ or ‘–phase’ argument to the tester.</li>

 <li>Note​: If you run an individual phase with “-p”, it will always be a fresh install. However if you run all phases (without “-p”), it will not always be a fresh install; the grader will do a fresh-install before phase 1, and do another fresh-install before phase 2. Afterwards, there will be no install. ​This means that all data from previous phases will remain intact.</li>

 <li>‘-h’ argument will show you what options are available.</li>

 <li>The grader uses multiple threads to test your code and each thread will independently print out its own log messages. This means that an error message might appear in the middle of the combined log messages from all threads, rather than at the end.</li>

 <li>Download a testing program for your platform. If your platform does not run it, please report it on Piazza.</li>

</ul>

○    <u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledynamo-grading.exe">Windows</a></u>​: We’ve tested it on 32- and 64-bit Windows 8.

○    <u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledynamo-grading.linux">Linux</a></u>​: We’ve tested it on 32- and 64-bit Ubuntu 12.04.

○    <u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledynamo-grading.osx">OS X</a></u>​: We’ve tested it on 32- and 64-bit OS X 10.9 Mavericks.

<ul>

 <li>Before you run the program, please make sure that you are running five AVDs. ​python py 5​ will do it.</li>

 <li>Run the testing program from the command line.</li>

 <li>On your terminal, it will give you your partial and final score, and in some cases, problems that the testing program finds.</li>

</ul>