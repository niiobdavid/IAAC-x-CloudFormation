<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Infrastructure as Code with CloudFormation

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-devops-cloudformation-updated)

**Author:** davidniiamui@gmail.com  
**Email:** niiobdavid@gmail.com

---

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_bd8b836b)

---

## Introducing Today's Project!

In this project, I will demonstrate how to set u a CloudFormation template.
I'm doing this project to learn about infrastructure as code, and how I can apply this to my architecture (the CI/CD architecture).
By the end of this project, resources like CodeArtifact, CodeBuild, CodeConnection and more will be defined together in a single template

### Key tools and concepts

Services I used were CloudFormation and the code suite of services.
Key concepts I learned include infrastructure as code, how to use the IaC generator, and resolving circular dependencies and resources that might depend on other resources to be created first.
I also learned how to write my own resource definitions in the secret mission.

### Project reflection

This project took me approximately 3 hours including documentation and troubleshooting time.
The most challenging part was editing the CloudFormation template, and finding each resource's IDs.
It was most rewarding to deploy the template and see it live in my account.

This project is part six of a series of DevOps projects where I'm building a CI/CD pipeline! I'll be working on the next project later today.
Final lap💪🏽 Let's go!🐽

---

## Generating a CloudFormation Template

The IaC Generator is a tool in the CloudFormation console that helps with writing CloudFormation templates much faster. 
It works in a three-step process where the IaC generator first scans the resources, then lets me create a template based on the resources it found, then finally allows me to launch that template to deploy the resources described inside.

A CloudFormation template is a text file that defines the resources I want to deploy inside.
The resources that I added to my template include CodeArtifact domain and repositories, CodeDeploy Application, IAM roles and policies, and S3 bucket.

The resources I couldn’t add to my template were CodeBuild projects and CodeDeploy deployment group.
This is because both resources are quite complicated in nature (there are lots of settings I would need to configure.
The IaC generator is not yet capable of scanning these settings, so I'll have to write these definitions manually

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_0495b046)

---

## Template Testing

Before testing my template, I deleted the existing resources in my account because they share the same name as the resources in my generated template.
This will cause an error so I deleted the existing resources to avoid that conflict/overlap.

I tested my template by launching a stack using the template I generated.
The result of my first test was CREATE_FAILED because my IAM policy tried to attach to an IAM role that didn't exist yet.
This is because the role is getting created in the exact same template, so it was not yet ready by the time the policy wanted to attach to it.

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_f56730fd)

---

## DependsOn

To resolve the error, I opened the template in a code editor and updated it manually with a DependsOn statement/attribute.
The DependsOn attribute means a resource needs to wait for another resource in the same template to be created first.

The DependsOn line was added to four different parts of my template: all 4 policies for CodeBuild, which are policies that allow access to CodeArtifact, CodeConnection, CloudWatch, and EC2 instances (CodeBuild's base policy).
For the CodeArtifact policy, I had to add TWO roles that it depends on; the CodeBuild service role, And the EC2 instance role.

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_f0df8018)

---

## Circular Dependencies

I gave my CloudFormation template another test! But this time there was a new error - circular dependencies.
This error tells me that CloudFormation doesn't know what resource it should deploy first - as it turns out, the Policies depend on the Roles but the Roles also depend on the Policies to be created first.
This puts CloudFormation in a loop so the template is unusable. 

To fix this error, I revisited the template and tried to understand why both Roles and Policies were referencing each other in the template.
As it turns out, the Roles have a section called 'ManagedPolicyARN' that references the policies they use.
Then the Policy definitions have the 'DependsOn' section that asks for the roles to be created first.
To resolve this loop, I simply had to delete the 'ManagedPolicyARN' sections off of the roles.

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_e6fd85ed)

---

## Manual Additions

In a project extension, I manually defined two more resources; a CodeBuild project And a CodeDeploy deployment group

I also had to make sure the references were consistent in this template, so I edited the values for the S3 Bucket ID, CodeDeploy Application ID, and service role IDs to match the ID of those resources in the template.

I also introduced Parameters, which are like form fields in a CloudFormation template - instead of hardcoding a value inside the template, I can get the CloudFormation stack to ask the user the value or 'xyz' when they launch the resources.
In this example, I created parameters for my GitHub account information (account name and repo name)

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_1cee0428)

---

## Success!

I could verify all the deployed resources by visiting the hyperlinks provided for each deployed resource.
I could see all the resources defined in the template. from the IAM policies to the manually defined CodeBuild project.

![Image](http://learn.nextwork.org/genuine_navy_mysterious_monkey/uploads/aws-devops-cloudformation-updated_bd8b836b)

---

---
