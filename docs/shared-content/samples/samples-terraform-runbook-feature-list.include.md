**Pattern - IaC**

- Dynamic worker army
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-68/operations/runbooks/Runbooks-1893/process/RunbookProcess-Runbooks-1893" target="_blank">Create Infrastructure</a>: <i>Spins up the worker army</i>
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-68/operations/runbooks/Runbooks-1894/process/RunbookProcess-Runbooks-1894" target="_blank">Destroy Infrastructure</a>: <i>Tears down the worker army.</i>
- Random Quotes - Azure
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-1851/operations/runbooks/Runbooks-1897/process/RunbookProcess-Runbooks-1897" target="_blank">Create and Configure Terraform Infrastructure</a>: <i>Creates necessary infrastructure in Azure [using Terraform](https://dev.azure.com/octopussamples/_git/Azure-Terraform-RandomQuotes) and configures it for application deployment.</i>
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-1851/operations/runbooks/Runbooks-1898/process/RunbookProcess-Runbooks-1898" target="_blank">Destroy Terraform Resources and Delete Deployment Targets</a>: <i>Destroys created Terraform resources and removes registered deployment targets.</i>
- Random Quotes AWS
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-1861/operations/runbooks/Runbooks-1923/process/RunbookProcess-Runbooks-1923" target="_blank">Create Infrastructure</a>: <i>Creates EC2 instances using Terraform, registers them as deployment targets with Octopus, and then provisions them with the necessary tooling for application deployment.</i>
   - <a href="https://samples.octopus.app/app#/Spaces-48/projects/Projects-1861/operations/runbooks/Runbooks-1924/process/RunbookProcess-Runbooks-1924" target="_blank">Destroy Infrastructure</a>: <i>Destroys created EC2 instances and all supporting resources created through Terraform along with deregistering them as targets within Octopus.</i>
    
**Target - Serverless**

- AWS Subscriber S3
   - <a href="https://samples.octopus.app/app#/Spaces-1/projects/Projects-1781/operations/runbooks/Runbooks-1821/process/RunbookProcess-Runbooks-1821" target="_blank">Spin Up Subscriber Infrastructure</a>
   - <a href="https://samples.octopus.app/app#/Spaces-1/projects/Projects-1781/operations/runbooks/Runbooks-1823/process/RunbookProcess-Runbooks-1823" target="_blank">Tear Down AWS Subscriber Infrastructure</a>
    
**Target - Windows**

- eShopOnWeb
   - <a href="https://samples.octopus.app/app#/Spaces-202/projects/Projects-1481/operations/runbooks/Runbooks-1901/process/RunbookProcess-Runbooks-1901" target="_blank">Create Infrastructure</a>: <i>Stands up an Azure VM with IIS and SQL Server Express using Terraform for eShopOnWeb to be deployed on to.</i>
   - <a href="https://samples.octopus.app/app#/Spaces-202/projects/Projects-1481/operations/runbooks/Runbooks-1902/process/RunbookProcess-Runbooks-1902" target="_blank">Destroy Infrastructure</a>: <i>Destroys the Azure VM and infrastructure created for this project.</i>
    
**Tenants - Regions**

- To Do - Linux
   - <a href="https://samples.octopus.app/app#/Spaces-102/projects/Projects-148/operations/runbooks/Runbooks-1283/process/RunbookProcess-Runbooks-1283" target="_blank">Runbook With Packages</a>
