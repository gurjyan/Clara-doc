
*****************
Service Threading
*****************

Currently multi-core processors are the trend in hardware design. To satisfy ever-growing demand for computing power the obvious choice was to increase processor clock speed. The situation based on this choice did however end a few years back, and in the last couple of years clock speeds were stabilized at around 2 to 3 GHz. The main reason for that is the power, which increases non-linearly with the clock frequency, making processor cooling quit challenging. So, a new approach was adapted to improve performance by increasing number of processing cores per chip. So far this approach keeps hardware evolution with the Moore’s law, i.e. the number of cores per chip is doubled every 18-24 months. The process of hardware parallelization, being it multi-core or multiprocessor systems, will continue evolution in foreseeable future.

Data processing parallelizeation is an essential design choice for Big data handling. However, software industry is lagging behind the hardware evolution. Some of the high level programming languages are not supporting or offering very coarse multi-threading implementations. Multi-threading in general is difficult to program and scale. The main source of the difficulty is the shared resource handling between threads. CLARA approach is to minimize resource dependencies between services, that makes multi-threading and scaling straight forward.  CLARA design solution is to pass information as a message rather than share states and resources. In another words CLARA service container is thread-native, where almost all resources are private.  This makes scaling of an application trivial (more instances of threads running copies of services). The only unknown is the user engine, that is going to be used in the service multi-threaded environment. Hence, the thread safety of the service-engine is the critical requirement. We hope that by minimizing mutex locks, context switching, synchronization, etc., CLARA services can run at a full native speed and make full use of a core. ZeroMQ library (used in xMsg) being single threaded perfectly fits into the CLARA architecture.


.. figure:: /_static/pictures/Slide07.jpg
    :width: 600px
    :align: center
    :height: 400px
    :alt: alternate text
    :figclass: align-center

    Figure 14. Clara Service multi-threading

In this particular implementation, depicted in the Figure 14, the only resource that requires synchronization is the object pool of the *service* engine and *engine* object in the *service-engine*. Everything else is private to the service-engine object, including zeroMQ socket connections, their persistency, composition analyses and auto-routing, etc. Service  will create object pool with the defined pool size, yet object pool will grow in case of number of *simultaneous* requests are larger than the number of available objects in the pool. We define *simultaneous* requests as requests having arrival time difference smaller than the service engine execution time.


