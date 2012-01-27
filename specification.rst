***************************************************************************************
 PulseWaves Specification (version 0.9)
***************************************************************************************

.. class:: heading4
    
    Principle Investigator: 

Dr. Martin Isenburg

.. class:: heading4

    Contact Address:
    
    martin.isenburg@gmail.com

.. class:: heading4
    
    Title: 

PulseWaves (version 0.9) - no pulse left behind

    Date: 

Dec 23th, 2011

.. raw:: pdf

    SetPageCounter 1 arabic

.. footer::

   Page ###Page###

.. raw:: pdf

    PageBreak

==============================================================================
Introduction
==============================================================================

`PulseWaves`_ is a 

How is this different from LAS?
------------------------------------------------------------------------------

LAS defines a 

Describing layout
..............................................................................

PulseWaves uses the concept 

.. raw:: pdf

    PageBreak

Table Example
------------------------------------------------------------------------------

Full waveform data ...

Example
..............................................................................

Consider the:

.. csv-table:: Basic Pulse
    :header:    "Name", "Data Type", "Byte Size"
    :widths: 70, 10, 10
    
    "X", "long", "4"
    "Y", "long", "4"
    "Z", "long", "4"

Let's define the concept 

::

    Example;
    Code;
    Is.here();

.. raw:: pdf

    PageBreak

Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* The `PulseWaves` format is composed of a `PuLSe` and a `WaVeS` file.

* In addition to the

Another Example
..............................................................................

It is going to be better for ...

::

    class PuLSe
    {
    public:
        PuLSe();
    private:
        // Magic
    };
    
    More.example();
    Code;
    Is.here();

.. raw:: pdf

    PageBreak

Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``PuLSe`` requires ...

* Knowledge of how to make ...


LAS 1.2 POINT10
..............................................................................

Here is an example that defines a typical LAS  POINT10.

.. csv-table:: LAS 1.2 POINT10
    :header:    "Name", "Data Type", "Byte Size", "Bit Size"
    :widths: 70, 10, 10, 10
    
    "X", "int32_t", "4","0"
    "Y", "int32_t", "4","0"
    "Z", "int32_t", "4","0"
    "Intensity", "uint16_t", "2", "0"
    "Return Number", "uint8_t", "0", "3"
    "Number of Returns","uint8_t", "0", "3"
    "Scan Direction Flag", "uint8_t", "0", "1"
    "Edge of Flight Line", "uint8_t", "0", "1"
    "Classification", "uint8_t", "1", "0"
    "Scan Angle Rank" "int8_t", "1", "0"
    "User Data", "uint8_t", "1", "0"
    "Point Source ID", "uint16_t", "2", "0"

.. raw:: pdf

    PageBreak

Some Object
------------------------------------------------------------------------------

PulseWaves currently defines 

1) Pulse ...

2) Waves ...
  
   ::
   
         Pulse pulse;
         // initialize throws in the case of an error
         pulse.initialize();

3) Other stuff ...

   ::
  
         Waves waves.header = pulse.header();
        
         for (unsigned i = 0; i < count(); ++i)
         {
             std::cout << "name: " << w.name() << " size: " << w.size() << std::endl;
         }

* 

.. _`LASzip`: http://laszip.org
.. _`ASPRS LAS`: http://www.asprs.org/a/society/committees/lidar/lidar_format.html

==============================================================================
References 
==============================================================================

.. [#] LASzip: lossless compression of LiDAR data http://lastools.org/download/laszip.pdf
