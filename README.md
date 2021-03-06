# Azure SDK for Golang
This project provides a Golang package that makes it easy to consume and manage Microsoft Azure Services.

# Installation
- Install Golang: https://golang.org/doc/install
- Get Azure SDK package: 

```sh
go get github.com/MSOpenTech/azure-sdk-for-go
```
- Install: 

```sh
go install github.com/MSOpenTech/azure-sdk-for-go
```

# Usage

Download publish settings file from https://manage.windowsazure.com/publishsettings.
Create linux VM:

```go
package main

import (
    "fmt"

    "github.com/MSOpenTech/azure-sdk-for-go/management"
    "github.com/MSOpenTech/azure-sdk-for-go/management/hostedservice"
    "github.com/MSOpenTech/azure-sdk-for-go/management/virtualmachine"
    "github.com/MSOpenTech/azure-sdk-for-go/management/vmutils"
)

func main() {
    dnsName := "test-vm-from-go"
    storageAccount := "mystorageaccount"
    location := "West US"
    vmSize := "Small"
    vmImage := "b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04-LTS-amd64-server-20140724-en-us-30GB"
    userName := "testuser"
    userPassword := "Test123"

    client, err := management.ClientFromPublishSettingsFile("path/to/downloaded.publishsettings", "")
    if err != nil {
        panic(err)
    }

    // create hosted service
    requestId, err := hostedservice.NewClient(client).CreateHostedService(dnsName, location, "", dnsName, "")
    if err != nil {
        panic(err)
    }
    err = client.WaitAsyncOperation(requestId)
    if err != nil {
        panic(err)
    }

    // create virtual machine
    role, err := vmutils.NewVmConfiguration(dnsName, vmSize)
    if err != nil {
        panic(err)
    }
    vmutils.ConfigureDeploymentFromPlatformImage(&role,
        vmImage, fmt.Sprintf("http://%s.blob.core.windows.net/sdktest/%s.vhd", storageAccount, dnsName), "")
    vmutils.ConfigureForLinux(&role, dnsName, userName, userPassword)
    vmutils.ConfigureWithPublicSSH(&role)

    requestId, err = virtualmachine.NewClient(client).CreateDeployment(role, dnsName)
    if err != nil {
        panic(err)
    }
    err = client.WaitAsyncOperation(requestId)
    if err != nil {
        panic(err)
    }
}
```

# License
[Apache 2.0](LICENSE-2.0.txt)
