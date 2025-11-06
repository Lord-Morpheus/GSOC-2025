# Google Summer of Code 2025 - Project Summary
![GSOC 2025](./media/gsoc.png)

## Project Title
**KDE Print Manager: CUPS 3.x Compatibility and Testing Infrastructure**
![Linux Foundation](./media/linux.png)

## Author
**Tarun Srivastava**  
GSoC 2025 Contributor - Open Printing @ Linux Foundation

## Mentors
[Till Kamppeter](https://github.com/tillkamppeter) - Open Printing Lead (Printing Guru)
[Mike Noee](https://invent.kde.org/noee) - KDE developer (Print Manager specialist)

## Overview
This project focused on modernizing the KDE Print Manager to support the newer CUPS 3.x while maintaining backward compatibility with CUPS 2.x. The work encompassed two major objectives:

1. **CUPS 3.x Migration**: Enabling KDE Print Manager to work seamlessly with the latest version of CUPS
2. **Automated Testing Infrastructure**: Transitioning from manual testing to automated test suites with CI/CD pipeline integration

## Project Links

### Repository
 - [KDE Print Manager (Development Fork)](https://invent.kde.org/tarunsri/print-manager) — branches of interest: `test_branch` (autotests), `for-V3` (refactoring), `kcupsrequest` (API changes)

### CUPS 3.x Migration Merge Requests
- [MR #227 - Initial CUPS 3.x Support](https://invent.kde.org/plasma/print-manager/-/merge_requests/227)
- [MR #235 - API Updates](https://invent.kde.org/plasma/print-manager/-/merge_requests/235)
- [MR #263 - Modern API Implementation](https://invent.kde.org/plasma/print-manager/-/merge_requests/263)
- [MR #264 - Additional Compatibility Changes](https://invent.kde.org/plasma/print-manager/-/merge_requests/264)

### Testing Infrastructure
- [MR #271 - Autotest Implementation](https://invent.kde.org/plasma/print-manager/-/merge_requests/271)

---

## Project Description

### Background
Modern printers have evolved to support IPP (Internet Printing Protocol) driverless printing, which enables automatic discovery by CUPS without requiring traditional CUPS queues or PPD (PostScript Printer Description) files. The transition to CUPS 3.x represents a paradigm shift away from these legacy components.

### Project Goals
This project aims to modernize the KDE Print Manager by:

- **Enabling CUPS 3.x Compatibility**: Implementing support for the latest CUPS version while maintaining backward compatibility with CUPS 2.x
- **IPP Print Destinations Support**: Incorporating support for various IPP-based printing scenarios:
  - Driverless network printers
  - IPP-over-USB printers
  - Printer Applications
- **Eliminating Legacy Dependencies**: Moving away from traditional CUPS queues and PPD files where appropriate
- **Backend Modernization**: Updating the CUPS backend integration within KDE Print Manager

---

## Technical Background

### What is CUPS 3.x?
CUPS 3.x is the new version of CUPS which uses libcups3 library. libcups3 is the latest major version of the CUPS (Common UNIX Printing System) library, developed as part of the OpenPrinting project’s effort to modernize and enhance the printing infrastructure. It represents a significant evolution from its predecessor, libcups2, introducing a redesigned and streamlined API that aligns with modern software development practices. The primary purpose of libcups3 is to provide a consistent and robust interface for applications and system components to interact with CUPS servers, enabling functionalities such as printer discovery, job management, and configuration using the Internet Printing Protocol (IPP).

**Major Changes:**
- **Removed Legacy APIs**: Deprecated and outdated interfaces eliminated
- **IPP & HTTP/HTTPS Focus**: Streamlined communications using modern protocols
- **Binary Compatibility Break**: Applications built against libcups2 require adjustments or recompilation
- **DNS-SD Support**: Enhanced DNS-based Service Discovery for automatic printer detection
- **Modern Network Printing**: Simplified setup and management, particularly beneficial for driverless printing workflows

---

### KDE Print Manager: Current Architecture with CUPS

The current KDE Print Manager is a collection of components integrated into the Plasma System Settings that provides comprehensive management of CUPS printer configurations.

#### Core Functionality
At its core, KDE Print Manager acts as a user-friendly interface wrapping the capabilities of the CUPS printing system, allowing users to:
- Configure printers easily
- Manage printer queues
- Monitor print jobs
- Avoid direct interaction with low-level CUPS commands

#### The libkcups Library

KDE Print Manager leverages a dedicated KDE library called **libkcups**, which provides:
- **Qt-Friendly Interface**: Modern C++ API wrapping native CUPS APIs
- **Printer Configuration**: Access to printer setup and management
- **Job Management**: Print job control and monitoring
- **Data Models**: Structures used by KDE's print-related components:
  - System settings module (KCM)
  - System tray plasmoid
  - Legacy utilities (`configure-printer`, `kde-print-queue`)

This modular approach allows KDE applications to uniformly interact with CUPS, supporting both legacy and modern device discovery and management features.

Notably, KDE Print Manager integrates with the OpenPrinting project’s system-config-printer interfaces to provide additional capabilities such as new device notifications, device discovery, grouping, and recommended driver suggestions. This integration enhances the management experience by enabling automatic detection of printers, both network-connected and USB, particularly for driverless and IPP-based printers. For USB printers supporting IPP but lacking network interfaces, features like IPP-USB support further ease configuration, fitting well with KDE’s goal of providing zero-configuration printing solutions.

The Print Manager also provides advanced CUPS daemon management options, including:
- Starting/stopping CUPS services
- Adding/deleting printer queues
- Managing printer instances (multiple queues for the same physical device with different default settings)
- Print job administration (cancellation, pausing, restarting, status monitoring)
- Direct integration within the KDE environment

Behind the scenes, KDE Print Manager's reliance on libcups2 enables it to communicate effectively with the CUPS server using IPP.

---

## Implementation Approach

### Phase 1: Implementing libcups3 Support

The migration to libcups3 involved extensive refactoring due to API changes, deprecated functions, and architectural modifications in the CUPS library.

#### API Migration Challenges

**Deprecated Functions and Macros:**

The following components required updates due to libcups3 changes:
- **`KCupsRequest`**: Uses legacy functions/macros that no longer exist in libcups3
- **`KCupsJob`**: Required updates for modern job state management
- **`KCupsConnection`**: Needed refactoring for current connection handling

**Specific Changes:**
- IPP job state macros renamed (deprecated since CUPS 2.5.x):
  - Old: `IPP_JOB_*`
  - New: `IPP_JSTATE_*`
- Legacy function replacements implemented throughout codebase
- Macro definitions updated for compatibility

**Reference:** [MR #227 - Initial CUPS 3.x Support](https://invent.kde.org/plasma/print-manager/-/merge_requests/227)

#### Modern API Adoption

**New Functions Implemented:**

1. **`cupsEnumDests()`**
   - Modern approach for enumerating available printers (destinations)
   - Replaces older, deprecated enumeration methods
   - Improved efficiency and compatibility

2. **`cupsCopyDestInfo()`**
   - Used to fetch PPD files
   - Modern substitute for the older `getPPD2` API
   - Better handling of printer-specific configurations and capabilities

#### Architectural Pattern Changes

**Asynchronous Printer Attribute Retrieval via Notified Signals:**

The new implementation introduces an event-driven approach for retrieving printer attributes:

**Old Pattern:**
- Request builds and updates a complete printer list internally
- Client notified only after full list compilation
- Synchronous, blocking approach
- Higher memory overhead

**New Pattern:**
- Client receives callbacks/signals for each printer individually
- Incremental processing of printers
- Event-driven architecture
- Client code can process and update UI incrementally
- Request object can be deleted after finishing with notifications

**Benefits:**
- Improved responsiveness
- Dynamic updates
- Better memory usage
- More reactive user experience
- No overhead of creating and copying full lists

**Reference:** [MR #263 - Modern API Implementation](https://invent.kde.org/plasma/print-manager/-/merge_requests/263)

---

### Phase 2: Automated Testing Infrastructure

#### Testing Challenges

After completing the CUPS 3.x migration, comprehensive testing became critical due to:
- Extensive backend API changes
- Risk of regressions in core functionality
- No existing automated test suite
- Manual testing requiring significant developer/tester resources
- CI/CD pipeline not configured for automated testing

#### Solution: Comprehensive Autotest Suite

The implementation of automated testing aimed to:
- Eliminate manual testing overhead
- Integrate with CI pipeline for continuous validation
- Establish foundation for ongoing code coverage improvements
- Verify API functionality automatically

**Reference:** [MR #271 - Autotest Implementation](https://invent.kde.org/plasma/print-manager/-/merge_requests/271)

#### Implemented Test Classes

##### 1. ModelAutoTest

**Purpose:**  
The ModelAutoTest class is designed to verify the correctness and reliability of the data models that serve as the foundation for KDE Print Manager's printer representation and management. These models are critical as they bridge the gap between the low-level CUPS API and the high-level Qt-based user interface components.

**Coverage:**
- Printer model functionality
- PPD model operations
- Device model management

**Validates:**
- Data structures for printer enumeration
- Driver information handling
- Device management in UI layer
- Model signals and data change notifications
- Proper filtering and sorting of printer lists

##### 2. CupsServerAutoTest

**Purpose:**  
The CupsServerAutoTest class focuses on validating the interaction between KDE Print Manager and the CUPS server daemon. This test suite is crucial for ensuring that the application can reliably communicate with CUPS to retrieve server settings, apply configuration changes, and maintain stable connections.

**Coverage:**
- Server settings retrieval
- Configuration updates
- Server communication protocols
- Authentication and permission handling
- Error handling and recovery mechanisms

**Components Tested:**
- `KCupsRequest` interface
- `KCupsServer` interface
- Server connection stability
- IPP request/response handling
- Timeout and retry logic

**Critical for:** Validating correct CUPS server communication essential for stable printing operations

##### 3. CallKcmTests

**Purpose:**  
The CallKcmTests class validates the KDE Control Module (KCM) component, which provides the primary user interface for printer configuration within the KDE System Settings. This test suite ensures that the KCM loads correctly, displays printer information accurately, and responds appropriately to user interactions.

**Type:** User-interaction test  
**Status:** Complements automated tests (not fully automated yet)

**Coverage:**
- KCM module loading and initialization
- Printer configuration UI display
- Settings persistence and retrieval
- Integration with Plasma System Settings
- UI state management and updates

##### 4. PrinterManagerTest

**Purpose:**  
The PrinterManagerTest class is designed to provide comprehensive end-to-end testing of the entire KDE Print Manager system, covering the complete printer lifecycle from discovery and addition through configuration, usage, and removal. This test suite aims to validate the integration of all components working together as a cohesive system.

**Status:** Currently disabled 
**Requirement:** Active CUPS server connection

**Coverage:**
- Complete printer lifecycle operations
- Printer discovery and enumeration
- Queue creation and configuration
- Print job submission and management
- Printer status monitoring
- Error handling and edge cases

## Remaining Work
It will take a long time before we get entire code coverage for Print manager. Getting the autotest working on CI will take continuous input and frequent input from KDE sysadmin developers because they will have to approve each and every request for any kind of change in CI images. Backend for CUPS 3.x is established for the print manager so there will be almost zero effort in moving to CUPS 3.x for KDE. There is another task of UI related changes and testing to be done to handle the changes which libcups3 will bring. This part will be explicitely handles by us later as per Mike suggests.

---