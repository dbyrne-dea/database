# database

A test suite for creating an accession. Includes rudimentary tools to assign UUIDs, calculate SHA checksums and add files to a database (from a set of web-accessible folders (WAFs)). This initial pass at the problem is intended to create an emergency database that can easily be incorporated into a more mature product as on becomes available.

## Workflow 

Ingest flow is anticipated to be as follows:

1. A suite of data, or dataset, is acquired consisting of files arranged in some provider defined directory structure. Files may be of any type. Metadata in XML or JSON files may or may not be present.  `(Remember: collections do not have their own files!)`
1. In ISO-14721 terminology, this suite is termed a Submission Information Package (SIP) and is assigned an accession number.    
    1. The accession number should be of the form YYYYMMDDAAAAAA, with the first six digits indicating the
date the dataset was ingested into the system, and the second six, a unique number. `(Using solely date based information can cause collisions.)` Example: 20150821000007. `(you might think of dividing the latter 6 digits into a couple of fields that provide further information, for example, operator number, and an ordinal that the operator assigns)`
1. The accession is assigned a UUID. 
1. The UUID and accession-level metadata are stored in an accession information table  (are you really going to use an ASCII TSV for metadata?)   
    1. initially in a TSV ASCII table    
    2. later converted to SQL table database. 
1. The directory structure is read. 
1. For each file in the collection,  the `SHASUM -a384`  is calculated and 
the file copied to a second database structure, `${DATABASE}/` which is organized according to the SHA digest. E.g., a file with shasum of 
`38b060a751ac96384cd9327eb1b1e36a21fdb71114be07434c0cc7bf63f6e1da274edebfe76f65fbd51ad2f14898b95b`
might be stored as   
`${DATABASE}/38/b0/60/a751ac96384cd9327eb1b1e36a21fdb71114be07434c0cc7bf63f6e1da274edebfe76f65fbd51ad2f14898b95b`. 
1. A UUID is assigned to each file in the second database and added
to another files table in the MySQL database. 
    1. The files table has the collection ID, the SHAsum, the file UUID, the original filename and path, and the name of an XML file that holds the metadata for that file. 

```
DATA/
|- RAWDATA/
|  |- YYYYMMDDHHMM/
|  |  |- My special data set/
|  |     |- Bunch of directories with stupid names/
|  |     |- More stupidly named directories/
|  |        |- Worse - named-file'_s with bad! punctuation & spelling
|  |- yyyymmddhhmm/
|     |- Cruise 321/
|        |- CTD/
|        |- XBT/
|        |- Chlorophyll/
|           |- chladata.xls
| 
|- DATABASE/
|  |- 00/
|  |  |- 00/
|  |  |  |- 00/
|  |  |  |- 01/ 
|  |  |     |- f0128adfea073e ... 12ff # shasum named file -
|  |  |                                # this file has shasum-384 of
|  |  |                                # 000001f0128adfea073e...12ff
|  |  |- 01/ 
.  .  .      
.  .  .      
.  .  .      
|  |- 38/
|  |  |- b0/
|  |     |- 60/
|  |        |- a751ac96384cd9327eb1b1e36a21fdb71114be07434c0cc7bf63f6e1da274edebfe76f65fbd51ad2f14898b95b
|  |- 39/
|  |- 3a/
|  .
|  .
|  .`
|  |- fe/
|  |- ff/ 
|
|- LINKFILES
|  |- YYYYMMDDHHMM/
|  |  |- My special data set/
|  |     |- Bunch of directories with stupid names/
|  |     |- More stupidly named directories/
|  |        |- Worse - named-file'_s with bad! punctuation & spelling --> DATA/DATABASE/00/00/01/f01...12ff
|  |- yyyymmddhhmm/
|     |- Cruise 321/
|        |- CTD/
|        |- XBT/
|        |- Chlorophyll/
|           |- chladata.xls -> DATA/DATABASE
|
|- UUIDLINKS/
   |- 8584e980-889c-486f-88f8-ee94100c7608 -> DATA/LINKFILES/YYYYMMDDHHMM/
   |- 3cd9cb29-fd07-4c89-b9aa-15b5cd56d850 -> DATA/LINKFILES/YYYYMMDDHHMM/My special data set/
   |- 8a99c65a-3104-4bd1-9ca3-0218cbdc993c -> DATA/LINKFILES/YYYYMMDDHHMM/My special data set/Bunch of directories with stupid names/
   |- 7abae45b-dcff-4ea2-bdc3-cc2553faff0f -> DATA/LINKFILES/YYYYMMDDHHMM/My special data set/More stupidly named directories/
   |- c0873272-3e3e-41fb-a2a1-240e19cf74b4 -> DATA/LINKFILES/yyyymmddhhmm/
   |- 0adec7af-36ce-45ea-93a0-38327c9111e0 -> DATA/LINKFILES/yyyymmddhhmm/Cruise 321/
   |- 6753993d-be5d-412d-8ba0-28252b0cfa08 -> DATA/LINKFILES/yyyymmddhhmm/Cruise 321/CTD/
   |- 8281b40d-da48-4b2b-acba-0b9c79d563ee -> DATA/LINKFILES/yyyymmddhhmm/Cruise 321/XBT/
   |- e24d23f9-1ba3-4e56-afcb-a5c7152ee437 -> DATA/LINKFILES/yyyymmddhhmm/Cruise 321/Chlorophyll/



```


## Glossary

*accession* is a bunch of related files.   
*collection* has no data: a superset dsescribing a bunch of related accessions   
*dataset* informal term . often maps to accession, not always.   

