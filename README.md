# Secure SMA Software Package Management Framework
## 1. Overview

This project designs and implements a secure software-package management
framework for communication between a Service Management Agent (SMA) and
Service Management and Orchestration (SMO).

The framework is responsible for securely handling software packages
distributed from the SMO side to the SMA.

The software package is represented as a `tar` or `tar.gz` archive.

The primary security requirement is to protect the software package.

# 2. Problem Statement

The SMO maintains a JSON configuration file containing information about
available software packages.

The JSON contains information such as:

- Software/package name
- Package version
- Source path of the software

The actual software is stored in the filesystem at the specified
`source_path`.

The SMA requests revision information from the SMO using a
`getRevision` operation.

The SMO returns the available revisions and corresponding paths.

The SMA compares the received revision information with its local
software requirements, represented by `required-pods.json`.

If the required software revision differs from the available/current
software, the corresponding software package is obtained from the
specified path.

The software directory is packaged as a `tar`/`tar.gz` archive and the
package must be protected before being used by the SMA.

The SMA then:

1. Obtains the protected package.
2. Validates/accesses the package.
3. Extracts the package.
4. Loads/updates the application.
5. Retains the package as required by the system.

The framework must also handle version differences without blindly
replacing a newer software version with an older one.

# 3. Scope
The framework covers:

- Software package identification
- Software version handling
- Package protection, validation, access, and extraction
- Application update
- Failure handling

# 4. High-Level Architecture

The system consists of the following major logical components:

                         SMO
                          |
                          |
                +---------v----------+
                |   SMO JSON Config  |
                |                    |
                | package            |
                | version/revision   |
                | source_path        |
                +---------+----------+
                          |
                          |
                +---------v----------+
                | Software Directory |
                +---------+----------+
                          |
                          |
                    Package Creation
                          |
                    +-----v------+
                    |    TAR     |
                    +-----+------+
                          |
                    +-----v------+
                    |   GZIP     |
                    +-----+------+
                          |
                    +-----v------+
                    |   Package  |
                    | Protection |
                    +-----+------+
                          |
                          |
                    Protected Package
                          |
                          |
                          v
                         SMA
                          |
                +---------v----------+
                | Package Manager    |
                +---------+----------+
                          |
                +---------v----------+
                | Package Validation |
                +-------5--+----------+
                          |
                +---------v----------+
                | Package Access      |
                | / Unlocking         |
                +---------+----------+
                          |
                +---------v----------+
                | Extraction         |
                +---------+----------+
                          |
                +---------v----------+
                | Version Comparison |
                +---------+----------+
                          |
                +---------v----------+
                | Application Update |
                +--------------------+

**#5. Selective Application Update & Package Retention**

When the SMA has an older software version, the application should be
updated without unnecessarily deleting unrelated package content.

The archive should not automatically be deleted after extraction if
package retention is required.
