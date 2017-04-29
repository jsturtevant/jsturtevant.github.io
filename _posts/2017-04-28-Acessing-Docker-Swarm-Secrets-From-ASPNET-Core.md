---
layout: post
title: Accessing Docker Swarm Secrets from ASP.NET Core
date: "2017-04-28"
categories:
  - docker
  - aspnetcore
---

[Docker Swarm introduced Secrets](https://docs.docker.com/engine/swarm/secrets/) in version 1.13, which enables your share secrets across the cluster securely and only with the containers that need access to them.  The secrets are encrypted during transit and at rest which makes them a great way to distribute connection strings, passwords, certs or any other sensitive information.  

I will leave it to the official documentation to describe exactly how all this works but when you give a service access to the secret you essentially give it access to an in-memory file.  This means your application needs to know how to read the secret from the file to be able to use the application.  

This article will walk through the basics of reading that file from an ASP.NET Core application.  The basic steps would be the same for ASP.NET 4.6 or any other language.  

> Note: In ASP.NET Core 2.0 (still in preview as of this writing) will have this functionality as a [Configuration Provider](https://github.com/aspnet/Configuration/tree/2519ffc7fc1071befeae021551c6203126e117d3/src/Microsoft.Extensions.Configuration.DockerSecrets) which you can simply add the nuget and wire up.

##  Create a Secret
We will start by creating a secret on our swarm.  Log into a Manager node on your swarm and add a secret by running he following command:

```bash
echo 'yoursupersecretpassword' | docker secret create secret_password -
```

You can confirm that the secret was created by running:

```bash
$ docker secret ls
ID                          NAME                CREATED             UPDATED
8gv5uzszlgtzk5lkpoi87ehql   secret_password     1 second ago        1 second ago
```

Notice that you will not be list the value of the secret. For any containers that you explicitly give access to this secret, they would be able to read the value as a string at the path:

```
/run/secrets/postgres_password
```

## Reading Secrets
Your next step is to read the secret inside you ASP.NET Core application. Before Docker Secrets were introduced you had a few options to pass the secrets along to your service:

- use environment variables for secrets.  This ends up being a [bad idea in a Docker containers](https://github.com/moby/moby/issues/13490) because any one who can do an ```inspect``` can see the secrets. 

- use a Secret manager like [Hashi Corp's Vault](https://www.vaultproject.io/) or [Azure's Key Vault](https://azure.microsoft.com/en-us/services/key-vault/).  This is a great way to store and manage secrets and is still a valid way to store secrets.  

## Modifying your ASP.NET Core to read Docker Swarm Secrets
During development, you may not want to read the value from a Swarm Secret for simplicity sake.  To accommodate this you can check to see if a Swarm Secret is available and if not read the ASP.NET Configuration variable instead.  This enables you to load the value from any of the other configuration providers as a fallback.  The code is fairly straight forward:

```csharp
public string GetSecretOrEnvVar(string key)
{
  const string DOCKER_SECRET_PATH = "/run/secrets/";
  if (Directory.Exists(DOCKER_SECRET_PATH))
  {
    IFileProvider provider = new PhysicalFileProvider(DOCKER_SECRET_PATH);
    IFileInfo fileInfo = provider.GetFileInfo(key);
    if (fileInfo.Exists)
    {
      using (var stream = fileInfo.CreateReadStream())
      using (var streamReader = new StreamReader(stream))
      {
        return streamReader.ReadToEnd();
      }
    }
  }
  
  return Configuration.GetValue<string>(key);
}
```

This checks to see if the directory exists and if it does checks for a file with a given key. Finally, if that key exists as a file then reads the value.  

You can use this function anytime after the configuration has been loaded from other providers.  You will likely call this function somewhere in your ```startup.cs``` but could be anywhere you have access to the ```Configuraiton``` object.  Note that depending on your set-up you may want to tweak the function to not fall back on the ```Configuration``` object.  As always consider your environment and adjust as needed.

##  Create a service that has access to the Secret
The final step is to create a service in the Swarm that has access to the value:

```bash
docker service  create --name="aspnetcore" --secret="secret_password" myaspnetcoreapp
```

or in your ```Compose``` file version 3.1 and higher (only including relevant info for example):

```yml
version: '3.1'

services:
  aspnetcoreapp:
    image: myaspnetcoreapp
  secrets:
    - secret_password

secrets:
  secret_password:
    external: true
```

## Creating an ASP.NET Core Configuration Provider
I am going to leave it to the reader to create an ASP.NET Core Configuration Provider (or wait until it is a NuGet package supplied with ASP.NET Core 2.0).  If you decide to turn the code into a configuration provider (or use ASP.NET Core 2.0's) you could simply use in you ```start.cs``` and it might look something like:

```csharp
 public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddEnvironmentVariables()
        .AddDockerSecrets();
    Configuration = builder.Build();
}
```

> IMPORTANT: this is just an example the actual implementation might look slightly different.

## Conclusion
Docker Secrets are a good way to keep your sensitive information only available to the services that need access to it.  You can see that you can modify your application code in a fairly straight forward way that supports backward compatibility.  You can learn more about managing secrets at [Docker's documentation](https://docs.docker.com/engine/swarm/secrets/).  
