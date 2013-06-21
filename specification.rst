.. raw:: pdf

    SetPageCounter 1 arabic

.. footer::

   This is the official PulseWaves document. It describes the specification of an open, stand-alone, vendor-neutral, LAS-compatible data exchange format for geo-referenced full waveform LiDAR data.

   Page ###Page###

Date: 

initial draft created on Dec 23th, 2011

***************************************************************************************
 PulseWaves - Full Waveform LiDAR Specification (version 0.3 revision 7)
***************************************************************************************

.. class:: heading4
    
This document describes the *PulseWaves* specification - an open, stand-alone, vendor-neutral, LAS-compatible data exchange format for storing geo-referenced full waveform LiDAR. The document is distributed for the purpose of discussing, evaluating, and brain-storming the PulseWaves format in its current 0.3 version (revision 7). The current draft is expected to be fairly close to the first actual 1.0 version is released. One of the design goals is to remain forward compatible and allow for changing demands such as additional or different fields in the data records without breaking PulseWaves readers that only implement olders version of the specification.

The PulseWaves format consists of two binary files: The *Pulse* files (\*.pls) describe the emitted laser pulses with geo-referenced origin and direction. The *Waves* files (\*.wvs) contain the samples of the outgoing and returning waveform shapes for the relevant sections of these pulses (e.g. in the vicinity of where something was hit). The PulseWaves format is meant to be compatible with the LAS format of the ASPRS. These *Laser* files (\*.las) describe discrete returns with attributes where either the sensor hardware or some post-processing software have computed that something was "hit" by the laser beam. Via the GPS time it is possible to find the PulseWaves that the Laser returns are "attached" to.

==============================================================================
Introduction
==============================================================================

This document describes the simple PulseWaves data exchange format for storing geo-referenced full waveform data that is compatible with the LAS format and just as simple to parse and use. It consists of two binary files: a Pulse file (\*.pls) and a Waves (\*.wvs) file. 

The Pulse file is stand-alone. It describes a geo-referenced locations and directions of the pulses. For each fired pulse it stores the geo-referenced coordinates of the lasers's optical center and the geo-referenced direction vector of the pulse together with the moment that the first and last sample for the returning waveform was recorded. This file alone is, for example, already sufficient to verify coverage or "sweep out" the scanned 3D space.

The Waves file is *not* stand-alone and depends upon the Pulse file. Each pulse in the Pulse file contains an offset into the Waves file to where the actual digitized samples for the relevant segments of that pulse are stored that describe the shape of the waveform in detail. The format how the waveforms are sampled is kept flexible as each pulse references a sampling description. 

Via the GPS time the pulses in the Pulse file and their associated Waves may (optionally) be linked to the discrete point returns stored in corresponding LASer files and vice-versa.

==============================================================================
Standard Data Types
==============================================================================

For better readability of the document we use old-fashioned data type names such as "int" or "unsigned short" instead of "int32_t" or "uint16_t"  to refer to a 32 bit signed integer or a 16 bit unsigned integer. Here a table of the data types used in this document.

.. csv-table:: Standard Data Types
    :header: "Name", "C Standard", "Size in Bytes"
    :widths: 70, 10, 10
    
    "unsigned char", "uint8_t", "1"
    "char", "int8_t", "1"
    "unsigned short", "uint16_t", "2"
    "short", "int16_t", "2"
    "unsigned long", "uint32_t", "4"
    "long", "int32_t", "4"
    "unsigned long long", "uint64_t", "8"
    "long long", "int64_t", "8"
    "float", "float64_t", "4"
    "double", "float64_t", "8"

==============================================================================
The Pulse file (\*.pls)
==============================================================================

The Pulse file is a binary file consisting of a Header, any number of optional Variable Length Records (VLRs), the Pulse Records, and one or more Appended Variable Length Records (AVLRs). All data are in little-endian format.

The Header contains generic data such as the version of the file, creation date, the number of pulses, etc. The Variable Length Records (VLRs) and the Appended Variable Length Records (AVLRs) contain various types of additional data, most importantly the description of how the waveform are sampled and the geo-referencing information, but also additional sensor data or user defined data. The AVRLs are essentially just like the VLRs but can be appended at the end of the file. This allows, for example, to add another pulse description, correct the projection information, or include a spatial indexing data structure to an existing Pulse file without having to re-write the entire file.

.. csv-table:: The Pulse File Structure 
    :widths: 100

    "Header"
    "Variable Length Records (VLRs)"
    "Pulse Records"
    "Appended Variable Length Records (AVLRs)"

The waveform samples of the pulses that are reported in the Pulse Records are stored in a separate Waves file that must be in the same directory and have the same base name as the *.pls file, but have the ending *.wvs. 

.. csv-table:: The Pulse Header
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10
    
    "File Signature (“PulseWavesPulse”)", "char[16]", "16 bytes"
    "Global Parameters", "unsigned long", "4 bytes"
    "File Source ID", "unsigned long", "4 bytes"
    "Project ID - GUID data 1", "unsigned long", "4 bytes"
    "Project ID - GUID data 2", "unsigned short", "2 bytes"
    "Project ID - GUID data 3", "unsigned short", "2 bytes"
    "Project ID - GUID data 4", "unsigned char[8]", "8 bytes"
    "System Identifier", "char[64]", "64 bytes"
    "Generating Software", "char[64]", "64 bytes"
    "File Creation Day of Year", "unsigned short", "2 bytes"
    "File Creation Year", "unsigned short", "2 bytes"
    "Version Major", "unsigned char", "1 byte"
    "Version Minor", "unsigned char", "1 byte"
    "Header Size", "unsigned short", "2 bytes"
    "Offset to Pulse Data", "long long", "8 bytes"
    "Number of Pulses", "long long", "8 bytes"
    "Pulse Format", "unsigned long", "4 bytes"
    "Pulse Attributes", "unsigned long", "4 bytes"
    "Pulse Size", "unsigned long", "4 bytes"
    "Pulse Compression", "unsigned long", "4 bytes"
    "Reserved", "long long", "8 bytes"
    "Number of Variable Length Records", "unsigned long", "4 bytes"
    "Number of Appended Variable Length Records", "long", "4 bytes"
    "T Scale Factor", "double", "8 bytes"
    "T Offset", "double", "8 bytes"
    "Min T", "long long", "8 bytes"
    "Max T", "long long", "8 bytes"
    "X Scale Factor", "double", "8 bytes"
    "Y Scale Factor", "double", "8 bytes"
    "Z Scale Factor", "double", "8 bytes"
    "X Offset", "double", "8 bytes"
    "Y Offset", "double", "8 bytes"
    "Z Offset", "double", "8 bytes"
    "Min X", "double", "8 bytes"
    "Max X", "double", "8 bytes"
    "Min Y", "double", "8 bytes"
    "Max Y", "double", "8 bytes"
    "Min Z", "double", "8 bytes"
    "Max Z", "double", "8 bytes"

Any field in the Pulse Header that is not required or that is not used must be zero filled.

File Signature:
  The file signature must contain the zero-terminated string of 16 characters “PulseWavesPulse" that can be checked by user software as a quick look validate the file type.

Global Parameters:
  This is a bit field used to specify global properties about the file.

File Source ID:
  If this file contains the pulses from a single flight line or a single scan, then this field should contain the corresponding flight line number, the drive path ID, or the scan site identifier. To be meaningful this ID should be non-zero because a File Source ID of zero is meant to imply that the file is result of a "merge" operation and that each pulse should store a (non-zero) Pulse Source ID attribute. 

Project ID (GUID data):
  These four fields describe a Globally Unique IDentifier (GUID) to identify a project. These fields are at the discretion of processing software. They should be the same for all files associated with a unique project. By assigning a Project ID and using a unique Scan Index for every scan of the project, every pulse can be uniquely identified.

System Identifier:
  This information is ASCII data describing the hardware sensor that collected or the process that generated the pulse records in this file. If the character data is less than 64 characters, the remaining data must be null.

Generating Software:
  This information is ASCII data describing the generating software itself.  This field provides a mechanism for specifying which generating software package and version was used during Pulse file creation (e.g. “TerraScan V-10.8”,  “REALM V-4.2”, " RiPROCESS 1.4.16.51", etc.).  If the character data is less than 64 characters, the remaining data must be null.

File Creation Day of Year:
  The day on which this file was created. Day is computed as the Greenwich Mean Time (GMT) day. January 1 is considered day 1.

File Creation Year:
  The year, expressed as a four digit number, in which the file was created.  

Version Number:
  The version number consists of a major and minor field. All minor versions of the same major version will be fully forward and backward compatible.

Header Size:
  The size, in bytes, of the Pulse Header itself. In the current version this is 352 bytes. If the header is extended through the addition of data at the end of the header by a new revision of the Pulse specification, the Header Size field will reflect this. 

Offset to Pulse Data:
  The actual number of bytes from the beginning of the file to the first pulse record data field. In the current version this is at least 352 bytes. This data offset must be updated if any software adds/removes Variable Length Records.

Number of Pulses:
  This field contains the total number of pulse records within the file.

Pulse Format:
  The format of the pulse records. In the current version this is always 0.

Pulse Attributes:
  A bit mask that allows specifying up to 32 additional attributes that can be defined in future versions of the specification. These attributes will directly follow the pulse record in the order defined by the appearance of 1 bits when reading the bitmask from lowest to highest bit. Currently two additional attributes are defined: a 16 bit pulse source ID (0x00000001) and a 32 bit pulse source ID (0x00000002).

Pulse Size:
  The size, in bytes, of the pulse record. All pulse records within a pulse file have the same format, the same attributes, the same extra bytes and the same size. If the specified size is larger than implied by the pulse format plus attributes (e.g. 50 bytes instead of 48 bytes for format 0 without attributes) the remaining bytes are user-specific “extra bytes”. The meaning of such “extra bytes” can be described with an Extra Bytes VLR (see Table 12 and Table 24) to make them useful to others as well.

Pulse Compression:
  The compression scheme used for the pulses. Currently there is no compression and this is always 0.

Reserved:
  Must be zero.

Number of Variable Length Records:
  This field contains the current number of VLRs that are stored in the file before the Pulse Records. This number must be updated if the number of VLRs changes.

Number of Appended Variable Length Records:
  This field contains the current number of AVLRs that are stored the file after the Pulse Records. This number should be updated if the number of AVLRs changes. This number may be set to \"-1\", which indicates that the number of AVLRs is not known and must be determined my parsing the AVLRs starting at the end of the file.

T Scale Factor:
  This field contains a double-precision floating point value that is used to scale the GPS time stamps T of the pulse records which are integer values. If these integers represent the GPS time in microseconds the scale factor should be set to 1e-6, if these integers represent the GPS time as nanoseconds the scale factor should be set to 1e-9.

T Offset:
  This field contains a double-precision floating point value that is used to offset the GPS time stamps T of the pulse records after they were scaled. If the timestamps are in GPS seconds of week and no GPS week information is available then a suitable offset is zero. If the timestamps are in standard GPS time then a suitable offset is 1 billion (or a similar high number). This is because standard GPS time is a uniformly counting time measure in seconds since midnight of January 5th to January 6th 1980 and that number has recently passed 1 billion seconds. The actual time stamps t could then be computed with

  t = (T_{record} \* T_{scale}) + T_{offset}

  But careful, we advise against doing a conversion to floating-point for standard GPS time. A standard 64 bit floating-point number is not able to store the resulting without precision loss when T_{offset} is aforementioned large number.

Min and Max T:
  The min and max of the integer timestamps stored in the T field of all pulses. To convert the min and max numbers to actual GPS times use the formula above.

X, Y, and Z Scale Factors:
  The scale factor fields contain double-precision floating point values used to scale the X, Y, and Z long values of the pulse records. If the actual x, y, z coordinates are in meter and have centimeter resolution (e.g. two decimal digits) then each scale factor will contain the number 0.01.   

X, Y, and Z Offset:
  The offset fields contain double-precision floating point values used to offset the X, Y, and Z long values of the pulse records. The formulas shown below convert from the X, Y, and Z long values of each pulse to the actual x, y, z coordinates.

  x_{coordinate} = (X_{record} \* x_{scale}) + x_{offset}

  y_{coordinate} = (Y_{record} \* y_{scale}) + y_{offset}

  z_{coordinate} = (Z_{record} \* z_{scale}) + z_{offset}

Min and Max X, Y, Z:
  The min and max fields describe the bounding box that includes the first and the last points of the sampled parts of the returning waveforms of all pulses.

Variable Length Records (VLRs):
------------------------------------------------------------------------------

The Pulse Header can be followed by any number of Variable Length Records (VLRs). The number of VLRs is specified in the “Number of Variable Length Records” field in the Pulse Header. The Variable Length Records must be accessed sequentially since the size of each Variable Length Record is contained in the Variable Length Record Header. Each Variable Length Record Header is 96 bytes in length. 

.. csv-table:: Variable Length Records (VLRs)
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "User ID", "char[16]", "16 bytes"
    "Record ID", "unsigned long", "4 bytes"
    "Reserved", "unsigned long", "4 bytes"
    "Record Length After Header", "long long", "8 bytes"
    "Description", "char[64]", "64 bytes"

User ID:
  The User ID field of ASCII characters identifies the user which created the Variable Length Record. If the character data is less than 16 characters, the remaining data must be null. The User ID "PulseWaves_Spec" is reserved.

Record ID:
  The Record ID allows to distinuish different VLRs with the same User ID. The Record IDs for the User ID "PulseWaves_Spec" are reserved. Publicizing the meaning of a Record ID is left to the owner of the given User ID. 

Reserved:
  Must be zero.

Record Length after Header:
  The record length is the number of bytes for the record after the end of the standard part of the header. The entire record length is 96 bytes (the header size of the VLR) plus the Record Length after Header.

Description:
  Null terminated text description (optional).  Any characters not used must be null.

Appended Variable Length Records (AVLRs):
------------------------------------------------------------------------------

The Pulse Records are followed by Appended Variable Length Records (AVLRs). The AVLRs are in spirit just like the VLRs but carry their payload "in front" of the footer that desribes them. They are accessed sequentially in reverse starting from the end of the file. There is at least one mandatory AVLR that indicates the end of the AVLR array. Because the AVLRs are accessed in reverse this mandatory AVLR is the first AVLR after the pulse records. The number of AVLRs is specified in the “Number of Appended Variable Length Records” field in the Pulse Header. Setting this number to a negative value (e.g. -1) means that their number is not known but must be discovered by parsing the AVLRs starting from the end of the file. Each Appended Variable Length Record Header is 96 bytes in length. 

.. csv-table:: Appended Variable Length Records (AVLRs)
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "User ID", "char[16]", "16 bytes"
    "Record ID", "unsigned long", "4 bytes"
    "Reserved", "unsigned long", "4 bytes"
    "Record Length Before Footer", "long long", "8 bytes"
    "Description", "char[64]", "64 bytes"

Pulse Records:
------------------------------------------------------------------------------

All records must be the same type. Unused attributes must be set to the equivalent of zero for the respective data type (e.g. 0.0 for floating-point numbers, NULL for ASCII, 0 for integers). The pulse record format 0 expresses the pulse as an anchor point plus target point that is 1000 sampling units away in the direction that the laser pulse was emitted. This makes it easy to represent the pulse in another coordinate system as it only requires to transform the two points, anchor and target.

.. csv-table:: Pulse Record Type 0
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10

    "GPS timestamp T", "long long", "8 bytes"
    "Offset to Waves", "long long", "8 bytes"
    "Anchor X", "long", "4 bytes"
    "Anchor Y", "long", "4 bytes"
    "Anchor Z", "long", "4 bytes"
    "Target X", "long", "4 bytes"
    "Target Y", "long", "4 bytes"
    "Target Z", "long", "4 bytes"
    "First Returning Sample [sampling units]", "short", "2 bytes"
    "Last Returning Sample [sampling units]", "short", "2 bytes"
    "Pulse Descriptor Index", "8 bits (bit 0-7)", "8 bits"
    "Reserved", "4 bits (bit 8-11)", "4 bits"
    "Edge of Scan Line", "1 bit (bit 12)", "1 bit"
    "Scan Direction", "1 bit (bit 13)", "1 bit"
    "Mirror Facet", "2 bits (bit 14-15)", "2 bits"
    "Intensity", "unsigned char", "1 byte"
    "Classification", "unsigned char", "1 byte"

GPS timestamp T:
  The GPS time in seconds at which the laser pulse was fired as a scaled and offset 64 bit integer. This field stores either the GPS week time or the Standard GPS time. The rational to use a scaled integer instead of a double-precision floating-point number is that the latter slowly looses precision as time progresses.

Offset to Waves:
  The offset in bytes from the start of the Waves file to the samples of the waveform. How the pulse is sampled (and more) is described in the Pulse Descriptor that is indexed by a later field.

Anchor X, Anchor Y, and Anchor Z:
  The anchor point of the pulse. Scaling and offseting the integers Anchor X, Anchor Y, and Anchor Z with the scale and offset from the header gives the actual coordinates of the anchor point. In case the Offset from Optical Center to Anchor Point field of the corresponding Pulse Descriptor is zero, the anchor point coincides with the location of the scanner's optical origin (or the pseuso origin) at the time the laser was fired.

  x_{anchor} = (X_{anchor} \* x_{scale}) + x_{offset}

  y_{anchor} = (Y_{anchor} \* y_{scale}) + y_{offset}
 
  z_{anchor} = (Z_{anchor} \* z_{scale}) + z_{offset}

Target X, Target Y, and Target Z:
  Specified the pulse by providing a target point theough which the pulse passes that is situated 1000 sampling units away from the anchor in the direction that the pulse was emitted (e.g. towards the ground in an airborne survey). Scaling and offseting the integers Target X, Target Y, and Target Z with the scale and offset from the header gives the actual coordinates of the target point:

  x_{target} = (X_{target} \* x_{scale}) + x_{offset}

  y_{target} = (Y_{target} \* y_{scale}) + y_{offset}
 
  z_{target} = (Z_{target} \* z_{scale}) + z_{offset}

Using the difference between anchor and target point, a pulse direction vector (dx,dy,dz) can be computed that expresses the distance that the laser pulse travels in one thousand sampling units. Dividing this vector by one thousand results results in a direction vector that is scaled in the length of units of the world coordinate system (e.g. meters for UTM, decimal degrees for long/lat, feet for US stateplane reference systems) chosen for anchor and target points and points away from the origin of the laser:

  dx = (x_{target} - x_{anchor}) / 1000 = (X_{anchor} - X_{target}) \* x_{scale} / 1000

  dy = (y_{target} - y_{anchor}) / 1000 = (Y_{anchor} - Y_{target}) \* y_{scale} / 1000

  dz = (z_{target} - z_{anchor}) / 1000 = (Z_{anchor} - Z_{target}) \* z_{scale} / 1000
 
First Returning Sample:
  The duration in sampling units from the anchor point to the first recorded waveform sample. Together with the anchor point and the pulse direction vector, this value allows computing the x/y/z world coordinates of the first sample that was recorded for the returning waveform of this pulse:

  x_{first} = x_{anchor} + first_returning_sample \* dx

  y_{first} = y_{anchor} + first_returning_sample \* dy

  z_{first} = z_{anchor} + first_returning_sample \* dz
  
  For pulses that do *not* have a returning waveform this value must be set to zero.

Last Returning Sample:
  Same concept as the First Returning Sample but for the last one:

  x_{last} = x_{anchor} + last_returning_sample \* dx

  y_{last} = y_{anchor} + last_returning_sample \* dy

  z_{last} = z_{anchor} + last_returning_sample \* dz
  
  For pulses that do *not* have a returning waveform this value must be set to zero.

Index of Pulse Descriptor:
  The record ID minus 200,000 of the "PulseWaves_Spec" VLR or AVLR that contains a description of this laser pulse and the exact details how its waveform is sampled in form of a "Pulse Descriptor". Up to 255 different descriptors can be specified. A pulse descriptor consist of a "Composition Record" followed by a variable number "Sampling Records".

Reserved:
  Must be zero.

Scan Direction Flag:
  This bit remains the same as long as pulses are output with the mirror of the scanner travelling in the same direction. It flips whenever the mirror direction changes. [We should define which bit means which direction from an airborne / mobile collection point of view].

Edge of Scan Line:
  This bit has a value of 1 when the output pulse is at the end of a scan line. It is the last pulse before the scanning hardware has a change in direction or mirror facet.

Mirror Facet:
  These two bits encode which mirror facet the pulse is reflected from. These two bits do not change as long as subsequent pulses are from the same mirror facet of the scanner.

Intensity:
  This value characterizes the returned intensity of the pulse for easy understanding and quick visualization purposes. It should be properly scaled so that it can be used to color the pulse for previewing purposes. It could, for example, be scaled according to the highest digitized value on the returning wave. The value may or may not have a physical meaning.

Classification:
  This value could be used to (pre-)classify entire pulses into a yet to be established metric. Possible are the number of waveform peaks or a simple roof, forest, grass, road, water flagging to provide some insight and understanding of the attached waveforms when previewing only the pulse data.

Defined Variable Length Records (VLRs or AVLRs):
------------------------------------------------------------------------------

The same mechanism described for the "LASF_Projection" VLR of the LAS 1.4 specification can be used to geo-reference the pulse file. The same mechanism described for the "LASF_Proj" VLR "Extra Bytes" of the LAS 1.4 specification can be used to specify extra attributes per pulse.

First Appended Variable Length Record:
------------------------------------------------------------------------------

User ID:                        PulseWaves_Spec

Record ID:                      4,294,967,295 (0xFFFFFFFF)

Record Length Before Footer:    0

This empty AVLR record *must* directly follow the pulse records and it must be the first AVLR in case there are multiple AVLRs. It does not carry a payload but is used to mark the end (as seen in reverse) of the appendable list of AVLRs. This is needed as the exact number of AVLRs may not be specified in the header and needs to be discovered by parsing all AVLRs starting *at the end* of the file until this one is reached. This Record ID makes no sense when used with an VLR. 

Scanner:
------------------------------------------------------------------------------

User ID:                            PulseWaves_Spec

Record ID: 	                    n (where 100,001 <= n < 100,255)

The Scanner VLR describes the scanner system that the pulse originated from.

.. csv-table:: Scanner VLR
    :header: "Item", "Unit", "Format", "Size"
    :widths: 70, 10, 10, 10

    "Size", "---", "unsigned long", "4 bytes"
    "Reserved", "---", "unsigned long", "4 bytes"
    "Instrument", "---", "char[64]", "64 bytes"
    "Serial", "---", "char[64]", "64 bytes"
    "Wave Length", "[nanometer]", "float", "4 bytes"
    "Outgoing Pulse Width", "[nanometer]", "float", "4 bytes"
    "Scan Pattern", "---", "unsigned long", "4 bytes"
    "Number of Mirror Facets", "---", "unsigned long", "4 bytes"
    "Scan Frequency", "[hertz], "float", "4 bytes"
    "Scan Angle Min", "[degree], "float", "4 bytes"
    "Scan Angle Max", "[degree], "float", "4 bytes"
    "Pulse Frequency", "[kilohertz], "float", "4 bytes"
    "Beam Diameter at Exit Aperture", "[millimeters]", "float", "4 bytes"
    "Beam Divergence", "[milliradians]", "float", "4 bytes"
    "Minimal Range", "[meter]", "float", "4 bytes"
    "Maximal Range", "[meter]", "float", "4 bytes"
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "Description", "---", "char[64]", "64 bytes"

Size:
  The byte-aligned size of attributes from and including "Size" to and including "Description".

Reserved:
  Must be zero.

Instrument:

Serial:

Wave Length:
  The physical wavelength of the laser in nanometers.

Outgoing Pulse Width:
  The width of the outgoing pulse as defined by the full width at half maximum (FWHM) in nanometer. The exact width and intensity tends to vary from pulse per pulse which is why the outgoing waveform is often sampled and stored per pulse as well.

Scan Pattern:
  Stores the type of scanning pattern used: 0 = undefined, 1 = oscillating, 2 = line, 3 = conic

Number of Mirror Facets:
  Stores the number of mirror facets for a line scanner.

Scan Frequency:
  Stores the scan frequency at which the scanner was operating in Hertz.

Scan Angle Min:
  Stores the minimal scanner angle at which the scanner was operating in angular degree.

Scan Angle Max:
  Stores the maximal scanner angle at which the scanner was operating in angular degree.

Pulse Frequency:
  Stores the pulse frequency at which the scanner was operating in Kilohertz.

Beam Diameter at Exit Aperture:
  The diameter of the laser beam in the moment it leaves the scanner hardware in millimeter.

Beam Divergence:
  The divergence of the laser beam in milliradians @ 1/e2. [or should we use @ 1/e]?

Minimal Range:
  Stores the minimal range at which the scanner is able to operate in meters.

Maximal Range:
  Stores the maximal range at which the scanner is able to operate in meters.

Description:
  Null terminated text description (optional).  Any characters not used must be null.

Pulse Descriptor:
------------------------------------------------------------------------------

User ID: 	                    PulseWaves_Spec

Record ID: 	                    n (where 200,001 <= n < 200,255)

The Pulse Descriptor describes the (optionally segmented) sampling(s) of the pulse's outgoing and/or returning waveform(s). For example, the outgoing waveform with 32 samples and the returning waveform with 256 samples. Waveforms can also be sampled with multiple sensors. For example, the outgoing waveform with 40 samples and the returning waveform with two sensors of different sensitivity both at 480 samples. Waveforms can also be sampled with multiple discontinuous segments. For example, three successive segments for the returning waveforms, the first with 80, the second with 160, and the last with 80 samples, ... etc. A Pulse Descriptor consists of a "Composition Record" that is immediately followed by a variable number of "Sampling Records" that allow a very flexible description of segmentings and samplings of the waveforms with one or multiple sensors.

.. csv-table:: Composition Record 
    :header: "Item", "Unit", "Format", "Size"
    :widths: 70, 10, 10, 10

    "Size", "---", "unsigned long", "4 bytes"
    "Reserved", "---", "unsigned long", "4 bytes"
    "Optical Center to Anchor Point", "[sampling units]", "long", "4 bytes"
    "Number of Extra Wave Bytes", "---", "unsigned short", "2 bytes"
    "Number of Samplings", "---", "unsigned short", "2 bytes"
    "Sample Units", "[nanoseconds]", "float", "4 bytes"
    "Scanner Index", "---", "unsigned long", "4 bytes"
    "Compression", "---", "unsigned long", "4 bytes"
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "Description", "---", "char[64]", "64 bytes"

Size:
  The byte-aligned size of attributes from and including "Size" to and including "Description".

Reserved:
  Must be zero.

Optical Center to Anchor Point:
  This value specifies the constant temporal offset in sampling units from the optical center to the anchor point - given such a constant exists. If the value is 0, anchor point and optical center coincide. Otherwise the optical center of a pulse can be found by "walking" backwards from its anchor point as many units of its direction vector as specified here (a conversion step may be necessary in case that anchor point and direction vector are not in a Euclidean coordinate system). If the value is 0x8FFFFFFF there is no constant temporal offset between the optical center and the anchor point. In this case the optical center cannot be "reached" from the anchor point by "walking" a constant multiple of the direction vector but the duration may be specified for each anchor point individually.

Number of Extra Waves Bytes:
  Specified the number of extra bytes that the waves are storing before the actual data describing the waves begins. These extra bytes may or may not be meaningful to the current version of the PulseWaves reader, but knowing their number assures forward-compatibility in case later versions add new attribute information to all waves.

Number of Samplings:
  A value larger than 0 specifying the number of "Sampling Records" that immediately follow this "Composition Record".

Sample Units:
  Specifies the temporal unit of sampling in nanoseconds that sample the waveform. One nanosecond (1e-9 seconds) is 1,000 picoseconds (1e-12 seconds). If multiple sample resolutions are used by the following "Sampling Records" then the shortest common multiple is specified here.

Scanner Index:
  There may be several laser scanning units that are simultaneously storing their output to the same PulseWaves file. They can be then be distinguished by letting their pulse descriptors index a different scanner. The default is 0 which indicates that no particular scanner is specified. Up to 255 different scanners can be specified.

Compression:
  In the current version this is always 0.

Description:
  Null terminated text description (optional).  Any characters not used must be null.

Sampling Records:
------------------------------------------------------------------------------

.. csv-table:: Sampling Record 
    :header: "Item", "Unit", "Format", "Size"
    :widths: 70, 10, 10, 10

    "Size", "---", "unsigned long", "4 bytes" 
    "Reserved", "---", "unsigned long", "4 bytes" 
    "Type", "---", "unsigned char", "1 byte" 
    "Channel", "---", "unsigned char", "1 byte" 
    "Unused", "---", "unsigned char", "1 byte"
    "Bits for Duration from Anchor", "---", "unsigned char", "1 byte" 
    "Scale for Duration from Anchor", "---", "float", "4 bytes"
    "Offset for Duration from Anchor", "---", "float", "4 bytes"
    "Bits for Number of Segments", "---", "unsigned char", "1 byte" 
    "Bits for Number of Samples", "---", "unsigned char", "1 byte" 
    "Number of Segments", "---", "unsigned short", "2 bytes"
    "Number of Samples", "---", "unsigned long", "4 bytes"
    "Bits per Sample", "---", "unsigned short", "2 byte" 
    "Lookup Table Index", "---", "unsigned short", "2 bytes" 
    "Sample Units", "[nanosecond]", "float", "4 bytes"
    "Compression", "---", "unsigned long", "4 bytes" 
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "...", "...", "...", "..."
    "Description", "---", "char[64]", "64 bytes"

Size:
  The byte-aligned size of attributes from and including "Size" to and including "Description".

Reserved:
  Must be zero.

Type:
  This number is 1 when the sampling describes the outgoing waveform. This number is 2 when the sampling describes a returning waveform.

Channel:
  This number is 0 when sampling with a single sensor. If the waveform is sampled with h channels the number is between 0 and h-1.

Unused:
  Must be zero.

Bits for Duration from Anchor:
  Specifies how many bits are used in the Waves file to store the integers that express the duration from the anchor point to the start of a segment (i.e. to the first sample of the segment) in sample units. In case the number of bits is zero the duration between anchor point to the first sample must be zero and there should only be one segment. The only non-zero values supported in the current version are 8, 16, or 32 bits.

Scale for Duration from Anchor:
  A scaling value that adjusts the resolution with which the duration from the anchor point to the start of a segment (i.e. to the first sample of the segment) is stored. A scaling value of 1.0 implies that all durations are integer multiples of the sampling unit. A scaling factor of, for example, 0.1 implies that the resolution is one tenth of the sampling unit.

Offset for Duration from Anchor:
  An offset value that adds a constant to every duration that can be used to avoid storing the same large offset with every duration. An offset value of 3000.0, for example, implies that all durations are implicitely 3000 sampling units longer than specified in the Waves file.

Hencem, the durations from anchor values that may be specified in the Waves file are scaled and offset integers and need to be multiplied with the scale and have the offset added to get the actual duration according to this formula:

  d = scale_for_duration_from_anchor \* D + offset_for_duration_from_anchor
  
Bits for number of segments:
  Specifies the number of bits used to store the number of segments in the sampling in case segmenting is variable. If this number is zero the segmenting is fixed and specified by the "Number of Segments" field below. The only non-zero values supported in the current version are 8 or 16 bits.

Bits for number of samples:
  Specifies the number of bits used to store the number of samples in the sampling in case the sampling is variable. If this number is zero the sampling is fixed and specified by the "Number of Samples" below.  The only non-zero values supported in the current version are 8 or 16 bits.

Number of Segments:
  If a fixed segmenting is used because the "Bits for Number of Segments" above is zero, this field specifies the number of segments in the segmenting. If a variable segmenting is used because the "Bits for Number of Segments" above is non-zero, this field is meaningless and should be zero.

Number of Samples:
  If a fixed sampling is used because the "Bits for Number of Samples" above is zero, this field specifies the number of samples in the sampling. If a variable sampling is used because the "Bits for Number of Samples" above is non-zero, this field is meaningless and should be zero.

Bits per sample:
  Specifies the number of bits used to store each sample.

Lookup Table Index:
  Specifies the index to an (optional) table that maps the the sample values to actually measured physical values. In the current version this is not supported and this value should always be 0.

Sample Units:
  Specifies the temporal unit of spacing between subsequent samples in nanoseconds (1e-9 seconds). Example values might be 0.5, 1.0, 2.0 and so on, representing digitizer frequencies of 2 GHz, 1 GHz and 500 MHz respectively.

Compression:
  The compression scheme used for the samples. In the current version there is no compression and this is always 0.

Description:
  Null terminated text description (optional). Any characters not used must be null.


GeoKeyDirectory (optional):
------------------------------------------------------------------------------

User ID:                         PulseWaves_Proj

Record ID: 	                     34735

This record contains the key values that define the coordinate system. A complete description can be found in the GeoTIFF format specification. Here is a summary from a programmatic point of view for someone interested in implementation. The GeoKeyDirectory is defined as an array of unsigned short values where the first 4 shorts describe the number of keys and the remaining groups of four shorts are the actual keys. See the LAS 1.4 specification (rev 11) for details.


GeoDoubleParams (optional):
------------------------------------------------------------------------------

User ID:                         PulseWaves_Proj

Record ID:                       34736

This record is simply an array of doubles that contain values that may be referenced by tags in the GeoKeyDirectory record. See the LAS 1.4 specification (rev 11) for additional details.


GeoAsciiParams (optional):
------------------------------------------------------------------------------

User ID:                         PulseWaves_Proj

Record ID:                       34737

This record is simply an array of ASCII data. It contains one or many strings separated by null or space characters which are referenced by position from tags in the GeoKeyDirectory. See the LAS 1.4 specification (rev 11) for additional details.


OGC Math WKT (optional):
------------------------------------------------------------------------------

User ID:                         PulseWaves_Proj

Record ID:                       2111

This record contains the textual data representing a Math Transform WKT as defined in section 7 of the Coordinate Transformation Services Spec, with the following notes:

    1) The OGC Math Transform WKT VLR data shall be a null-terminated string.
    2) The OGC Math Transform WKT VLR data shall be considered UTF-8.
    3) The OGC Math Transform WKT VLR data shall be considered C locale-based, and no localization of the numeric strings within the WKT should be performed.

See the LAS 1.4 specification (rev 11) for additional details.

OGC Coordinate System WKT (optional):
------------------------------------------------------------------------------

User ID:                         PulseWaves_Proj

Record ID:                       2112

This record contains the textual data representing a Coordinate System WKT as defined in section 7 of the Coordinate Transformation Services Spec, with the following notes:

    1) The OGC Math Transform WKT VLR data shall be a null-terminated string.
    2) The OGC Math Transform WKT VLR data shall be considered UTF-8.
    3) The OGC Math Transform WKT VLR data shall be considered C locale-based, and no localization of the numeric strings within the WKT should be performed.

See the LAS 1.4 specification (rev 11) for additional details.

==============================================================================
The Waves file (\*.wvs)
==============================================================================

The Waves file (\*.wvs) is not a stand-alone file but needs a corresponding Pulse file (\*.pls) to be meaningful. It contains the actual samples of the waveforms. Each pulse of the Pulse file contains a reference into the Waves file. All data are in little-endian format.

.. csv-table:: The Waves File Structure 
    :widths: 100

    "Header"
    "Waves of Pulse 0"
    "Waves of Pulse 1"
    "Waves of Pulse 2"
    "Waves of Pulse 3"
    "..."
    "Waves of Pulse k"

.. csv-table:: The Waves Header
    :header: "Item", "Format", "Size"
    :widths: 70, 10, 10
    
    "File Signature (“PulseWavesWaves”)", "char[16]", "16 bytes"
    "Compression", "unsigned long", "4 bytes"
    "Reserved", "unsigned char[44]", "40 bytes"

File Signature:
  The file signature must contain the zero-terminated string of 16 characters “PulseWavesWaves" that can be checked by user software as a quick look validate the file type.

Compression:
  Specifies whether the waves are uncompressed (0) or compressed. Currently only one experiemental compression scheme (1) is supported.

Reserved:
  Must be zero.

The header is a mostly place holder of 60 bytes to make it possible that a Waves file can easily be converted into a valid WDP file to accompany a LAS 1.3 or LAS 1.4 file that contains point types 4, 5, 9, or 10 without a full re-write of the Waves file. 

.. csv-table:: Waves of Pulse
    :header: "Item", "Units", "Format", "Size"
    :widths: 70, 10, 10, 10

    "Extra Waves Bytes", "---", "unsigned char[e]", "e bytes"
    "Number of Segments in Sampling 0", "---", "bits", "0, 8, or 16 bits"
    "Duration from Anchor for Segment 0 of Sampling 0", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 0 from Sampling 0", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 0 from Sampling 0", "---", "unsigned char[s0]", "s0 bytes"
    "...", "...", "...", "..."		
    "...", "...", "...", "..."
    "Number of Segments in Sampling 1", "---", "bits", "0, 8, or 16 bits"
    "Duration from Anchor for Segment 0 of Sampling 1", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 0 from Sampling 1", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 0 from Sampling 1", "---", "unsigned char[s1]", "s1 bytes"
    "Duration from Anchor for Segment 1 of Sampling 1", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 1 from Sampling 1", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 1 from Sampling 1", "---", "unsigned char[s0]", "s0 bytes"
    "Duration from Anchor for Segment 2 of Sampling 1", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 2 from Sampling 1", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 2 from Sampling 1", "---", "unsigned char[s0]", "s0 bytes"
    "...", "...", "...", "..."		
    "...", "...", "...", "..."
    "Number of Segments in Sampling 2", "---", "bits", "0, 8, or 16 bits"
    "Duration from Anchor for Segment 0 of Sampling 2", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 0 from Sampling 2", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 0 from Sampling 2", "---", "unsigned char[s2]", "s2 bytes"
    "Duration from Anchor for Segment 1 of Sampling 2", "sample units", "bits", "0, 8, 16, or 32 bits"
    "Number of Samples in Segment 1 from Sampling 2", "---", "bits", "0, 8, or 16 bits"
    "Samples of Segment 1 from Sampling 2", "---", "unsigned char[s0]", "s0 bytes"
    "...", "...", "...", "..."		
    "...", "...", "...", "..."		

Extra Waves Bytes:
  This field only exists if the "Number of Extra Waves Bytes" in the corresponding sampling record is non-zero. This field is currently not used but will allow forward compatibility in case that later versions of the PulseWaves format add additional attributes to the waves. The corresponding number of e extra bytes need then to be read or be skipped before attempting to read the next field of the waves of a pulse.

Number of Segments in Sampling m:
  This field only exists if the number of "Bits for Number of Segments" in the corresponding sampling record is non-zero. It then specifies the number of segments in this sampling that can vary from one pulse to the next (i.e. "variable segmentation"). If the number of "Bits for Number of Segments" in the corresponding sampling record is zero, the number of segments is fixed and is specified in the "Number of Sements" field of the corresponding sampling record (i.e. "fixed segmentation").

Duration from Anchor for Segment k of Sampling m:
  This field only exists if the number of "Bits for Duration from Anchor" in the corresponding sampling record is non-zero. It then specifies the duration from the anchor point to the first sample in sample units. Depending on the value of the corresponding "Scale for Duration from Anchor" and "Offset for Duration from Anchor" fields, this number may need to be scaled and offset by the respective amounts. If the "Scale for Duration from Anchor" field is 1.0 then the durations between the anchor point and the first sample can only be integer multiples of the sample unit. If the number of "Bits for Duration from Anchor" in the corresponding sampling record is zero, then this duration is zero. This means that the anchor point coincides with the first sample of the sampling. This can only be the case if the sampling consists of a single segment (or else all segments would start at the anchor). The duration determines the x/y/z coordinate of the 3D location of the first sample via the following calculation:

  x_{first_sample} = x_{anchor} + duration_from_anchor \* dx 

  y_{first_sample} = y_{anchor} + duration_from_anchor \* dy 

  z_{first_sample} = z_{anchor} + duration_from_anchor \* dz

  while the x/y/z coordinates of all following samples can be reached one by one by adding the dx/dy/dz vector again and again.

  One exception is the start of the sampling for the outgoing waveform. Here the duration in sampling units is expressed in relation to the origin of the pulse. Nothing changes if anchor point and origin are identical (i.e. if the "Optical Center to Anchor Points" field is zero).

Number of Samples in Segment k from Sampling m:
  This field only exists if the number of "Bits for Number of Samples" in the corresponding sampling record is non-zero. It then specifies the number of samples in the next segment that can vary from one pulse to the next (i.e. "variable sampling"). If the number of "Bits for Number of Samples" in the corresponding sampling record is zero, the number of samples is fixed and is specified in the the "Number of Samples" field of the corresponding sampling description (i.e. "fixed sampling").

Samples of Segment k from Sampling m:
  The actual waveform samples of sampling m either raw or compressed.

.. figure:: pulsewaves.jpg
   :scale: 100 %
   :alt: illustration of a Pulse Descriptor

   An illustration of a typical Pulse Descriptor

The rest of the document is gibberish ...
------------------------------------------------------------------------------

`PulseWaves`_ is a 

Example
..............................................................................


Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* The `PulseWaves` format is composed of a `Pulse` and a `Waves` file.


Future Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``Pulse`` requires ...

* Knowledge of how to make ...

Example Formatting
------------------------------------------------------------------------------

PulseWaves currently defines 

1) Pulse ...

2) Waves ...
  
   ::

    class Pulse
    {
    public:
        Pulse();
    private:
        // Magic
    };
    
    More.example();
    Code;
    Is.here();

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
