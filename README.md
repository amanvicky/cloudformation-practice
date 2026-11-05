# CloudFormation — Practice Templates

Ten CloudFormation templates, each isolating one core concept of the service.

## Contents

| Template | Concept |
|---|---|
| `first-example.yaml` | Minimal stack |
| `init.yaml` | Getting started structure |
| `InputParameters.yaml` | `Parameters` + constraints |
| `MappingAndPseudo.yaml` | `Mappings` + pseudo-parameters (`AWS::Region`, …) |
| `Ref-function.yaml` | `Ref` / `Fn::GetAtt` |
| `functions.yaml` | Intrinsic functions (`Fn::Join`, `Fn::Sub`, …) |
| `multiResource.yaml` | Multi-resource stacks + dependencies |
| `PrintOutputs.yaml` | `Outputs` |
| `change-set.yml` | Change sets for safe updates |
| `update-example.yaml` | Stack update flows |
| `Cloudformation.pdf` | Course deck |

## Use

```bash
aws cloudformation deploy --template-file InputParameters.yaml --stack-name demo --parameter-overrides Env=dev
aws cloudformation create-change-set --stack-name demo --template-body file://change-set.yml --change-set-name feat
```

Pairs with [Tf-EC2](https://github.com/amanvicky/Tf-EC2) — same infra, two IaC philosophies.
