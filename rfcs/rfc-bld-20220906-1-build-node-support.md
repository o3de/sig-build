Intro
-----

This policy proposal documents O3DF's support levels for major dependencies that are maintained in the Automated Review (AR) system.

Policy
------

### Ownership

SIG-Build will maintain dependencies for the AR build nodes. Feature requests and bug fixes can be sent via Github and assigned to the SIG through labels or directly in the SIG-Build repo.

### Scope

These are currently AWS EC2 AMI (Amazon Machine Image) that contain dependencies which include, but not limited to the following:

*   OS and its build level - Windows, Ubuntu, MacOS
*   OS level packages - libxcb\*, mesa, Java, libssl, zlib
*   Compilers/Backends - MSVC, LLVM, GCC
*   Build systems/Frontends - Visual Studio (MSBuild), Ninja, Clang, XCode
*   Pre-build tools - System Python (including PyPi packages), CMake
*   Platform specific SDK - Android SDK
*   Build/Test specific tools - Git/Git LFS

### Support priority

In conjunction with SIG-Platform, the general policy will be:

#### OS

*   One OS will be supported in AR at any given time. This will generally be one version behind the latest version (ex. Windows 2019 while latest is Windows 2022, or Ubuntu 20.04 LTS while latest is Ubuntu 22.04 LTS). The timing of the version supported will be agreed upon between SIG-Build and SIG-Platform
*   For nightly builds, latest OSes will be supported. Generally, only LTS versions are deployed. The version supported will be set by SIG-Platform and implemented by SIG-Build
*   For simplicity, the same build dependencies will be installed on all OS versions where possible. Multiple permutations will require agreement with TSC, SIG-Build, and SIG-Platforms

#### Build System

*   One Build system/Frontend will be supported in AR at any given time. The timing of the version supported will be agreed upon between SIG-Build and SIG-Platform
*   For nightly builds, the latest or preview version of the build system will be supported. The version supported will be set by SIG-Platform and implemented by SIG-Build

#### Pre-Build Tools and SDK

*   One set version will be supported in AR at any given time. The timing of the version supported will be agreed upon between SIG-Build and SIG-Platform
*   For nightly builds, the latest or preview version of the build system will be supported. The version supported will be set by SIG-Platform and implemented by SIG-Build

### Current support levels

#### OS

| OS  | Version | Environment | Notes |
| --- | --- | --- | --- |
| Windows | 2019 1809 (eqv Windows 10 20H2) | AR  |
| Windows | 2022 21H2 (eqv Windows 11 21H2) | periodic-daily | Planned |
| Ubuntu | 20.04 LTS | AR  |
| Ubuntu | 22.04 LTS | periodic-daily | Planned |
| MacOS | Big Sur | AR  |
| MacOS | Monterey | periodic-daily | Planned |

#### Build Systems/Compiler

| Build System | Version | Compiler | Version | OS  | Environment |
| --- | --- | --- | --- | --- | --- |
| Visual Studio | 2022 17.3.0 | MSVC | v143 | Windows 2019 | AR  |
| Visual Studio | 2019 16.9.2 | MSVC | v142 | Windows 2019 | periodic-daily |
| Ninja | 1.10.0 | Clang/LLVM | 12  | Ubuntu 20.04 | AR  |
| Ninja | 1.10.1 | Clang/LLVM | 14  | Ubuntu 22.04 | periodic-daily |
| Ninja | 1.10.0 | GCC | 9.3.0 | Ubuntu 20.04 | periodic-daily |
| Ninja | 1.10.1 | GCC | 11.2.0 | Ubuntu 22.04 | periodic-daily |
| XCode | 13.2 | Clang/LLVM | 12  | MacOS Big Sur | AR  |
| XCode | 13.4 | Clang/LLVM | 13  | MacOS Monterey | periodic-daily |

#### SDK

| SDK Type | Package Versions | OS  | Environment |
| --- | --- | --- | --- |
| Android SDK | "platforms;android-28" "platforms;android-29" "platforms;android-30" | Windows 2019 | AR and periodic-daily |
| Android NDK | '"ndk;21.4.7075529"' | Windows 2019 | AR and periodic-daily |
| Android Google Play Packages | '"extras;google;market\_apk\_expansion" "extras;google;market\_licensing"' | Windows 2019 | AR and periodic-daily |
| Android Build Tools | '"build-tools;30.0.2" | Windows 2019 | AR and periodic-daily |

#### Tools

| Tool Type | Package Versions | OS  | Environment |
| --- | --- | --- | --- |
| CMake | 3.24.0 | Windows 2019 | AR and periodic-daily |
| CMake | 3.24.0 | Ubuntu 20.04 | AR and periodic-daily |
| CMake | 3.24.0 | Ubuntu 22.04 | AR and periodic-daily |
| CMake | 3.24.0 | MacOS Big Sur | AR and periodic-daily |
