.. raw:: pdf

    SetPageCounter 1 arabic

.. footer::

   This is the official PulseWaves document. It describes an open, stand-alone, vendor-neutral, geo-referenced, LAS-compatible specification for full waveform LiDAR data.

   Page ###Page###

Date: 

draft created on Dec 23th, 2011

***************************************************************************************
 PulseWaves - Full Waveform LiDAR Specification (version 0.1)
***************************************************************************************

.. class:: heading4
    
This document describes an open standard specification for storing full waveform LiDAR data called PulseWaves. It is distributed for the purpose of discussing, evaluating, and brain-storming the PulseWaves format in its initial 1.0 version.  The current draft is expected to significantly change before the actual 1.0 version is released. One of the design goals is to remain forward compatible and allow for changing demands such as extra or different fields in the data records without breaking PulseWaves readers that only implement an older version.

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
    :header: "Item", "Format", "Size"
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

  x_{coordinate} = (X_{record} \* x_{scale}) + x_{offset}

  y_{coordinate} = (Y_{record} \* y_{scale}) + y_{offset}

  z_{coordinate} = (Z_{record} \* z_{scale}) + z_{offset}

Max and Min X, Y, Z:
  The max and min fields describe the bounding box that includes the start and end points of the sampled parts of the returning waveforms of all pulses.

Variable Length Records (VLRs):
------------------------------------------------------------------------------

The PuLSe Header can be followed by any number of Variable Length Records (VLRs). The number of VLRs is specified in the “Number of Variable Length Records” field in the PuLSe Header. The Variable Length Records must be accessed sequentially since the size of each Variable Length Record is contained in the Variable Length Record Header.  Each Variable Length Record Header is 64 bytes in length. 

.. csv-table:: Variable Length Records (VLRs)
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "User ID", "char[16]", "16 bytes"
    "Record ID", "unsigned long", "4 bytes"
    "Reserved[4]", "unsigned char", "4 bytes"
    "Record Length After Header", "long long", "8 bytes"
    "Description", "char[32]", "32 bytes"

User ID:
  The User ID field of ASCII characters identifies the user which created the Variable Length Record. If the character data is less than 16 characters, the remaining data must be null. The User ID "PulseWaves_Spec" is reserved. The User IDs "LASF_Spec and "LASF_Projection" from the LAS 1.4 specification are also reserved.

Record ID:
  The Record ID allows to distinuish different VLRs with the same User ID. The Record IDs for the User ID "PulseWaves_Spec" are reserved. Publicizing the meaning of a Record ID is left to the owner of the given User ID. 

Reserved:
  Must be zero.

Record Length after Header:
  The record length is the number of bytes for the record after the end of the standard part of the header. The entire record length is 64 bytes (the header size of the VLR) plus the Record Length after Header.

Description:
  Optional, null terminated text description of the data. Any remaining characters not used must be null.

Appended Variable Length Records (AVLRs):
------------------------------------------------------------------------------

The Pulse Records are followed by Appended Variable Length Records (AVLRs). The AVLRs are in spirit just like the VLRs but carry their payload "in front" of the footer that desribes them. They are accessed sequentially in reverse starting from the end of the file. There is at least one mandatory AVLR that indicates the end of the AVLR array. Because the AVLRs are accessed in reverse this mandatory AVLR is the first AVLR after the pulse records. The number of AVLRs is specified in the “Number of Appended Variable Length Records” field in the PuLSe Header. Setting this number to a negative value (e.g. -1) means that their number is not known but must be discovered by parsing the AVLRs starting from the end of the file. 

.. csv-table:: Appended Variable Length Records (AVLRs)
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "User ID", "char[16]", "16 bytes"
    "Record ID", "unsigned long", "4 bytes"
    "Reserved[4]", "unsigned char", "4 bytes"
    "Record Length Before Footer", "long long", "8 bytes"
    "Description", "char[32]", "32 bytes"

Pulse Records:
------------------------------------------------------------------------------

All records must be the same type. Unused attributes must be set to the equivalent of zero for the respective data type (e.g. 0.0 for floating-point numbers, NULL for ASCII, 0 for integers). The pulse record format 0 expresses the pulse as an anchor point plus direction vector.

.. csv-table:: Pulse Record Type 0
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "GPS time", "double (or long long)", "8 bytes"
    "Offset to WaVeSamples", "long long", "8 bytes"
    "X_A", "long", "4 bytes"
    "Y_A", "long", "4 bytes"
    "Z_A", "long", "4 bytes"
    "dx", "float", "4 bytes"
    "dy", "float", "4 bytes"
    "dz", "float", "4 bytes"
    "First Returning Sample [sampling units]", "short", "2 bytes"
    "Last Returning Sample [sampling units]", "short", "2 bytes"
    "Index of Pulse Description Record", "14 bits (bit 0-13)", "14 bits"
    "Edge of Flight Line", "1 bit (bit 14)", "1 bit"
    "Scan Direction", "1 bit (bit 15)", "1 bit"

GPS time:
  The GPS time at which the laser pulse was fired. For compatibility with LAS 1.4 this field will usually store either the GPS week time or the Adjusted Standard GPS time as a double-precision floating point number. This is specified by the global encoding bits in the PuLSe header.

Offset to WaVeSamples:
  The offset in bytes from the start of the WaVeS file to the samples of the waveform. How the pulse is sampled is described in the indexed "Pulse Description Record".

X_A, Y_A, and Z_A:
  The anchor point of the pulse. Scaling and offseting the integers X_A, Y_A, and Z_A with scale and offset from the header gives the actual coordinates in world coordinates. The anchor point equals the location of the scanner's optical origin at the time the laser was fired, if the "Offset from Optical Center to Anchor Points" field of the "Pulse Description Record" is zero.

  x_{anchor} = (X_A \* x_{scale}) + x_{offset}

  y_{anchor} = (Y_A \* y_{scale}) + y_{offset}
 
  z_{anchor} = (Z_A \* z_{scale}) + z_{offset}

dx, dy, and dz:
  The pulse direction vector is scaled to the length of units in the chosen world coordinate system (e.g. meters for UTM, decimal degrees for long/lat, feet or survey feet for US stateplane reference systems) that the laser pulse travels in one (1) picosecond away from the origin (e.g. towards the ground in an airborne survey).

First Returning Sample:
  The duration in sampling units from the anchor point to the first recorded waveform sample. Together with the "Sample Units" value from the corresponding "Pulse Description Record" this value allows computing the x/y/z world coordinates of the first intensity sample that was recorded for the returning waveform of this pulse:

  x_{first} = x_{anchor} + first_returning_sample \* sample_units * dx

  y_{first} = y_{anchor} + first_returning_sample \* sample_units * dy

  z_{first} = z_{anchor} + first_returning_sample \* sample_units * dz

Last Returning Sample:
  Same concept as the "First Returning Sample" but for the last one:

  x_{last} = x_{anchor} + last_returning_sample \* sample_units * dx

  y_{last} = y_{anchor} + last_returning_sample \* sample_units * dy

  z_{last} = z_{anchor} + last_returning_sample \* sample_units * dz

Index of Pulse Description Record:
  The record ID of the "PulseWaves_Spec" VLR or AVLR containing a description of this laser pulse and the exact details how its waveform is sampled in form of a "Pulse Description Record". Up to 16,384 different descriptions can be  specified.

Scan Direction Flag:
  This bit remains the same as long as pulses are output with the mirror of the scanner travelling in the same direction or as long as they are reflected from the same mirror facet of the scanner. It flips whenever the mirror direction or the facet changes.

Edge of Flight Line:
  This bit has a value of 1 when the output pulse is at the end of a scan line. It is the last pulse before the scanning hardware changes direction, mirror facet, or zigs back.


Defined Variable Length Records (VLRs or AVLRs):
------------------------------------------------------------------------------

The "LASF_Projection" VLR from LAS 1.4 can be used to geo-reference the pulse file. The "LASF_Proj" VLR "Extra Bytes" from LAS 1.4 can be used to specify extra attributes per pulse.

First Appended Variable Length Record:
------------------------------------------------------------------------------

User ID:     		PulseWaves_Spec
Record ID: 			4,294,967,295 (0xFFFFFFFF)
Record Length Before Footer:	0

This empty AVLR record *MUST* directly follow the pulse records and it must be the first AVLR in case there are multiple AVLRs. It does not carry a payload but is used to mark the end of the appendable list of AVLRs. This is needed as the exact number of AVLRs may not be specified in the header and needs to be discovered by parsing all AVLRs starting at the end of the file until this one is readed. This Record ID makes no sense when used with an VLR. 

Pulse Description Records:
------------------------------------------------------------------------------

User ID: 	PulseWaves_Spec
Record ID: 	n

Where 100,000 <= n < 116,384

The Pulse Description Records describes the scanner system that the pulse originates from and the sampling(s) of the pulse's outgoing and/or returning waveform(s). For example, the outgoing waveform with 32 samples and the returning waveform with 256 samples. Waveforms can also be sampled with multiple sensors. For example, the outgoing waveform with 40 samples and the returning waveform with two sensors of different sensitivity both at 480 samples. Waveforms can also be sampled with multiple discontinuous segments. For example, three successive segments for the returning waveforms, the first with 80, the second with 160, and the last with 80 samples, ... etc.

.. csv-table:: Pulse Description Record 
    :header: "Item", "Units", "Format", "Size"
    :widths: 70, 10, 10, 10

    "Version", "-", "unsigned char", "1 byte"
    "Reserved, "-", "unsigned char[7]", "7 bytes"
    "Offset from Optical Center to Anchor Points", "[picoseconds]", "long long", "8 bytes"
    "Sample Units", "[picoseconds]", "unsigned long", "4 bytes"
    "Offset To Sampling Description Array", "[bytes]", "unsigned long", "4 bytes"
    "Number of Sampling Descriptions", "-", "unsigned long", "4 bytes"
    "Size of Sampling Description Records", "[bytes]", "unsigned long", "4 bytes"
    "Description", "-", "char[32]", "32 bytes"
    "Laser Scanner ID", "-", "unsigned long", "4 bytes"
    "Wavelength of Laser", "[nanometer]", "unsigned long", "4 bytes"
    "Outgoing Pulse Width", "-", "[picometer]", "unsigned long", "4 bytes"
    "Beam Diameter at Exit Aperture", "[micrometers]", "unsigned long", "4 bytes"
    "Beam Divergance", "[microradians]", "unsigned long", "4 bytes"
    "Sampling Description Records[n]", "-", "struct of size m", "n*m bytes"

Version:
  Must be zero.

Reserved:
  Must be zero.

Offset from Optical Center to Anchor Points:
  Specifies a constant temporal offset in picoseconds between the optical center and the anchor point. If the value is 0, anchor point and optical center coincide. Otherwise the optical center of a pulse can be found by "walking" backwards from its anchor point as many units of its direction vector as specified here (a conversion step may be necessary in case that anchor point and direction vector are not in a Euclidean coordinate system). If the value is  0xFFFFFFFFFFFFFFFF there is no constant temporal offset between the optical center and the anchor point. In this case the optical center cannot be "reached" from the anchor point by "walking" a constant mutliple of the direction vector.

Sample Units:
  Specifies the temporal unit of sampling in picoseconds that is used in the Pulse Records for specifying the "First Returning Sample" and the "Last Returning Sample".

Offset to Sampling Description Array:
  The offset in bytes from the start of the Pulse Description Record to the first "Sampling Description Record" of the "Sampling Description Records[n]" array. PulseWaves readers should use this value to seek to the first "Sampling Description Record" of the "Sampling Description Records [n]" array because later versions of the PulseWaves specification may insert additional fields after "Beam Divergence".

Number of Sampling Descriptions:
  A value larger than 0 specifying the number of "Sampling Description Records" start at the byte indicated by the "Offset to Samplings Array" field. 

Size of Sampling Description Records:
  A value that specifies the size of each of the "Sampling Description Records" that start at the byte indicated by "Offset to Sampling Description Array" field.  PulseWaves readers should use this value to seek forward to the next "Sampling Description Record" as later versions of the PulseWaves specification may enlarge each "Sampling Description Record" by adding new fields at the end.

Description:
  Optional, null terminated text description of the data.  Any remaining characters not used must be null.

Laser Scanner ID:
  In case there are several laser scanning units that are simultaneously storing their output to the same PulseWaves file. They can be then be distinguished by assigning their respective pulse descriptions a different ID. The default is 0.

Wavelength of Laser:
  The physical wavelength of the laser in nanometers.

Outgoing Pulse Width:
  The width of the outgoing pulse in picometer. The exact width and intensity tends to vary from pulse per pulse which is why the outgoing waveform is often sampled as well.

Beam Diameter at Exit Aperture:
  The diameter of the laser beam in micrometer in the moment it leaves the scanner hardware.

Beam Divergance:
  The divergance of the laser beam in microradians [urad] @ 1/e2. [or should we use @ 1/e]?

Sampling Description Records:
  An array of Sampling Description Records as described in Table XXX.




The rest of the document is gibberish ...
------------------------------------------------------------------------------

`PulseWaves`_ is a 

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
