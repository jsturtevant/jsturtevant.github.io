---
layout: post
title: The differences between Azure Service Configuration(.cscfg) and Azure Service Definition(.csdef) Files
date: '2015-08-11'
categories:
  - azure
---

Remembering the differences between [Azure Service Configuration(.cscfg)](#cscfg) and [Azure Azure Service Definition(.csdef)](#csdef) files can be challenging.  Not only do you need to know how to configure the Cloud Services for the Azure [70-532](https://www.microsoft.com/learning/en-us/exam-70-532.aspx) and the [70-533](https://www.microsoft.com/learning/en-us/exam-70-533.aspx) exams, you need to be able to implement it for you real world projects ;-)

I have complied some of the major configurations that each file contains.  Having all the optional configurations for each file in one location really helped me remember the differences and have one place to go when I needed to look it up.  Hope it helps!

## <a name="cscfg"></a> Azure Service Configuration(.cscfg)

### Certification thumbprints

  <Role name="Deployment">

    <Certificates>
        <Certificate name="SampleCertificate"
            thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff"
            thumbprintAlgorithm="sha1" />
    </Certificates>

</Role>

From <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-configure-ssl-certificate/>

### Access Controls
  <ServiceConfiguration>
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="aclName1">
        <Rule order="<rule-order>" action="<rule-action>" remoteSubnet="<subnet-address>" description="rule-description"/>
      </AccessControl>
    </AccessControls>

From <https://msdn.microsoft.com/en-us/library/azure/jj156091.aspx>

### Endpoint ACLS
   <EndpointAcls>
      <EndpointAcl role="<role-name>" endpoint="<endpoint-name>" accessControl="<acl-name>"/>
    </EndpointAcls>

From <https://msdn.microsoft.com/en-us/library/azure/jj156091.aspx>

### DNS
    <Dns>
      <DnsServers>
        <DnsServer name="<server-name>" IPAddress="<server-address>" />
      </DnsServers>
    </Dns>

From <https://msdn.microsoft.com/en-us/library/azure/jj156091.aspx>

### Virtual networks
     <VirtualNetworkSite name="<site-name>"/>

From <https://msdn.microsoft.com/en-us/library/azure/jj156091.aspx>

### Subnets and Reserved IPS
  <AddressAssignments>
      <InstanceAddress roleName="<role-name>">
        <Subnets>
          <Subnet name="<subnet-name>"/>
        </Subnets>
      </InstanceAddress>
      <ReservedIPs>
        <ReservedIP name="<reserved-ip-name>"/>
      </ReservedIPs>
    </AddressAssignments>

From <https://msdn.microsoft.com/en-us/library/azure/jj156091.aspx>


##<a name="csdef"></a> Azure Azure Service Definition(.csdef)

### Startup tasks
  <WebRole name="WebRole1" vmsize="Small">
    <Startup>
     <Task commandLine="install.cmd" executionContext="elevated" />
    </Startup>

From <http://weblogs.asp.net/shijuvarghese/startup-tasks-for-windows-azure-roles>

### Environment variables
  <WebRole name="WebRole1">
      <Runtime>
         <Environment>
            <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
         </Environment>
      </Runtime>
   </WebRole>

From <https://msdn.microsoft.com/en-us/library/azure/gg432991.aspx>

### Certification storage:
  <WebRole name="CertificateTesting" vmsize="Small">
    <Certificates>
        <Certificate name="SampleCertificate"
                     storeLocation="LocalMachine"
                     storeName="CA" />
    </Certificates>
</WebRole>

From <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-configure-ssl-certificate/>

### EndPoints
  <WebRole name="CertificateTesting" vmsize="Small">
    <Endpoints>
        <InputEndpoint name="HttpsIn" protocol="https" port="443"
            certificate="SampleCertificate" />
    </Endpoints>
</WebRole>

From <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-configure-ssl-certificate/>

### Bindings
  <WebRole name="CertificateTesting" vmsize="Small">
    <Sites>
        <Site name="Web">
            <Bindings>
                <Binding name="HttpsIn" endpointName="HttpsIn" />
            </Bindings>
        </Site>
    </Sites>
</WebRole>

From <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-configure-ssl-certificate/>
