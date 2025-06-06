# Automated Host Collection Tool

[[toc]]

## Introduction

Prophet is an open-source automated collection and analysis tool developed by OneProCloud. It supports efficient data collection and detailed analysis for various environments, including physical servers, cloud hosts, and VMware. The tool is designed for cloud migration and disaster recovery planning. By automatically gathering key information from source hosts and comparing technical metrics, it ensures smooth migration or disaster recovery. Prophet has been successfully used in multiple real-world projects, proving its stable performance and reliable features.

More Details: [View in Github](https://github.com/Cloud-Discovery/prophet/blob/main/docs/README_EN.md)

Submit New Issue: [New Issue](https://github.com/Cloud-Discovery/prophet/issues)

## Functions

Currently, Prophet consists of the following main features:

1. **Network Host Scanning**
    Scans the network for live hosts using the `nmap` command, and attempts to analyze the basic information of the hosts through packet data.

2. **VMware Environment Information Collection**
    Collects detailed host information through VMware API, including compute resources, storage, and network-related data, supporting host migration-related analysis.

3. **Linux Host Information Collection**
    Uses Ansible to gather detailed information from Linux hosts, including compute resources, storage, and network-related data.

4. **Windows Host Information Collection**
    Collects detailed information from Windows hosts using the Windows WMI interface, including compute resources, storage, and network-related data.

5. **Data Packaging and Anonymization**
    Packages and compresses the collected data in YAML format, and performs data anonymization by removing sensitive user information.

6. **Data Analysis and Technical Conclusions**
    Analyzes the collected data to generate a comprehensive technical research report, providing a reliable foundation for migration and disaster recovery planning.

## Installation Guide

### **Recommended Environment:**

To ensure the best performance of the Prophet tool, use the following environment:

* Operating System: Ubuntu 20.04 and above
* Container Support: Docker (Ubuntu's built-in version)

#### **Installation Process:**

1. **Install Docker:** On Ubuntu, directly install Docker using the system's package manager:

```bash
sudo apt update
sudo apt install -y docker.io
```

2. **Start and Verify Docker Service:** After installation, start the Docker service, enable it to start on boot, and check its status:

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

Verify Docker installation:

```bash
sudo docker --version
```
After installation, you can run the Prophet tool in a containerized environment.

### **Get Prophet Container**

1. **Download Prophet Container Image**

    Use the following command to download the latest version of Prophet from the container image repository:

    ```bash
    docker pull registry.cn-beijing.aliyuncs.com/oneprocloud-opensource/cloud-discovery-prophet:latest
    ```

2. **Run Prophet Container Service**

    Use the following command to start the Prophet container service:

    ```bash
    docker run -d -ti --name prophet registry.cn-beijing.aliyuncs.com/oneprocloud-opensource/cloud-discovery-prophet:latest
    ```

3. **Access Prophet Container**

    After the container starts, you can enter the container using the following command for further operations:

    ```bash
    docker exec -ti prophet bash
    ```

4. **Use Prophet CLI**

    After entering the container, you can use the Prophet CLI tool with commands supporting `scan`, `collect`, and `report` operations:

    ```bash
    prophet-cli {scan,collect,report}
    ```
Follow these steps to complete the deployment and use of the Prophet container service.

## User Guide

### Basic Usage Process

1. Scan the specified IP address range.
2. In the CSV file of the scan results, provide authentication information for the hosts you want to retrieve details for.
3. Perform batch collection.
4. Analyze and obtain results.

### Function 1: Scan All Running Instances on the Network

#### Function Description

Discover live hosts within a specific network segment through network scanning and record the findings. This information can be used as input for more detailed information collection in subsequent steps. After the scan is complete, a `scan_results.csv` file will be automatically generated at the specified path to store the information.

**Note: To prevent significant pressure on the production environment, the scan is performed in a single-process mode, resulting in a slower scanning progress. It has been estimated that scanning a subnet with a mask of 24 requires approximately 30 minutes.**


```
usage: prophet-cli scan [-h] --host HOST [--arg ARG] --output-path OUTPUT_PATH
                        [--report-name REPORT_NAME]

optional arguments:
  -h, --help            show this help message and exit
  --host HOST           Input host, example: 192.168.10.0/24, 192.168.10.1-2
  --arg ARG             Arguments for nmap, for more detailed, please check
                        nmap document
  --output-path OUTPUT_PATH
                        Generate initial host report path
  --report-name REPORT_NAME
                        Scan report csv name
```

#### Example 1: Obtain Subnet Hosts

Scan all live hosts in the 192.168.10.0/24 subnet and generate a CSV file in the /tmp directory.


```
prophet-cli scan --host 192.168.10.0/24 --output-path /tmp/
```

#### Example 2: Obtain Hosts in a Specified IP Range

Scan all live hosts in the range 192.168.10.2 to 192.168.10.50 and generate a CSV file in the /tmp directory.

```
prophet-cli scan --host 192.168.10.2-50 --output-path /tmp/
```

#### CSV Structure Explanation

| Field Name   | Field Description                                               |
|--------------|-------------------------------------------------------------|
| hostname     | Hostname, can be empty                                          |
| ip           | User IP address, mandatory                                      |
| username     | Username; for VMware, it is the username for ESXi or vCenter     |
| password     | Password; for VMware, it is the password for ESXi or vCenter     |
| ssh_port     | For Linux, this field represents the SSH port; for VMware ESXi or vCenter, it is the connection port, defaulting to 443; for Windows, it is empty by default |
| key_path     | If using key-based login, specify the absolute path to the key; otherwise, it is empty |
| mac          | Host MAC address, can be empty                                   |
| vendor       | Vendor, can be empty; if it is a virtual machine running on VMware, it will be VMware |
| check_status | Whether detailed information collection is needed; set to "check" if required, otherwise, the tool will automatically skip |
| os           | Operating system type; currently supported types are: Linux/Windows/VMware; case-sensitive |
| version      | Operating system version, can be empty                           |
| tcp_ports    | Open external ports, can be empty                                 |
| do_status    | Detailed information collection status, indicating whether collection is complete or failed; default is empty                         |

### Function 2: Detailed Information Collection

#### Function Description

After users input authentication information into the template, further detailed scanning is performed.

Note:

* For hosts to be scanned, set the `check_status` to "check"; otherwise, no check will be performed.
* For VMware virtual machines, scanning will only be done through the associated ESXi host.
* For Windows hosts, scanning requires the Administrator user.
* Successfully collected hosts will not undergo collection again upon subsequent script runs unless the user specifies the `force-check` parameter.
* Hosts that fail in collection will be reattempted in the next collection.
* In the final generated compressed package, all sensitive information related to user authentication has been removed.
* (Stable) The VMware collection part is currently stable.
* (Testing) The Linux and Windows collection parts are still in testing.

```
usage: prophet-cli collect [-h] --host-file HOST_FILE --output-path
                           OUTPUT_PATH [-f] [--package-name PACKAGE_NAME]

optional arguments:
  -h, --help            show this help message and exit
  --host-file HOST_FILE
                        Host file which generated by network scan
  --output-path OUTPUT_PATH
                        Output path for batch collection
  -f, --force-check     Force check all hosts
  --package-name PACKAGE_NAME
                        Prefix name for host collection package
```

#### Example: Execute Collection

First, update the authentication information for the hosts to be collected in the generated `scan_results.csv`. The project's examples directory includes a [sample scan_results.csv](https://github.com/Cloud-Discovery/prophet/blob/master/examples/scan_results.csv).


|hostname|ip            |username                   |password             |ssh_port|key_path|mac              |vendor|check_status|os     |version                                   |tcp_ports                                                                            |do_status|
|--------|--------------|---------------------------|---------------------|--------|--------|-----------------|------|------------|-------|------------------------------------------|-------------------------------------------------------------------------------------|---------|
|        |192.168.10.2  |administrator@vsphere.local|your_vcenter_password|        |        |00:50:56:9f:03:f0|      |check       |Windows|Microsoft Windows 7 or Windows Server 2012|80,88,135,139,389,443,445,514,636,2020,3389,8088,9009,9090                           |         |
|        |192.168.10.6  |root                       |your_esxi_password   |443     |        |0c:c4:7a:e5:5d:20|      |check       |VMware |VMware ESXi Server 4.1                    |22,80,427,443,902,5988,5989,8000,8080,8100,8300                                      |         |
|        |192.168.10.185|Administrator              |your_windows_password|        |        |00:50:56:9a:6d:2e|      |check       |Windows|Microsoft Windows Server 2003 SP1 or SP2  |135,139,445,1028,3389                                                                |         |
|        |192.168.10.201|root                       |your_linux_password  |22      |        |ac:1f:6b:27:7f:e4|      |check       |Linux  |Linux 2.6.32 - 3.9                        |22,80,3306,4567,5000,5900,5901,5902,5903,5904,5906,5907,5910,5911,5915,8022,8080,9100|         |


```
prophet-cli collect --host-file /tmp/scan_hosts.csv --output-path /tmp
```

#### Collection Result Explanation

Collection Directory Structure

```
hosts_collection
|-- LINUX -> Linux Host Collection Information
|-- VMWARE -> VMWare Host Collection Information
|-- WINDOWS -> Windows Host Collection Information
|-- prophet.log -> Log during the collection process, useful for analyzing unknown scenarios
|-- scan_hosts.csv -> File containing information on collected hosts, including open port details
```

Additionally, a `hosts_collection_xxxxxxx.zip` file will be generated in the output directory. This file is the final compressed file used for analysis.

### Function 3: Analyze and Generate Reports

#### Function Description

Analyze the collected results and generate the final migration report. This section can be expanded based on specific requirements.

```
usage: prophet-cli report [-h] --package-file PACKAGE_FILE --output-path
                          OUTPUT_PATH [--report-name REPORT_NAME] [--clean]

optional arguments:
  -h, --help            show this help message and exit
  --package-file PACKAGE_FILE
                        Investigate package file which is genreated by
                        prophet-collect
  --output-path OUTPUT_PATH
                        Generate report path
  --report-name REPORT_NAME
                        Generate report name
  --clean               Clean temp work dir or not, by default is not
```

#### Example: Analyze and Generate Report

```
prophet-cli -d -v report --package-file /tmp/hosts_collection_20211215202459.zip --output-path /tmp
```

#### Example: Analyze Report

| Platform Type | Hostname | VMware Hostname | IP | Mac | Operating System Type | Operating System Version | Operating System Bit | Operating System Kernel Version | Boot Method | CPU | CPU Cores | Memory | Total Memory (GB) | Free Memory (GB) | Disk Count | Total Disk Capacity (GB) | Disk Information | Partition Information | Network Card Count | Network Card Information | Virtualization Type | Virtualization Version | ESXi Server |
|--------|---------------------|---------|------------------------|-----------------|----------------------------------|----------------------------------|------|---------------------|----|-----------------------------------------|-----|---------------|--------|--------|----|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|----|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|-------------------------------|---------------------|
|Physical|compute201           |         |192.168.10.201          |ac:1f:6b:27:7f:e4|CentOS                            |7.5.1804                          |x86_64|3.10.0-862.el7.x86_64|bios|Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz|40   |               |62.65   |5.35    |7   |5467004.78 |sda&#124;56266.78&#124;ATA&#124;SuperMicro SSD sdb&#124;213212.97&#124;ATA&#124;INTEL SSDSC2BB24 sdc&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sdd&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sde&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sdf&#124;1039505.00&#124;TOSHIBA&#124;AL15SEB120NY sdg&#124;1039505.00&#124;TOSHIBA&#124;AL15SEB120NY                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |/dev/sda2&#124;62245572608&#124;58963365888&#124;0.95&#124;xfs /dev/sda1&#124;1063256064&#124;922746880&#124;0.87&#124;xfs /dev/sdc1&#124;1199655976960&#124;1141407465472&#124;0.95&#124;xfs|4   |eno1&#124;0c:c4:7a:a5:29:3a&#124;True&#124;1500&#124;10000&#124;10.0.100.201&#124;255.255.255.0&#124;10.0.100.0&#124;10.0.100.255&#124;fe80::ec4:7aff:fea5:293a ens2f3&#124;ac:1f:6b:27:7f:e7&#124;True&#124;1500&#124;1000&#124;None&#124;None&#124;None&#124;None&#124;fe80::ae1f:6bff:fe27:7fe7 ens2f0&#124;ac:1f:6b:27:7f:e4&#124;True&#124;1500&#124;1000&#124;192.168.10.201&#124;255.255.255.0&#124;192.168.10.0&#124;192.168.10.255&#124;192.168.10.1&#124;fe80::ae1f:6bff:fe27:7fe4 ens2f1&#124;ac:1f:6b:27:7f:e5&#124;True&#124;1500&#124;1000&#124;172.16.100.201&#124;255.255.255.0&#124;172.16.100.0&#124;172.16.100.255&#124;fe80::ae1f:6bff:fe27:7fe5|                 |                               |                     |
|Physical|compute202           |         |192.168.10.202          |0c:c4:7a:e5:5c:f8|CentOS                            |7.5.1804                          |x86_64|3.10.0-862.el7.x86_64|bios|Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz|40   |               |62.65   |13.27   |7   |5467004.78 |sda&#124;56266.78&#124;ATA&#124;SuperMicro SSD sdb&#124;213212.97&#124;ATA&#124;INTEL SSDSC2BB24 sdc&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sdd&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sde&#124;1039505.00&#124;SEAGATE&#124;ST1200MM0007 sdf&#124;1039505.00&#124;TOSHIBA&#124;AL15SEB120NY sdg&#124;1039505.00&#124;TOSHIBA&#124;AL15SEB120NY                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |/dev/sda2&#124;62245572608&#124;52926242816&#124;0.85&#124;xfs /dev/sda1&#124;1063256064&#124;922746880&#124;0.87&#124;xfs                                               |4   |eno1&#124;0c:c4:7a:a5:2e:d0&#124;True&#124;1500&#124;10000&#124;10.0.100.202&#124;255.255.255.0&#124;10.0.100.0&#124;10.0.100.255&#124;fe80::ec4:7aff:fea5:2ed0 ens2f3&#124;0c:c4:7a:e5:5c:fb&#124;True&#124;1500&#124;1000&#124;None&#124;None&#124;None&#124;None&#124;fe80::ec4:7aff:fee5:5cfb ens2f0&#124;0c:c4:7a:e5:5c:f8&#124;True&#124;1500&#124;1000&#124;192.168.10.202&#124;255.255.255.0&#124;192.168.10.0&#124;192.168.10.255&#124;192.168.10.1&#124;fe80::ec4:7aff:fee5:5cf8 ens2f1&#124;0c:c4:7a:e5:5c:f9&#124;True&#124;1500&#124;1000&#124;172.16.100.202&#124;255.255.255.0&#124;172.16.100.0&#124;172.16.100.255&#124;fe80::ec4:7aff:fee5:5cf9   |                 |                               |                     |
|VMware  |node01               |         |                        |00:50:56:9a:49:b7|CentOS 4/5/6/7                    |CentOS 4/5/6/7                    |64-bit|                     |efi |Intel(R) Xeon(R) CPU E5-2680 0 @ 2.70GHz |4    |               |8192.00 |        |1   |51200.00   |[10.3-4T-5] centos7.4_node_139/centos7.4_node_139.vmdk&#124;51200.00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |                                                                                                                                 |1   |VM Network&#124;00:50:56:9a:49:b7                                                                                                                                                                                                                                                                                                                                                                                                                                               |VMware ESX Server|VMware ESXi 6.0.0 build-2715440|192.168.10.3         |
|VMware  |master03             |         |                        |00:50:56:9a:63:a0|CentOS 4/5/6/7                    |CentOS 4/5/6/7                    |64-bit|                     |efi |Intel(R) Xeon(R) CPU E5-2680 0 @ 2.70GHz |4    |               |4096.00 |        |1   |51200.00   |[10.3-4T-5] centos7.2_132/centos7.2_132.vmdk&#124;51200.00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |                                                                                                                                 |1   |VM Network&#124;00:50:56:9a:63:a0                                                                                                                                                                                                                                                                                                                                                                                                                                               |VMware ESX Server|VMware ESXi 6.0.0 build-2715440|192.168.10.3         |
|VMware  |master02             |         |                        |00:50:56:9a:06:ad|CentOS 4/5/6/7                    |CentOS 4/5/6/7                    |64-bit|                     |efi |Intel(R) Xeon(R) CPU E5-2680 0 @ 2.70GHz |4    |               |4096.00 |        |1   |51200.00   |[10.3-4T-5] centos7.3_131/centos7.3_131.vmdk&#124;51200.00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |                                                                                                                                 |1   |VM Network&#124;00:50:56:9a:06:ad                                                                                                                                                                                                                                                                                                                                                                                                                                               |VMware ESX Server|VMware ESXi 6.0.0 build-2715440|192.168.10.3         |
|Physical|COMPUTER-PC          |         |192.168.10.62           |00:0c:29:9a:59:73|Microsoft Windows 7 旗舰版           |Microsoft Windows 7 旗舰版           |64-bit|6.1.7600             |bios|Intel(R) Xeon(R) CPU E5-2680 0 @ 2.70GHz |4    |Physical Memory|8191.55 |4.83    |2   |255996.72  |0&#124;51199.34&#124;VMware Virtual disk SCSI Disk Device&#124;VMware Virtual disk SCSI Disk Device 1&#124;204797.37&#124;VMware Virtual disk SCSI Disk Device&#124;VMware Virtual disk SCSI Disk Device                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |C:&#124;53580132352&#124;4455071744&#124;0.08&#124;NTFS D:&#124;109887614976&#124;109790175232&#124;1.0&#124;NTFS E:&#124;104857595904&#124;104760311808&#124;1.0&#124;NTFS                  |1   |[00000007] Intel(R) PRO/1000 MT Network Connection&#124;00:0c:29:9a:59:73&#124;192.168.10.1&#124;192.168.10.62&#124;255.255.255.0                                                                                                                                                                                                                                                                                                                                                              |                 |                               |                     |