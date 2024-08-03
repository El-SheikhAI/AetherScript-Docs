# Infrastructure as Code (IaC) Integration  

## Overview  
AetherScript provides first-class support for Infrastructure as Code (IaC) workflows within continuous integration pipelines. This enables teams to:  
- **Version infrastructure configurations** alongside application code  
- **Automate environment provisioning** as part of pipeline execution  
- **Enforce security and compliance** through code review processes  
- **Achieve deterministic deployments** with declarative infrastructure definitions  

## Core Capabilities  
### 1. Multi-Provider IaC Execution  
Execute infrastructure definitions using:  
```yaml
plugins:
  - iac-terraform@1.2.0
  - iac-aws-cdk@0.9.1
  - iac-azure-arm@2.0.4
```  

### 2. State Management  
```bash
aetherscript iac state lock --backend=s3 --workspace=production
aetherscript iac state pull --format=json
```  

### 3. Real-Time Visualization  
![IaC Workflow Visualization](../assets/iac-visualizer.png)  

## Workflow Integration  
### Sample aetherscript.yaml  
```yaml
workflows:
  infra-deployment:
    steps:
      - name: Provision Base Infrastructure
        plugin: iac-terraform
        config:
          path: "./infra/core"
          vars:
            env: ${{ ENVIRONMENT }}
            region: us-west-2
          commands:
            - init
            - validate
            - plan -out=tfplan
            - apply tfplan

      - name: Deploy Networking
        plugin: iac-aws-cdk
        config:
          stacks:
            - VpcStack
            - SecurityGroups
          require-approval: false

      - name: Audit Configuration
        plugin: security-scanner
        ruleset: aws-cis-2.0
```  

## Security Considerations  
1. **Secrets Management**  
   ```bash
   aetherscript vault set terraform_aws_key --scope=env
   ```  

2. **State Backend Configuration**  
   ```hcl
   terraform {
     backend "s3" {
       bucket = "aetherscript-tfstate"
       key    = "prod/network"
       region = "us-east-1"
       encrypt = true
     }
   }
   ```  

3. **Least Privilege Execution**  
   ```yaml
   # .aether/permissions.yaml
   iac-terraform:
     allowed_actions:
       - plan
       - apply
     forbidden_flags:
       - "-destroy"
   ```  

## Extensibility  
### Custom IaC Providers  
```javascript
// plugins/iac-custom-provider.js
module.exports = {
  execute: async (config) => {
    const { engine, command, args } = config;
    return exec(`${engine} ${command} ${args.join(' ')}`);
  },
  validate: (config) => {
    // Custom validation logic
  }
};
```  

## Troubleshooting  
| Error Code | Resolution |  
|------------|------------|  
| `IAC-403`  | Verify state locking permissions |  
| `IAC-502`  | Check provider version compatibility |  
| `IAC-601`  | Validate environment variable propagation |  

## Best Practices  
1. **Version Pinning**  
   ```yaml
   plugins:
     - iac-terraform@1.5.7 # Pinned version
   ```  

2. **Modular Configuration**  
   ```bash
   aetherscript config split --input=monolithic.tf --output=modules/
   ```  

3. **Policy Enforcement**  
   ```yaml
   policies:
     infrastructure:
       require_plan_approval: true
       max_resource_changes: 50
   ```  

## Next Steps  
- [CLI Command Reference](../02-command-line-interface/04-iac-commands.md)  
- [Visualizer Configuration](../05-visualization/02-iac-diagrams.md)  
- [Plugin Development Guide](../08-extensibility/03-iac-plugins.md)