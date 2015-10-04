# Introduction on Debugging the CWE-170

## 1. Preparation
 * Download and install server [SonarQube](http://www.sonarqube.org/).
 * Follow the installation guide for [Sonar C++ Community plugin](https://github.com/wenns/sonar-cxx/wiki/Installation).
 * Donwload and install the analyzer [SonarQube Runner/Scanner](http://www.sonarqube.org/downloads/)
 * Please start the server and verify that the plugin is available.
 * Note that the plugin itself has the feature to include XML report created manually by [CPPcheck](https://github.com/wenns/sonar-cxx/wiki/Supported-configuration-properties). To manually create XML report, you will need CPPcheck installation, please refer to this [guide](https://www.google.com/search?q=--inclusive&ie=utf-8&oe=utf-8#q=cppcheck+--inclusive). When performing CPPcheck, please add the option **- - inconclusive** as the memory leak is false positive and by default, CPPcheck will overlook it.
 * By default, most of the rules are inactive, you will need to **reactivate** them.
 * Every project need to have one configuration file name: **sonar-project.properties**
 * My configuration file is already provided. Navigating through the root project folder **narnia** and execute the **sonar-runner**.

## 2. Instruction

##### #00:00 - #01:15. Manually inspect memory with Overwritten Memory Case.
In this scenario, we used **strncpy()** to copy 10 bytes from *bufs* to *dest*. It will copy exactly 10 bytes (including data of *bufs*) into *dest*. It is important to note that on the memory, the *bufs* was initialized after the *dest* 8 bytes offset. Because 10 bytes will be written to *dest*, the memory in bufs will be overwritten. The *dest* pointer is not appropriately terminated with NULL character '\0' and therefore, prone to CWE-170.

##### #01:15 - #02:44. Manually inspect memory with Memory Leak Case.

In this scenario, we used **strncpy()** to copy 3 bytes from *bufs* to *dest*. Since NULL-terminated wasn't handled properly, it will leave the *dest* becomes a not NULL-terminated string. While the initial size of *dest* is only 3, we can see the trailing value it holds is **Hel\x8e\x0f**, which is more than 3 bytes.

The *dest* pointer is not appropriately terminated with NULL character '\0' and therefore, prone to CWE-170.

##### #02:44 - #05:54. Static Analysis with SonarQube.
