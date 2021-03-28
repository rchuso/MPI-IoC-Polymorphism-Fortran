Modern Fortran with Polymorphism, CMake, MPI, Inversion of Control
=============

This repository is a subset of my other repo- that also includes HDF5. Some people have had problems building the MPI and HDF5 environment to work with Fortran, so this group of files doesn't include anything of the HDF5. It's still a reasonable repository that can be a springboard for someone who is developing an application in Fortran that uses CMake and MPI following the software pattern Inversion of Control.

This CMake project builds a modern Fortran executable that uses MPI.


Requirements
-------------

The Fortran requires a recent compiler. I used 7.3 (or .4) for the development. Many of the modern features are not available in versions prior to gfortran-6.
You will also need a relatively recent version of OpenMPI (there is a very slight difference in how to reference the polymorphic base class when using mpich - I've successfully used both, and the code in MsgBase.f90 declares the integer variable "baseId" which is required by OpenMPI but not by mpich, but all my recent testing has been with OpenMPI). When the packet is sent with the MPI_SEND function, this variable is referenced for OpenMPI, but with mpich the object itself is referenced (as it should be - the OpenMPI is deficient in this respect relative to implementations in C/C++ or Python).

You will also need a relatively recent version of CMake. I developed this with CMake version 3.10, but any recent version should work with Fortran.


Running
-------------

Build the makefiles with

     cmake .

Then build the executable with

     make

Once the executable is built, you can run it like any other MPI application:

     mpiexec -n \`nprocs\` bin/TestMpiIocHdf5

If all worked correctly, you should see something like this:

     mpiexec -n \`nproc\` bin/TestMpiIoc
     Capabilities (to examine, process, use):
        1 MCT:: rank=0 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        2 MCT:: rank=1 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        3 MCT:: rank=2 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        4 MCT:: rank=3 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        5 MCT:: rank=4 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        6 MCT:: rank=5 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        7 MCT:: rank=6 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        8 MCT:: rank=7 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        9 MCT:: rank=8 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        10 MCT:: rank=9 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        11 MCT:: rank=10 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        12 MCT:: rank=11 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        13 MCT:: rank=12 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        14 MCT:: rank=14 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        15 MCT:: rank=15 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
        16 MCT:: rank=13 size=16 datatype=75 RAM free:111834 total:128445 cores:16 Irigal
     1:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     2:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     3:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     4:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     5:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     7:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     8:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     9:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     10:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     11:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     12:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     13:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     14:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     0:16 rx:MXT:: myPi=2.718 datatype=76 from:3 tag:37
     6:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37
     15:16 rx:MXT:: myPi=2.718 datatype=76 from:0 tag:37


The Application and Structure
-------------

The main Fortran program is TestMpiIoc.f90, and it is in the "code/apps" folder. All the code is under the "code" folder, and I do that to simplify the directory structure. The CMake script creates the "bin", "mod", and "lib" folders. The executable is put in the "bin" folder, the intermediate Fortran module files go in the "mod", and the created libraries go in the "lib" folder.

Under the "code" folder is the "apps" folder which holds the main program(s). There is also a sub-folders in "code" that where the source code resides for the library called "libMpi".

The "libMpi" library wraps MPI with inversion-of-control, which I've used with great success (well, the earlier C version anyway) to create a suite of seismic data processing applications that run on the world's largest privately-owned supercomputers (like the one owned by Total - total.com). The code here is a completely new creation, and probably even faster than the earlier versions, but I don't have access to a supercomputer to test that claim. (This one removes the need to have a small packet for use within the *layer* to coordinate the message traffic.)


MPI
-------------

Applications that want to make use of the standard software paradigm "inversion-of-control" (or IoC), are different from what most people think of when writing MPI applications. It's my experience that scientists who write MPI applications, or even compiler and library designers, think the application should divide the work and send that to the worker nodes. I've written this IoC layer to allow the worker nodes to request data when they're ready, instead of having the control node (rank=0) divide and distribute the workload. In doing so, the workers request work when they have capacity, and the data transmission is more chaotically split than if the work is sent out all at once. This allows the application to adjust dynamically to the workload or even other applications that may also be using the computer. In fact, this works so well, I've seen my applications crowd out other poorly-written MPI applications, while mine keeps the CPU on that node at close to 100%. Testing one of these applications on 2000 nodes showed a linear increase over smaller tests for the Kirchhoff 3D Time Migration application. I understand some of these applications have been run on 86000 nodes (that's eighty six thousand nodes). It also allows me to change the functionining of any node by simply sending that node a message telling it what to do.

This layer allows any node on any core or CPU to send a data packet to any node at any time without prior coordination. This greatly simplifies the creation of MPI applications, as each node only needs to respond to received messages as if the application had a single State and reacted to incoming messages. When the application starts the IoC layer, the layer gets the capabilities of the node on which it's running and sends this to the control node (rank=0) for storage and reference. This allows the control node to initially designate the functioning of other nodes: if 16 nodes report the same hostname (for the SRME3D application for instance) I designate one of them as a file reader and data buffer and the others of that same name as users of the data from the file reader. This keeps their local MPI communications off the backplane (no TCP message, the MPI layer simply performs a memory data copy of the message), and stops that potential interferrence on the network, while keeping the number of file pointers in use under control (number of file readers is configurable on most systems, but it's not efficient to have too many of them or to have multiple nodes reading from the same input file). I wrote a bit more about this on my mjollnir.com web site, and there is a recent version of the C code as well.

The test application also comes with a test message, which is transferred between the nodes. On one relatively arbitrary message receipt, the application stops the IoC layer, and control is returned to the main program.



Disclaimer
-------------

I'm certainly not an expert on all these fields, and I'm definitely open to suggestions - send me a message with your suggestions or corrections.
This sort of data processing is also not suitable for some sorts of problems - which is why it's important to consult with an actual (reputably certified) Software Engineer or Architect when making applications that need MPI and lots of time running on expensive hardware.
The requirement for one of the applications made using this technology was to process a small dataset (only 5 TB of seismic data) on a relatively small network (100 cores) in one week (168 hours on the clock). Many of the applications that use this technology ingest significantly more data, use substantially more cores, and run from much longer periods.
YMMV

