# ftpb
Introduction and Doc.

           FTPB - The File Transfer Protocol Batch ISPF Dialog
                        by Lionel B. Dyck


This ISPF dialog provides a simple to use ISPF interface for using the
TCP/IP FTP function to transfer data sets from the current MVS host
system to other host systems that support a TCP/IP FTP Server (e.g. MVS,
VM, OS/2, most Unix, ...).

The dialog present the user with a simple ISPF panel from which to
specify the source (original) data set, the target host, optionally the
name of the target data set if it is different from the source, and
signon information (userid and password).  Once this information is
complete the dialog generates the necessary JCL and FTP statements to
allow the user to execute the FTP in the foreground (execpt for load
library transfers) or to submit the JCL for a batch execution of FTP.
Prior to submission the user is allowed to review and change (edit) the
generated JCL and FTP control statements if they desire.

The JCL will be submitted using an FTP service that will wait for 10
minutes (as defined by the FTP.DATA used by the installation) and if the
JOB ends within 10 minutes the sysout will be retrieved and printed in
the FTPPRINT generated step.  The PRINT job step will print the report
to the //REPORT DD and will process the data to generate a return code
equal to the max return code for the remote job.

The specification of the z/OS Target may be Yes or No.  If yes, then the
data is transferred using z/OS native tranfer methodology. If no, then
the data is transfered as text (or binary if binary is selected).

The SYNC option for the z/OS Target enabled z/OS to z/OS processing and
allows for the comparison of the ISPF member statistics of both the
Source (local) and Target (remote) versions of the PDS. This allows
the option to Get (copy the target member to the local PDS), Put (copy
the local PDS member to the target PDS), or the ability to Compare the
target member to the local member. During this function the recommended
actions (GET/PUT) are presented, with the ability to change or ignore the
recommended action.


For partitioned data sets the dialog provides member selection if a
member is not specified as part of the source data set name.  Two
options are provided for the transfer of a PDS.  One option is to unload
the data set using IEBCOPY and to transfer that sequential data set,
while generating the JCL to reload the data set at the target host.  The
other option is to not unload in which case the dialog processed a member
at a time.

For VSAM and DA data sets the dialog will prompt to ask if the target
data set should be Replaced or not on the restore.  The transfer will
occur in batch using generated JCL. To transfer VSAM/DA the dialog uses
IBM's DFSMSdss (PGM=ADRDSSU) to DUMP the data set (using the SPHERE
option) and then the restore JCL and the dump'd data set are FTP'd to
the target host.  Once these FTP's are complete the JCL is submitted to
the target system for processing where ADRDSSU will be used to RESTORE
the data set.  The batch process is the same as described above for load
library transfers except that ADRDSSU is used instead of IEBCOPY.

Also supported are pattern dsnames which will then consist of a set
of data sets dump'd and restore'd using ADRDSSU.

Known Problems/Challenges:

1.  There exists a problem when using FTP to transfer load modules
    caused by the inability of FTP to correctly transfer the directory
    information for the load module.  The 'solution' for this is to
    use IEBCOPY to unload the load library into a sequential data set
    and then transfer that with a reload using IEBCOPY at the target
    site.

2.  The submitted JOB's SYSOUT will remain at the remote executiion
    site in the SPOOL until the SPOOL cleanup removes it as it is
    submitted with an OUTPUT JCL statement to be held so that it can
    be retrieved by the FTP GET (when filetype=jes is used) and thus
    included in the originating JOBs SYSOUT.


4.  When using DFSMSdss to dump/ftp/restore a PDSE the target may not
    restore if PDSE is not enabled and you will need to use the CONVERT
    statement (check the manual for syntax) in the restore.

Note that an ISPF Table is built and maintained in Library(ISPPROF)
that contains the jobcard information for each target node.

