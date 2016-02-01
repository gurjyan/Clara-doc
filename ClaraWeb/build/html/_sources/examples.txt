
*************
Code Examples
*************

Examples presented in this section of the document are in Python. Document will be expanded to show Java examples soon.
.. _monitor_example:

Discover a service
==================

This example shows how to contact *Registrar* service and request particular service description
by specifying the canonical name of a service, requesting the canonical name of an registered service by specifying
the engine name of a service, list of all services deployed in a specified service container canonical
name, and asking the names of all services running is specified DPE/host.

.. code-block:: c

    from core.xMsgUtil import xMsgUtil
    from src.base.OrchestratorBase import OrchestratorBase

    __author__ = 'gurjyan'


    class ServiceFinder(OrchestratorBase):
        def __init__(self):
            OrchestratorBase.__init__(self)

        def start(self):

            while True:
                xMsgUtil.sleep(1)
                print "acceptable commands: getServiceByName, getServiceByEngineName, " \
                      "getServicesByContainer, getServicesByHost"
                cmd = raw_input("enter a command:")
                if cmd == "getServiceByName":
                    service_name = raw_input("what is the canonical name of the service?")
                    lr_data = self.find_service(service_name)
                    print lr_data[0]

                elif cmd == "getServiceByEngineName":
                    engine_name = raw_input("what is the service engine name?")
                    lr_data = self.get_service_by_engine(engine_name)
                    for d in lr_data:
                        print d

                elif cmd == "getServicesByContainer":
                    container_name = raw_input("what is the canonical name of the container?")
                    lr_data = self.get_service_by_container(container_name)
                    for d in lr_data:
                        print d

                elif cmd == "getServicesByHost":
                    host_name = raw_input("what is the IP of the host?")
                    lr_data = self.get_service_by_host(host_name)
                    for d in lr_data:
                        print d

    def main():
        o = ServiceFinder()
        o.start()


    if __name__ == '__main__':
        main()


.. _orchestrator_example:

Simple application design
=========================

This orchestrator uses deployed engine names to construct and start service based application. It assumes that all
services are running on a local DPE and that the name of a service container is known. This example is not
using CLARA lookup and discovery services.

Input parameters to the constructor of this orchestrator will define the name engine name of the initiator service,
i.e. the first service in the composition, the container name where all the services of the composition are deployed,
data size of the byte array that will be sent to the first service, and the string of the CLARA application composition.
Using base class methods orchestrator substitutes engine names in the user provided composition with proper
canonical names. Then it defines the CLARA transient data object by filling appropriate fields,
including application composition string. After transient data is created it sends to the first service
in the composition chain. Now the application is started and service execution schema will follow data flow
diagram defined and presented to each service in the composition as part of the request. Each service receiving the
request will know where the from the request is coming and where it should direct the result of its engine execution.
Thus entire service based application algorithm is packed in the composition string of the transient data object and
is presented to each service. It is important tp mention that parsing (compiling) of the composition is done only once
and will be recompiled only if the composition is changed. Due to the fact that the same service can be part of one or
many applications (with different compositions) the service will parse the composition string in case of a new
application request. This behaviour must be considered during an application performance optimization studies.

.. code-block:: c

    import sys
    from core.xMsgConstants import xMsgConstants
    from data import xMsgData_pb2
    from src.base.OrchestratorBase import OrchestratorBase


    __author__ = 'gurjyan'


    class PatternTester(OrchestratorBase):
        """
        This orchestrator is design to test Clara patterns.
        It assumes that services are deployed on the local DPE

        constructor accepts 4 parameters

            Note: all services are assumed to run on a local
                  host and have the same container name

            1) the name of the first service engine in the service chain
            2) data size in bytes
            3) actual application composition,
               e.g. s1+s2+s3+s4 or s1,s2,s3+s4
               Note: using engine names only. Actual service names
                     will constructed using the local host and defined
                     container name.
        """
        initiator_engine = xMsgConstants.UNDEFINED
        data_size = 0
        composition = xMsgConstants.UNDEFINED

        def __init__(self, name, data_size, composition):
            OrchestratorBase.__init__(self)
            self.initiator_engine = name
            self.data_size = int(data_size)
            self.composition = composition

        def start(self):

            assert isinstance(self.initiator_engine, str)

            l = self.get_service_by_engine(self.initiator_engine)

            if l[0] == xMsgConstants.NO_RESULT:
                print "Engine = " + self.initiator_engine + " is not registered as a service"
                return
            else:
                service_1 = l[0]

            # recreate composition string by substituting engine
            # names with proper service canonical names
            _cmd = self.engine_to_composition(self.composition)

            # define transient data
            tr = xMsgData_pb2.Data()
            tr.sender = self.get_my_name()
            tr.id = 1
            tr.action = xMsgData_pb2.Data.EXECUTE
            tr.composition = _cmd
            # creating a byte buffer using a character as a byte
            tr.BYTES = 'v' * self.data_size
            # tr.data = str(randint(1, 100))
            tr.dataType = xMsgData_pb2.Data.T_BYTES
            tr.dataGenerationStatus = xMsgData_pb2.Data.INFO

            print "sending request to " + service_1.name + " with the payload: \n"
            print tr

            self.send(service_1.name, tr)


    def main(name, d_size, cmd):
        orc = PatternTester(name, d_size, cmd)
        orc.start()


    if __name__ == '__main__':
        main(sys.argv[1], sys.argv[2], sys.argv[3])


.. _sentence_example:

Sentence builder
================

This example is a little bit more entertaining. There are 6 service involved in this exercise. The first 4 services
play the role of a data-source (F1-F4). At the configure request these services open and read into the memory the content
of a requested/given file, containing some of letters of a given sentence that wae spread between 4 files in a round robin fashion.
Here is the example of a service engine, implementing *configure* and *execute* methods of the  Clara engine abstract
class.

.. code-block:: c

    def configure(self, x):
        print "GOT CONFIGURE REQUEST"
        f = open(str(x.data), "r+b")
        self.f_content = f.readline()
        self.c_count = 0
        f.close()

    def execute(self, x):
        print "INPUT SERVICE .....> " + x.sender + " " + self.f_content
        if self.c_count < len(self.f_content):
            x.data = self.f_content[self.c_count]
            self.c_count += 1
        else:
            self.c_count = 0
            x.data = " "
        print "SENDING DATA = "+x.data
        return x

These data-source services will send their data letter by letter to the the event builder service, that will
recover sentence 4 letter at a time, and send the subset of the build sentence to the recorder service (R).
Below is the diagram of the sentence builder application. Note that event-builder (EG) service inputs are
logically ANDed, i.e. EG engine will be execute only if all the inputs are received.

 .. figure:: /_static/pictures/Slide08.jpg
    :width: 600px
    :align: center
    :height: 400px
    :alt: alternate text
    :figclass: align-center

Since EB is a multi-input gate where inputs are logically ANDed, engine of this service must implement
*execute_group* abstract method of the CLARA engine class.

.. code-block:: c

    def execute_group(self, x):
        print "MULTI-INPUT DATA .....> "
        d = ["", "", "", ""]
        for k in x:
            if "F1" in k.sender:
                d[0] = k.data
            elif "F2" in k.sender:
                d[1] = k.data
            elif "F3" in k.sender:
                d[2] = k.data
            elif "F4" in k.sender:
                d[3] = k.data
        s = ""
        for y in d:
            s += y
        x[0].data = s
        return x[0]

Data recorder service does very little by simply printing the received data.

.. code-block:: c

     def execute(self, x):
        self.result = self.result + x.data
        print self.result
        return x

And finally the orchestrator code below that first asks Discovery and Registry services to locate services
of interest. Here orchestrator checks only starting services, but in reality it should check
all service in the composition.Note that CLARA canonical name of a service (containing complete address of
a service) will be returned to the orchestrator, in the case services were properly registered with the platform.
And the rest is simply creating CLARA transient data object (lines 47-55) by setting composition field = command-line
provided string of the composition: F1,F2,F3,F4+\&EB+R+F1,F2,F3,F4

.. code-block:: c

    import sys
    from core.xMsgConstants import xMsgConstants
    from core.xMsgUtil import xMsgUtil
    from data import xMsgData_pb2
    from src.base.OrchestratorBase import OrchestratorBase

    __author__ = 'gurjyan'


    class Composer(OrchestratorBase):

        engine1 = xMsgConstants.UNDEFINED
        engine2 = xMsgConstants.UNDEFINED
        engine3 = xMsgConstants.UNDEFINED
        engine4 = xMsgConstants.UNDEFINED
        composition = xMsgConstants.UNDEFINED

        def __init__(self, e1, e2, e3, e4, composition):
            OrchestratorBase.__init__(self)
            self.engine1 = e1
            self.engine2 = e2
            self.engine3 = e3
            self.engine4 = e4
            self.composition = composition

        def start(self):

            # create service canonical name from the engine
            l = self.get_service_by_engine(self.engine1)
            if l[0] == xMsgConstants.NO_RESULT:
                print "Engine = " + self.engine1 + " is not registered as a service"
                return
            else:
                service_1 = l[0]

            l = self.get_service_by_engine(self.engine2)
            if l[0] == xMsgConstants.NO_RESULT:
                print "Engine = " + self.engine2 + " is not registered as a service"
                return
            else:
                service_2 = l[0]

            l = self.get_service_by_engine(self.engine3)
            if l[0] == xMsgConstants.NO_RESULT:
                print "Engine = " + self.engine3 + " is not registered as a service"
                return
            else:
                service_3 = l[0]

            l = self.get_service_by_engine(self.engine4)
            if l[0] == xMsgConstants.NO_RESULT:
                print "Engine = " + self.engine4 + " is not registered as a service"
                return
            else:
                service_4 = l[0]

            # recreate composition string by substituting engine
            # names with proper service canonical names
            _cmd = self.engine_to_composition(self.composition)

            # define transient data
            tr = xMsgData_pb2.Data()
            tr.sender = self.get_my_name()
            tr.id = 1
            tr.action = xMsgData_pb2.Data.EXECUTE
            tr.composition = _cmd
            # creating a byte buffer using a character as a byte
            tr.STRING = 'v'
            tr.dataType = xMsgData_pb2.Data.T_STRING
            tr.dataGenerationStatus = xMsgData_pb2.Data.INFO

            print "sending request to " + service_1.name + " with the payload: \n" + str(tr)
            self.send(service_1.name, tr)
            xMsgUtil.sleep(0.01)
            print "sending request to " + service_2.name + " with the payload: \n" + str(tr)
            self.send(service_2.name, tr)
            xMsgUtil.sleep(0.01)
            print "sending request to " + service_3.name + " with the payload: \n" + str(tr)
            self.send(service_3.name, tr)
            xMsgUtil.sleep(0.01)
            print "sending request to " + service_4.name + " with the payload: \n" + str(tr)
            self.send(service_4.name, tr)


    def main(e1, e2, e3, e4, cmd):
        orc = Composer(e1, e2, e3, e4, cmd)
        orc.start()

    if __name__ == '__main__':
        main(sys.argv[1], sys.argv[2], sys.argv[3], sys.argv[4], sys.argv[5])



