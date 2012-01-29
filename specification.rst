.. raw:: pdf

    SetPageCounter 1 arabic

.. footer::

   This is the official PulseWaves document. It describes an open, stand-alone, vendor-neutral, geo-referenced, LAS-compatible specification for full waveform LiDAR data.

   Page ###Page###

Date: 

Dec 23th, 2011

***************************************************************************************
 PulseWaves - Full Waveform LiDAR Specification (version 0.1)
***************************************************************************************

.. class:: heading4
    
This document describes an open standard specification for storing full waveform LiDAR data called PulseWaves. It is distributed for the purpose of discussing, evaluating, and brain-storming the initial PulseWaves format in the 1.0 version.  The current draft is expected to significantly change before the actual 1.0 version is released. One of the design goals is to remain forward compatible and allow for changing demands such as extra or different fields in the data records without breaking PulseWaves readers that only implement an older version.

The *Pulse* files describe the emitted laser pulses with geo-referenced origin and direction. The *Waves* files describe the outgoing and returning waveform shapes for the relevant sections of these pulses (e.g. in the vicinity of where something was hit). The PulseWaves format is compatible to the LASer format of the ASPRS. These LASer files describe discrete returns with attributes where either the sensor hardware or some post-processing software have computed that something was "hit" by the laser beam. Via the GPS time it is possible to find the PulseWaves that the LASer returns are "attached" to.

==============================================================================
Introduction
==============================================================================

This document describes the simple stand-alone PulseWaves format for storing geo-referenced full waveform data that is compatible with the LAS format and that is just as simple to parse and use. It consists of two binary files: a PuLSe file (\*.pls) and a WaVeS (\*.wvs) file. 

The PuLSe file is stand-alone. It describes a geo-referenced locations and directions of the pulses. For each fired pulse it stores the geo-referenced coordinates of the lasers's optical center and the geo-referenced direction vector of the pulse together with the moment that the first and last sample for the returning waveform was recorded. This file alone is, for example, already sufficient to verify coverage or "sweep out" the scanned 3D space.

The WaVeS file is *not* stand-alone and depends upon the PuLSe file. Each pulse in the PuLSe file contains an offset into the WaVeS file to where the actual digitized samples for the relevant segments of that pulse are stored that describe the shape of the waveform in detail. The format how the waveforms are sampled is kept flexible as each pulse references a sampling description. 

Via the GPS time the pulses in the PuLSe file and their associated WaVeS may (optionally) be linked to the discrete point returns stored in corresponding LASer files and vice-versa.

==============================================================================
The PuLSe file (\*.pls)
==============================================================================

The PuLSe file is a binary file consisting of a Header, any number of optional Variable Length Records (VLRs), the Pulse Records, and one or more Appended Variable Length Records (AVLRs). All data are in little-endian format.

The Header contains generic data such as the version of the file, creation date, the number of pulses, etc. The Variable Length Records (VLRs) and the Appended Variable Length Records (AVLRs) contain various types of additional data, most importantly the description of how the waveform are sampled and the geo-referencing information, but also additional sensor data or user defined data. The AVRLs are essentially just like the VLRs but can be appended at the end of the file. This allows, for example, to add another pulse description, correct the projection information, or include a spatial indexing data structure to an existing PuLSe file without having to re-write the entire file.

.. csv-table:: The PuLSe File Structure 
    :widths: 100

    "Header"
    "Variable Length Records (VLRs)"
    "Pulse Records"
    "Appended Variable Length Records (AVLRs)"

The waveform samples of the pulses that are reported in the Pulse Records are stored in a separate WaVeS file that must be in the same directory and have the same base name as the *.pls file, but have the ending *.wvs. 

.. csv-table:: The PuLSe Header
    :header:    "Item", "Format", "Size"
    :widths: 70, 10, 10
    
    "File Signature (“PuLS”)", "char[4]", "4 bytes"
    "File Source ID", "unsigned long", "4 bytes"
    "Global Encoding", "unsigned long", "4 bytes"
    "Project ID - GUID data 1", "unsigned long", "4 bytes"
    "Project ID - GUID data 2", "unsigned long", "4 bytes"
    "Project ID - GUID data 3", "unsigned long", "4 bytes"
    "Project ID - GUID data 4", "unsigned char[8]", "8 bytes"
    "System Identifier", "char[32]", "32 bytes"
    "Generating Software", "char[32]", "32 bytes"
    "File Creation Day of Year", "unsigned short", "2 bytes"
    "File Creation Year", "unsigned short", "2 bytes"
    "Version Major", "unsigned char", "1 byte"
    "Version Minor", "unsigned char", "1 byte"
    "Header Size", "unsigned short", "2 bytes"
    "Offset to Pulse Records", "long long", "8 bytes"
    "Number of Variable Length Records", "unsigned long", "4 bytes"
    "Number of Appended Variable Length Records", "long", "4 bytes"
    "Pulse Format", "unsigned long", "4 bytes"
    "Pulse Record Size", "unsigned long", "4 bytes"
    "Number of Pulse Records", "long long", "8 bytes"
    "X Scale Factor", "double", "8 bytes"
    "Y Scale Factor", "double", "8 bytes"
    "Z Scale Factor", "double", "8 bytes"
    "X Offset", "double", "8 bytes"
    "Y Offset", "double", "8 bytes"
    "Z Offset", "double", "8 bytes"
    "Max X", "double", "8 bytes"
    "Min X", "double", "8 bytes"
    "Max Y", "double", "8 bytes"
    "Min Y", "double", "8 bytes"
    "Max Z", "double", "8 bytes"
    "Min Z", "double", "8 bytes"

Any field in the PuLSe Header that is not required or that is not used must be zero filled.

File Signature:
  The file signature must contain the four characters “PuLS" that can be checked by user software as a quick look validate the file type.

File Source ID:
  If this file contains the pulses from an original flight line this field should contain the flight line number. A value of zero (0) is interpreted to mean that an ID has not been assigned. 

Global Encoding:
  This is a bit field used to indicate certain global properties about the file.

Project ID (GUID data):
  These four fields describe a Globally Unique Identifier (GUID) for use as a Project Identifier (Project ID). These fields are at the discretion of processing software. They should be the same for all files associated with a unique project. By assigning a Project ID and using a File Source ID for every file within the project, every pulse can be uniquely identified.

Version Number:
  The version number consists of a major and minor field. All minor versions of the same major version will be fully forward and backward compatible.

System Identifier:
  This information is ASCII data describing the hardware sensor that collected or the process that generated the pulse records in this file. If the character data is less than 31 characters, the remaining data must be null.

Generating Software:
  This information is ASCII data describing the generating software itself.  This field provides a mechanism for specifying which generating software package and version was used during PuLSe file creation (e.g. “TerraScan V-10.8”,  “REALM V-4.2”, " RiPROCESS 1.4.16.51", etc.).  If the character data is less than 31 characters, the remaining data must be null.

File Creation Day of Year:
  The day on which this file was created. Day is computed as the Greenwich Mean Time (GMT) day. January 1 is considered day 1.

File Creation Year:
  The year, expressed as a four digit number, in which the file was created.  

Header Size:
  The size, in bytes, of the PuLSe Header itself. For PuLSe 1.0 this size is 224  bytes. If the header is extended through the addition of data at the end of the header by a new revision of the PuLSe specification, the Header Size field will reflect this. 

Offset to Pulse Records:
  The actual number of bytes from the beginning of the file to the first pulse record data field.  This data offset must be updated if any software adds/removes data to/from the Variable Length Records.

Number of Variable Length Records:
  This field contains the current number of VLRs that are stored in the file before the Pulse Records. This number must be updated if the number of VLRs changes.

Number of Appended Variable Length Records:
  This field contains the current number of AVLRs that are stored the file after the Pulse Records. This number should be updated if the number of AVLRs changes. This number may be set to \"-1\", which indicates that the number of AVLRs is not known and must be determined my parsing the AVLRs starting at the end of the file.

Pulse Format:
  The format of the pulse records. In PuLSe 1.0 this is always 0.

Pulse Record Length:
  The size, in bytes, of the Pulse Record. All Pulse Records within a PuLSe file have the same type and hence the same length. If the specified size is larger than implied by the pulse format (e.g. 32 bytes instead of 28 bytes for format 0) the remaining bytes are user-specific “extra bytes”. The meaning of such “extra bytes” can be described with an Extra Bytes VLR (see Table 12 and Table 24) to make them useful to others as well.

Number of Pulse Records:
  This field contains the total number of pulse records within the file.

X, Y, and Z Scale Factors:
  The scale factor fields contain double-precision floating point values used to scale the X, Y, and Z long values of the pulse records. If the actual x, y, z coordinates have two decimal point values, then each scale factor will contain the number 0.01.   

X, Y, and Z Offset:
  The offset fields contain double-precision floating point values used to offset  the X, Y, and Z long values of the pulse records. The formulas shown below convert from the X, Y, and Z long values of each pulse to the actual x, y, z coordinates.

  xcoordinate = (X_{record} \* xscale) + xoffset

  ycoordinate = (Yrecord \* yscale) + yoffset

  zcoordinate = (Zrecord \* zscale) + zoffset

Max and Min X, Y, Z:
  The max and min fields describe the bounding box that includes the start and end points of the sampled parts of the returning waveforms of all pulses.


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
