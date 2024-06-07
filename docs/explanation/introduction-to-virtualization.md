(introduction-to-virtualization)=
# Virtualisation

Virtualization is a technology that allows you to create safe, isolated environments on your server. For developers, this often means creating self-contained sandboxes for development and testing that cannot negatively affect the host machine. For systems administrators, it allows resources to be scaled to meet changing demand, giving greater control and flexibility in managing infrastructure.

The virtualization stack is made using layers of **abstraction**. Each layer hides some of the complexity of the layer (or layers) beneath, presenting an increasingly high-level view of the technology. This makes the underlying technology progressively easier to understand and work with. 

## Virtual machines

Virtual machines (VMs) are essentially computers-within-computers. Every VM includes its own operating system and simulated resources, making it completely independent of the host machine (and any other VM). Although more resource-intensive (and slower to boot) than a container, a VM provides strong isolation and reduces the need for additional hardware when running different operating system environments. To find out more, see this overview of [the different VM tools and technologies](vm-tools-in-the-ubuntu-space.md) available in the Ubuntu space.

## Containers

Containers, on the other hand, are a more lightweight virtualization technology. They share the operating system of the host machine, so they are much quicker to provision when demand for resources is high. They are often used for packaging and running applications, since they contain everything the application needs (including any required dependencies and libraries). This ensures consistency across different environments. Containers come in two main flavours: **system** containers, and **application** containers.

### System containers

System containers simulate a full machine in a similar way to a VM. However, since containers run on the host kernel they don't need to install an operating system, making them quick to start and stop. They are often used for separating user spaces.

### Application containers

Application containers package all of the components needed for a specific application or process to run, including any required dependencies and libraries. This means the application can run consistently, even across different environments, without running into problems of missing dependencies. Application containers are particularly useful for running microservices.

For more details about container tools available in the Ubuntu space, [take a look at this overview](container-tools-in-the-ubuntu-space.md).