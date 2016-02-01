
***********
User engine 
***********
The CLARA service model assumes that the algorithm, as well as the solution itself is provided as a complete service. This approach is referred to as Software as a Service (SaaS). A CLARA service may be concisely described as a software application (service engine) that is deployed on a CLARA DPE and can be accessed locally as well as globally over the Internet. With the exception of a orchestrator and service interactions with a service engine, all the aspects of a service are abstracted away (including algorithmic solutions, technology, etc.). As was mentioned, CLARA SaaS supports multiple users and provides a shared data model through a single-instance, known as a multi-tenancy model (i.e. services that are shared between multiple data processing applications).  So, the use of the multi-tenancy model in the CLARA SaaS implementation dictates the only requirement to service engine: the service engine must be thread enabled or thread safe.

.. _Software-service:

Software as a CLARA service
===========================

In order to present software as a CLARA service we need 2 things:

*	Understanding of the CLARA transient data envelope

*	Implementing of the CLARA service interface (or inherit from the CLARA service abstract class)

The CLARA transient data envelope is described in the previous chapter. 
User provided service engines must extend CLARA interface **Engine**.

.. code-block:: c
	
	public interface Engine {
	
    	EngineData configure(EngineData input);
	
    	EngineData execute(EngineData input);
	
    	EngineData executeGroup(Set<EngineData> inputs);
	
    	Set<EngineDataType> getInputDataTypes();
	
    	Set<EngineDataType> getOutputDataTypes();
	
    	Set<String> getStates();
	
    	String getDescription();
	
    	String getVersion();
	
    	String getAuthor();
	
    	void reset();
	
    	void destroy();
	}	
.. _engine_class_example:

Engine class example
====================

Engine class example below are for illustration purposes only and do not have any practical use, even though it can be used to generate data volume.

.. code-block:: c

	public class E1 implements Engine {
    	private long nr = 0;
    	private long t1;
    	private long t2;
	
    	@Override
    	public EngineData execute(EngineData x) {
        	if (nr == 0) {
            	t1 = System.currentTimeMillis();
        	}
        	nr = nr + 1;
        	if (nr >= CConstants.BENCHMARK) {
            	t2 = System.currentTimeMillis();
            	long dt = t2 - t1;
            	double pt = (double) dt / (double) nr;
            	long pr = (nr * 1000) / dt;
            	System.out.println("E1 processing time = " + pt + " ms");
            	System.out.println("E1 rate = " + pr + " Hz");
            	nr = 0;
        	}
        	return x;
    	}
	
    	@Override
    	public EngineData executeGroup(Set<EngineData> x) {
        	System.out.println("E1 engine group execute...");
        	return x.iterator().next();
    	}
	
    	@Override
    	public EngineData configure(EngineData x) {
        	System.out.println("E1 engine configure...");
        	return x;
    	}
	
    	@Override
    	public Set<String> getStates() {
        	return null;
    	}
	
    	@Override
    	public Set<EngineDataType> getInputDataTypes() {
        	return null;
    	}
	
    	@Override
    	public Set<EngineDataType> getOutputDataTypes() {
        	return null;
    	}
	
    	@Override
    	public String getDescription() {
        	return null;
    	}
	
    	@Override
    	public String getVersion() {
        	return null;
    	}
	
    	@Override
    	public String getAuthor() {
        	return null;
    	}
	
    	@Override
    	public void reset() {
    	}

    	@Override
    	public void destroy() {	
    	}
	}