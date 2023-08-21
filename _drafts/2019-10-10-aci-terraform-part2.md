---
layout: post
excerpt: "Intro into Cisco ACI and Terraform"
title: Intro into Cisco ACI and Terraform
categories: [ACI, Terraform]
author: Matyas
published: false
---


### Intro
Recently I have started to explore Terraform as replacement of Ansible. Even though there are some great use cases for Ansible to manage your infrastructure as code through the time I have found out there are certain limitations. Luckily what I have also found out what that most of those limitations can be addressed by Terraform.

### Network Automation and Infrastracture as Code
Network automation has different meanings for the different people. Some of us will use the network automation to automate day to day tasks with simple scripts some of us will use the network automation to provision the network and some of us to change control the whole infrastructure. There is no right or wrong approach. Each of us has specific use cases for using the network automation due to the level of our skills or our team, the domain which we are managing or the network or solution we are managing. There is different way to manage your Catalyst 2960s with very small amount of Python skills and managing ACI in the team of experienced SREs already using Ansible and Terraform.

The whole point of managing your infrastructure as code that it should bring the same benefits (and much more) as if you are managing the code of modern applications. The engineer should be able to change control the infrastructure, they should be able to run the code which represent the change in the testing environment before it will be implemented into the production, engineer should be able to run the tests against the new code and than implement the new code. All of this above should be fully automated so every time the engineer will make the change it will automatically save the new version of the code, valid the code on syntax, run the code in the staging environment and tests against it. In case the tests passed it will be automatically pushed the code into the production.

Other great benefit of treating your network as code is when actually something goes really wrong. Imagine that the engineer will make some change and it will go terribly wrong. The engineer or the team knows exactly what change has been made but it would take hours to remove the configuration. Luckily they run the infrastructure as code so with the change control they can easily find the last time the infrastructure had no issues and they can easily reverse the code into the pre-change state.

Now when you (hopefully) understand what are the benefits of managing your infrastructure as code can we actually manage any network as code? The answer is ‘it depends’. Each network is different from technology and architecture perspective which can make it little bit difficult to automate. Don’t take me wrong it does not always have to be vendor’s fault and actually on many occassions I have seen that engineers haveing the latest technologies but were unable to adapt the new solution and get the most out of it.

### Cisco ACI
The reason why I have decided to use Cisco ACI as the network which I want to automate is that in my opinion it is one of the leading SDN DC solutions on the market which almost every enterprise can afford.

Cisco ACI has all the important attributes of SDN - centralized management, open APIs and large community of users which in the end enables to manage DC infrastructure as code.

Cisco ACI uses management cluster called APIC to centrally manage the whole fabric. Cisco APIC has nortbound APIs which can be leveraged by Python libraries via REST API requests or via automation tools like Ansible or Terraform.

### Terraform vs. Ansible
I will now very high level describe the difference between Ansible and Terraform. Ansible has been initially developed for the server management and provisioning. When the administrator has used Ansible it was usually to install new software or upgrade the current eventually deploy new configuration which can be easily reloaded by the application. If you wanted to remove something from the configuration you removed the current configuration, upload the new configuration and reloaded the processes. Few years ago we have started see the trend mainly accelerated by Google using Puppet (alternative to Ansible using the end point agent) to managing the network infrastructure. The challenge is that legacy and even the modern network usually do not allow to install any agents (which is not the case of Google network which is mostly open sourced) therefore the logical choice from the community was Ansible. We have seen initially that community was developing new modules for the solutions from Cisco or F5 but due to the huge adopotion it has been started to being maintained and owned by vendors.

If you used Ansible in ACI and you removed EPG from Ansible playbook nothing really happened. The EPG has stayed in your fabric and you had inconsistency in what you had in Ansible playbooks and your ACI Fabric.

Terraform address these issues with storing the state of the network or fabric in the external file. Terraform takes so called declarative approach. What does it mean? Imagine you have the ACI fabric where you want to change the name of EPG. With Ansible you will have to remove the EPG which you want to rename and create new one with new name and eventually the same attributes. Terraform helps in this case because you will **declare** the new state of your fabric and Terraform will take care of it which means it will remove the EPG and create the new EPG with new name and with the same attributes.

In this series of following posts I will be focusing only on Terraform. I may will run separate series for Ansible in the future. So this is it. I will not boring you with more theory and in the next post I will focu son how to use Terraform on Cisco ACI.
