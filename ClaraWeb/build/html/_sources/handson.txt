********
Hands On
********

Hands on section is for CLARA Python binding. Here we assume that you have successfully installed zmq python module and
have a proper PYTHONPATH setup, pointing to Clara-p.zmq package.

We suggest using easy_install, perhaps in connection with a Python virtual environment. You need to install the
libzmq-dev package so that the Python package compiles.

.. code-block:: c

    sudo apt-get install libzmq-dev
    sudo easy_install pyzmq

Google's Protobuf is used to define and serialize/deserialization CLARA transient data envelope. You do not have
to install Protobuf (open source) in order to use CLARA, unless you would like to change CLARA transient data format
(not recommended, since it will require substantial rewrite of the existing code.

These are two technologies used in the framework (all are open source).

The only environmental variable required to be set is $PCLARA_HOME, pointing to the home directory of the pClara
installation. In you .bashrc or any other shell startup scrip if you set $PATH pointing also to $PCLARA_HOME/script/unix
then you can use package presented shell scripts to start/stop CLARA components. All startup scripts support *-h* or
*-help* option to print synopsis of the scripts.


.. _starting_dpe:

Starting a DPE
==============

To run a DPE type:

.. code-block:: c

    >p_dpe
    ================================
               CLARA Dpe
    ================================
    lang = Python
    date = Fri Jan 23 14:37:39 2015
    host = 129.57.81.247
    ================================
    Info: xMsg local registration and discovery server is started
    Info: Running xMsg proxy server on the localhost...

DPE will start it's local proxy and Registrar service.

.. _starting_service:

Starting a service
==================

To deploy a service we run a service container and specify the service engine class name. Not that service engine class
must be accessible through PYTHONPATH variable in order the container to be able to dynamically load the engine
class.  Service container needs to parameters for successful service deployment:

 #. the module name containing the engine class (Python implementation specific)

 #. the name of the container and

 #. the service engine class name

 #. object pool size

Here is the synopsis of the p_service script:

.. code-block:: c

    >p_service -help

    Starts CLARA service (python binding)
    -------------------------------------
    synopsis:
    p_service [service engine module] [container] [service class] [object pool size]


An example for the service deployment with the package name *ServiceEngine* , engine class name = *F1*, and the
container name = *abc* is shown below:

.. code-block:: c

    >p_service ServiceEngine abc F1 3
    registering with registrar and discovery services...


After service is deployed the first thing it does is registering it's services wit the Registrar and Discovery services
Service container console will print a message about the status of the registration.
Note the canonical
name of the service that successfully completed the registration process.
Platform and DPE consoles will print Registrar service messages, informing
the incoming service registration request:

.. code-block:: c

    >p_dpe
    ================================
               CLARA Dpe
    ================================
    lang = Python
    date = Fri Jan 23 14:37:39 2015
    host = 129.57.81.247
    ================================
    Info: xMsg local registration and discovery server is started
    Info: Running xMsg proxy server on the localhost...
    Received a request from 129.57.81.247:abc:Engine1 to registerSubscriber

.. _monitor_example2:

Running discover service example
================================

Now let us run DiscoverService example described in the Examples section of this documentation.
Change directory to $PCLARA_HOME/example/orchestrators, and run DiscoverService.py.

.. code-block:: c

    >p_finder
    acceptable commands: getServiceByName, getServiceByEngineName, getServicesByContainer, getServicesByHost, listServices
    enter a command:

Let us choose the first command, requesting the description of a service, followed by the definition of
the canonical name of the service of interest.

.. code-block:: c

    >p_finder
    acceptable commands: getServiceByName, getServiceByEngineName, getServicesByContainer, getServicesByHost
    enter a command:getServiceByName
    what is the canonical name of the service?129.57.81.247:abc:F1
    ==== DISCOVER SERVICE RESULT ====
    F1 description
    =================================
    acceptable commands: getServiceByName, getServiceByEngineName, getServicesByContainer, getServicesByHost
    enter a command:

Not the result, i.e. the service description string set by the service engine developer
(see CLARA *ACEngine* abstract class *get_description* method). Service
engine examples can be found in the  Clara-p.zm1/example/engine/ServiceEngine.py file.

.. _orchestrator_example2:

Hello world application
=======================

It has become customary when learning a new programming language or testing an unfamiliar programming environment to
write a "Hello world" program. So, without braking the tradition let us create our first CLARA application that will
greet us. However, instead of writing a service that generates the "Hello world" string at the request, we will present
something more complex and interesting. Let us shovel letters of our string of greetings in 4 different files.
To decompose a string and distribute letters of a string in 4 different files we will use a program presented within
the CLARA package, called *Decomposer.py*.

.. code-block:: c

    > p_decompose "A warm greetings from CLARA"

As a result of this execution 4 files will be created in $PCLARA_HOME/example/engines/Data directory, with the
following contents:

.. code-block:: c

    >more d1.txt
    Arri mA

    >more d2.txt
     menf R

    >more d3.txt
    w egrCA

    >more d4.txt
    agtsoL

Now let us create a service based application that will reconstruct our initial string of greetings,
i.e. "A warm greetings from CLARA". For that purpose we will create 6 services:

    * 4 services: F1,F2,F3,F4 opening and reading 4 data files (d1.txt, d2.txt, d3.txt and d4.txt) respectively. Note that F1-F4 services are generic and will be configured at the *configure* request of the orchestrator, telling each of them what file they have to open and serve one letter at time for every service request.
    * Sentence builder service: EB, that will receive letters from each of the data services and will construct a word.
    * Recorder service: R that will simple receive the 4 letter word from the EB service and will print it on a screen.



Recall example code and above described application diagram from the section **Examples** of this document. Here are steps to deploy and run our custom *greetings* application.

Steps for starting the platform, the dpe and the service are described above. So, we have to start all 6 services first. After services are started and they register their addresses and service description data with the registration and discovery services we execute an orchestrator that will send configuration request to data and recording services of the application.

.. code-block:: c

    >p_configure F1 F2 F3 F4 R
    sending configure request to 129.57.81.247:abc:F1 with the payload:
    status: INFO
    data: "/Users/gurjyan/Devel/Clara/Clara-p.zmq.0.2/example/engines/Data/d1.txt"
    dataType: STRING
    request_id: 1
    sender: "Orchestrator_46"
    composition: ""
    action: CONFIGURE

    sending configure request to 129.57.81.247:abc:F2 with the payload:
    status: INFO
    data: "/Users/gurjyan/Devel/Clara/Clara-p.zmq.0.2/example/engines/Data/d2.txt"
    dataType: STRING
    request_id: 1
    sender: "Orchestrator_46"
    composition: ""
    action: CONFIGURE

    sending configure request to 129.57.81.247:abc:F3 with the payload:
    status: INFO
    data: "/Users/gurjyan/Devel/Clara/Clara-p.zmq.0.2/example/engines/Data/d3.txt"
    dataType: STRING
    request_id: 1
    sender: "Orchestrator_46"
    composition: ""
    action: CONFIGURE

    sending configure request to 129.57.81.247:abc:F4 with the payload:
    status: INFO
    data: "/Users/gurjyan/Devel/Clara/Clara-p.zmq.0.2/example/engines/Data/d4.txt"
    dataType: STRING
    request_id: 1
    sender: "Orchestrator_46"
    composition: ""
    action: CONFIGURE

    sending configure request to 129.57.81.247:abc:R with the payload:
    status: INFO
    data: "/Users/gurjyan/Devel/Clara/Clara-p.zmq.0.2/example/engines/Data/d4.txt"
    dataType: STRING
    request_id: 1
    sender: "Orchestrator_46"
    composition: ""
    action: CONFIGURE

For clarity reason orchestrator prints CLARA transient data object, sent to data services. In these printouts you can see that services receive *configure* request indicating the file name to be processed.
Now if you check the consoles of the data services you should see printouts indicating that services were configured. Note that configuration confirmation message is printed 3 times, indicating that all the objects in the service container object pool got configured.

.. code-block:: c

    GOT CONFIGURE REQUEST
    FILE CONTENT = Arri mA

    GOT CONFIGURE REQUEST
    FILE CONTENT = Arri mA

    GOT CONFIGURE REQUEST
    FILE CONTENT = Arri mA

After services are configured we can start or application main orchestrator by passing application design composition:

.. code-block:: c

    >p_compose F1 F2 F3 F4 F1,F2,F3,F4+\&EB+R+F1,F2,F3,F4

Note the application composition string, where we have to properly escape the *&* indicating that 4 inputs of the EB service are logically ANDed.
We started our application that will run for ever as we require in our composition string (F1,F2,F3,F4+\&EB+R+F1,F2,F3,F4). The console printout of the R service is shown below:

.. code-block:: c

    executing engine ...
    A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA
    A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA
    A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA
    A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA     A warm greetings from CLARA ..........






