# CloudFormation — One Concept per Template

Ten CloudFormation templates, each isolating exactly one concept of the service, plus the course deck. Written back to back with my [Terraform](https://github.com/amanvicky/Tf-EC2) cards — same infra ideas, two IaC philosophies, which is what makes the comparison visible.

## Contents

| Template | Concept |
|---|---|
| `first-example.yaml` | Minimal stack |
| `init.yaml` | The big one — parameters, mappings, `cfn-init` bootstrap |
| `InputParameters.yaml` | `Parameters` with `AllowedValues` + region `Mappings` |
| `MappingAndPseudo.yaml` | `Mappings` + pseudo-parameters (`AWS::Region`, …) |
| `Ref-function.yaml` | `Ref` / `Fn::GetAtt` |
| `functions.yaml` | Intrinsic functions (`Fn::Join`, `Fn::Sub`, …) |
| `multiResource.yaml` | Multi-resource stacks and dependencies |
| `PrintOutputs.yaml` | `Outputs` |
| `change-set.yml` | Change sets for safe updates |
| `update-example.yaml` | Stack update flows |
| `Cloudformation.pdf` | Course deck |

## The centerpiece — `init.yaml`

Most template collections stop at "here's an EC2 instance". `init.yaml` is a full bootstrap: instance + security group, with the instance's entire configuration declared as metadata and executed by `cfn-init`:

```yaml
Metadata:
  AWS::CloudFormation::Init:
    config:
      packages:
        yum: {httpd: [], php: [], wget: [], unzip: [], git: []}
      commands:
        test:
          command: "wget https://www.tooplate.com/zip-templates/2119_gymso_fitness.zip && unzip 2119_gymso_fitness.zip && cp -r 2119_gymso_fitness/* /var/www/html/"
      files:
        /var/www/html/hello.html:
          content: !Sub |
            <h1>Welcome to CloudFormation.</h1>
      services:
        sysvinit:
          httpd: {enabled: true, ensureRunning: true}
```

...and `UserData` wired to run it and signal back:

```yaml
'Fn::Base64': !Sub |
    #!/bin/bash -xe
    yum install -y aws-cfn-bootstrap
    /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource WebServer --region ${AWS::Region}
    /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebServer --region ${AWS::Region}
```

The AMI comes from a `FindInMap` over four regions (`AmiRegionMap`), the instance type is a constrained parameter (`AllowedValues`: t2.micro → m1.large), the key name is an `AWS::EC2::KeyPair::KeyName`. The security group opens 80 to the world and 22 to a single `/32` — the honest lab compromise.

Note the deploy command inside `commands:` — a Tooplate zip pulled and unpacked into the docroot. That's the same deployment my bash scripts ([Bash-scripting](https://github.com/amanvicky/Bash-scripting), [AWS-basic](https://github.com/amanvicky/AWS-basic)) do by hand: the ladder from typed commands → script → cfn-init.

## Deploy + change safely

```bash
aws cloudformation deploy --template-file InputParameters.yaml \
  --stack-name demo --parameter-overrides NameOfService=webserv

aws cloudformation create-change-set --stack-name demo \
  --template-body file://change-set.yml --change-set-name feat
```

Change sets are the CloudFormation habit worth keeping: preview the diff before the stack mutates.
