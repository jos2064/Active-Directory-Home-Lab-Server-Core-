# Active Directory Home Lab

A hands-on Active Directory home lab demonstrating enterprise-level domain configuration, user management, and Group Policy implementation using Windows Server Core and VirtualBox/VMware Fusion.

## Project Overview

This project demonstrates the setup and configuration of a functional Active Directory environment using Windows Server Core as a domain controller. The lab showcases core AD administration skills including domain services installation, organizational unit structure, user management, and group policy configuration.

## Architecture

### Hardware Setup
Due to hardware limitations, this lab utilizes a multi-device setup:

- **Windows Laptop**: Hosts Windows 10 VM (VirtualBox) - Domain member machine
- **MacBook**: Hosts Windows Server Core (VMware Fusion Pro) - Domain Controller

### Network Configuration

**Domain Controller (Windows Server Core)**:
- Platform: VMware Fusion Pro
- IP Configuration: Static IP address
- DNS Server: 8.8.8.8 (Google Public DNS)
- Role: Primary DNS server for the domain

**Client Machine (Windows 10)**:
- Platform: VirtualBox
- VM Name: `vboxuser`
- DNS Configuration: Pointed to Domain Controller's IP address
- Purpose: Enables domain name resolution and AD authentication

This DNS configuration ensures:
- DC can resolve external domains via Google DNS
- Client machines resolve internal domain resources through the DC
- Proper domain join and authentication functionality

## Technologies Used

- **Virtualization**: 
  - VMware Fusion Pro (macOS)
  - Oracle VirtualBox (Windows)
- **Operating Systems**:
  - Windows Server Core (Domain Controller)
  - Windows 10 (Client Machine)
- **Active Directory Components**:
  - Active Directory Domain Services (AD DS)
  - Remote Server Administration Tools (RSAT)
  - Active Directory Users and Computers (ADUC)
  - Group Policy Management

## Implementation Steps

### 1. Domain Controller Setup (Server Core)

**Installed Windows Server Core** on VMware Fusion Pro and configured it as the domain controller using PowerShell/Command line:

```powershell
# Installed Active Directory Domain Services
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promoted server to Domain Controller
Install-ADDSForest -DomainName "yourdomain.local"
```

**Network Configuration**:
```powershell
# Configured static IP address
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Set DNS to Google Public DNS for external resolution
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "8.8.8.8"
```

**Domain-Wide Security Policies**:
Configured password and account lockout policies via PowerShell on the Domain Controller:
```powershell
#Set minimum password length to 10 characters
net accounts /minpwlen:10

# Configure account lockout after 5 failed attempts
net accounts /lockoutthreshold:5

# Set account lockout duration to 15 minutes
net accounts /lockoutduration:15

# Set lockout observation window to 15 minutes
net accounts /lockoutwindow:15
```
These policies enforce:

- Password Complexity: Minimum 10-character passwords for all domain accounts
- Brute Force Protection: Account locks after 5 failed login attempts
- Security Balance: 15-minute lockout prevents attacks while minimizing user impact
### 2. Domain Member Configuration

**Windows 10 VM Setup** (VirtualBox):
- Created Windows 10 VM named `vboxuser`
- **Configured DNS**: Set DNS server to Domain Controller's IP address (critical for domain operations)
- Joined the VM to the newly created domain
- **Installed RSAT** (Remote Server Administration Tools) using PowerShell:

```powershell
# Install all RSAT tools on Windows 10
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online
```

- Logged in with domain Administrator credentials
- Verified RSAT installation by accessing Active Directory Users and Computers (ADUC)

**Why DNS Configuration Matters**:
- Client machines must use the DC as their DNS server
- This enables domain name resolution and location of domain controllers
- Without proper DNS configuration, domain join and authentication will fail

**RSAT Tools Enabled**:
- Active Directory Users and Computers (dsa.msc)
- Active Directory Administrative Center
- DNS Management tools
- Other AD administration utilities

### 3. Organizational Unit (OU) Structure

Created a logical OU hierarchy for organizational management:

```
Domain Root
├── Bangalore (Location-based OU)
├── Pune (Location-based OU)
└── Sales (Department-based OU)
```

**Purpose**: Segregate users and computers by location and department for granular policy application and administrative delegation.

### 4. User Management

**Created Test Users**:
- Username: `Jos Root`
- Created within appropriate OU structure
- Validated authentication by logging in with new credentials

**Verification Process**:
- Successfully logged into domain-joined machine using `Jos Root` credentials
- Confirmed user profile creation and domain authentication

### 5. Group Policy Configuration

**Challenge**: Group Policy Management Console (GPMC) not available 

**Solution**: Demonstrated policy concepts using Local Group Policy Editor on client machine

```cmd
# Opened Local Group Policy Editor
gpedit.msc

# Applied system-level policies
# Forced policy refresh
gpupdate /force
```

**Policies Implemented**:

#### Interactive Login Message
- **Policy Path**: `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options`
- **Configured**: Interactive logon message text and title
- **Purpose**: Display custom message and title to users attempting to login
- **Use Case**: Security warnings, acceptable use policies, or welcome messages

#### Removable Storage Restrictions
- **Policy Path**: `Computer Configuration > Administrative Templates > System > Removable Storage Access`
- **Configured**: Denied access to removable drives
- **Purpose**: Prevent unauthorized data transfer via USB drives
- **Security Benefit**: Data loss prevention and endpoint security

#### Machine Inactivity Limit

- **Policy Path**: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options
- **Setting**: Interactive logon: Machine inactivity limit
- **Value**: 300 seconds (5 minutes)
- **Purpose**: Automatically lock workstation after 5 minutes of inactivity
- **Security Benefit**: Prevents unauthorized access to unattended machines

#### Policy Application
- Successfully applied policies using `gpupdate /force`
- Verified policy enforcement through user login testing
- Demonstrated understanding of policy precedence and scope

## Key Learnings

### Technical Skills Developed
- Windows Server Core command-line administration
- Active Directory Domain Services installation and configuration
- DNS configuration and troubleshooting in AD environments
- Static IP configuration via PowerShell
- Cross-platform virtualization (VMware Fusion + VirtualBox)
- Domain joining and authentication processes
- OU design and implementation
- User account creation and management via ADUC
- Group Policy fundamentals and application
- Security policy implementation (login messages, removable storage restrictions)
- Network configuration for domain environments

### Challenges Overcome
- **Hardware Constraints**: Successfully implemented AD environment across two separate physical machines
- **Server Core Limitations**: Worked around GUI limitations using PowerShell and remote management tools
- **GPMC Unavailability**: Demonstrated understanding of Group Policy using alternative methods and Local Group Policy Editor

## Project Screenshots

Visual documentation of the implemented Active Directory environment:

### Organizational Units Structure
- Screenshot showing OU hierarchy (Bangalore, Pune, Sales)
- Demonstrates logical organizational design

### User Account Management
- New user account creation (Jos Root)
- Active Directory Users and Computers interface
- User properties and configuration

### User Login & Authentication
- Domain user login page
- Successful authentication with newly created credentials
- User profile creation and desktop access

### Group Policy Implementation
- Interactive login message configuration
  - Custom message text displayed at login
  - Custom message title for security notifications
- Removable storage access denial
  - Policy configuration in Local Group Policy Editor
  - Enforcement verification

**Screenshot Directory Structure**:
```
screenshots/
├── OU_structure.png
├── user_creation.png
├── user_login_page.png
├── successful_login.png
├── login_message_config.png
├── login_message_display.png
├── removable_storage_policy.png
└── policy_enforcement.png
```

*All screenshots demonstrate hands-on implementation and successful configuration of Active Directory components.*

## Prerequisites

To replicate this lab, you'll need:

- **Hardware**: 
  - 2 machines (or 1 machine with sufficient resources for multiple VMs)
  - Minimum 8GB RAM recommended
- **Software**:
  - VMware Fusion Pro or VirtualBox
  - Windows Server (Evaluation available from Microsoft)
  - Windows 10 (Evaluation available from Microsoft)
- **Knowledge**:
  - Basic understanding of Windows administration
  - Familiarity with virtualization concepts
  - Command-line interface comfort

## Future Enhancements

Potential additions to expand this lab:

- [ ] Implement full Group Policy Management using RSAT on dedicated management station
- [ ] Configure DNS and DHCP services
- [ ] Add file server role and shared folders with NTFS permissions
- [ ] Implement security groups and advanced permissions delegation
- [ ] Set up certificate services (AD CS)
- [ ] Configure Active Directory replication with second DC for fault tolerance
- [ ] Create user home directories and folder redirection policies
- [ ] Deploy software via Group Policy (MSI packages)
- [ ] Implement password policies and account lockout policies
- [ ] Set up Active Directory backup and recovery procedures

## Contributing

This is a personal learning project, but suggestions and feedback are welcome! Feel free to open an issue or submit a pull request.

## Author

Built as a hands-on learning project to develop practical Active Directory administration skills.

---

