
# global-irt

Global IRT (Incident Response Team) is a project to describe common IRT and abuse contact information

The global-irt project aims at finding a common format/naming scheme between different CERT/CSIRT directories.
At the time of this writing, there are a couple of CERT/CSIRT directories:

  * https://api.first.org  at FIRST.org
  * the [national CSIRT database] at cert.org

Each one of these directories has a different format, different field names and different (but sometimes overlapping) data.

See also [RIPE-658](https://www.ripe.net/publications/docs/ripe-658) for a list of known abuse contact (and CSIRT/CERT) lookup mechanisms.


The Global IRT project aims at defining a format which encompasses the existing formats. Also it shall provide a client 
library which can query (if permissions are given) the CERT/CSIRT directories and provide a common answer from all of these databases.
Global IRT tries to provide a query and common output format for the existing CSIRT/CERT directories.

In order to achieve this, a [common taxonomy of CSIRTs](https://github.com/FIRSTdotorg/global-irt/wiki/CSIRT-CERT-types) is also very helpful:
with a list of known CSIRT/CERT types (national, PSIRT, etc.) a sender can choose which CSIRT/CERT to contact.




# How to contribute?

Please read [CONTRIBUTING.md](https://github.com/FIRSTdotorg/global-irt/blob/master/CONTRIBUTING.md)
