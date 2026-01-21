# AWS Parameter Store Audit & Migration Solutions Guide

**Document Created:** January 21, 2026  
**Environment:** UAT Account  
**Primary Objective:** Identify services/resources using plain text String parameters for decommissioning or upgrading to SecureString

---

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Solution 1: CloudTrail Event History Analysis](#solution-1-cloudtrail-event-history-analysis)
3. [Solution 2: IAM Role Reverse Engineering](#solution-2-iam-role-reverse-engineering)
4. [Solution 3: Service-by-Service Manual Review](#solution-3-service-by-service-manual-review)
5. [Solution 4: Infrastructure-as-Code Repository Search](#solution-4-infrastructure-as-code-repository-search)
6. [Solution 5: Application Code Repository Search](#solution-5-application-code-repository-search)
7. [Solution 6: Tag-Based Inventory](#solution-6-tag-based-inventory)
8. [Solution 7: Controlled Deprecation Testing](#solution-7-controlled-deprecation-testing-canary-approach)
9. [Solution 8: VPC Flow Logs + Endpoint Analysis](#solution-8-vpc-flow-logs--endpoint-analysis)
10. [Solution 9: AWS Config Resource Inventory](#solution-9-aws-config-resource-inventory)
11. [Solution 10: CloudWatch Logs Insights](#solution-10-cloudwatch-logs-insights)
12. [Recommended Hybrid Approach](#recommended-hybrid-approach)
13. [Migration Strategy](#migration-strategy)
14. [Decision Matrix](#decision-matrix)

---

## Problem Statement

**Current State:**
- UAT environment has Parameter Store parameters of type "String" (plain text)
- These parameters are insecure - readable by anyone with `ssm:GetParameter` permission
- May contain sensitive data (passwords, API keys, connection strings)

**Required Action:**
- Identify which services/resources are using these plain text parameters
- Either decommission unused parameters OR upgrade them to SecureString (encrypted with KMS)

**Challenge:**
- Cannot safely migrate without knowing dependencies
- Breaking changes if services aren't updated for KMS permissions
- Need comprehensive visibility into actual usage

---

## Solution 1: CloudTrail Event History Analysis

### Description
Tracks all API calls to Parameter Store including GetParameter, GetParameters, and GetParametersByPath operations.

### How It Helps
- Shows which IAM principals (Lambda roles, EC2 instance profiles, users) accessed which parameters
- Provides timestamps of when parameters were accessed
- Reveals actual usage patterns, not just theoretical permissions
- Can identify the source IP and user agent making requests

### Implementation Method
- **Console:** CloudTrail → Event History → Filter by event source `ssm.amazonaws.com`
- **Extended Analysis:** Export CloudTrail logs to S3, query with Athena for historical data beyond 90 days
- **Automated:** Use CloudTrail Insights for anomaly detection

### Pros
- Shows real, actual usage
- Free for 90-day history in console
- No additional setup if CloudTrail already enabled
- Provides concrete evidence of access patterns
- Can identify unused parameters (no access in 90 days)

### Cons
- Limited to 90 days in console view (free tier)
- Won't catch parameters defined but never accessed
- Misses dormant or rarely-used parameters
- May show test/development access not relevant to production
- Doesn't show future planned usage
- Can miss parameters accessed before the 90-day window

### Best For
- Active, frequently-accessed parameters
- Identifying immediate dependencies
- Quick initial assessment
- Verifying which parameters are truly in use

### Time Estimate
- **Manual Console Review:** 1-2 hours for small environments
- **Athena Query Setup:** 30-60 minutes initial setup, 15 minutes per query
- **Ongoing Monitoring:** 10-15 minutes per check

---

## Solution 2: IAM Role Reverse Engineering

### Description
Find all IAM roles with SSM GetParameter permissions, then trace backward to identify which AWS resources use those roles.

### How It Helps
- Identifies potential parameter users even if they haven't accessed parameters recently
- Shows "could access" rather than just "did access"
- Uncovers over-provisioned permissions (security audit benefit)
- Maps the permission landscape comprehensively

### Implementation Method
- List all IAM roles
- For each role, check inline and managed policies for `ssm:GetParameter*` permissions
- Cross-reference roles with:
  - EC2 instance profiles
  - Lambda function execution roles
  - ECS task roles and execution roles
  - CodeBuild service roles
  - Step Functions execution roles
  - Glue job roles

### Pros
- More comprehensive than CloudTrail for dormant resources
- Identifies potential security risks (excessive permissions)
- Shows complete permission landscape
- Catches resources that haven't run recently
- Good for compliance documentation
- Reveals orphaned permissions

### Cons
- Labor intensive without automation
- Creates false positives (permissions but no usage)
- Roles might have broad `ssm:*` permissions making specifics unclear
- Requires understanding of IAM policy evaluation
- Doesn't show which specific parameters are accessed
- May require checking hundreds of roles in large environments

### Best For
- Comprehensive security audits
- Identifying all potential users
- Environments with good IAM hygiene
- Uncovering over-provisioned access

### Time Estimate
- **Small Environment (<20 roles):** 1-2 hours
- **Medium Environment (20-100 roles):** 3-5 hours
- **Large Environment (100+ roles):** 6-10 hours or automation required

---

## Solution 3: Service-by-Service Manual Review

### Description
Systematically check each AWS service for parameter references by examining configurations, environment variables, and settings.

### Services to Check

#### Lambda Functions
- Environment variables (may reference parameter names)
- Function code (SDK calls to get_parameter)
- Layers (shared code accessing parameters)
- Container image configurations

#### ECS/Fargate
- Task definition environment variables
- Task definition secrets (explicit Parameter Store references)
- Container definitions
- Task execution roles

#### EC2 Instances
- User data scripts (bootstrap scripts often fetch parameters)
- SSM Run Command documents
- Instance metadata
- CloudWatch agent configurations

#### CodeBuild/CodePipeline
- buildspec.yml environment variables
- Parameter store references in build specifications
- Pipeline action configurations
- Custom actions and Lambda integrations

#### Elastic Beanstalk
- Environment properties
- .ebextensions configuration files
- Platform hooks

#### AppConfig
- Configuration profiles
- Deployment strategies
- Feature flags

#### CloudFormation/SAM
- Template parameter sections
- Dynamic references: `{{resolve:ssm:parameter-name}}`
- Stack outputs and exports
- Nested stack parameters

#### Other Services
- API Gateway stage variables
- Step Functions state machine definitions
- Glue job parameters
- SageMaker notebook instance lifecycle configs
- Systems Manager Automation documents

### Pros
- Most thorough and comprehensive approach
- Catches static references CloudTrail might miss
- Creates complete documentation of infrastructure
- Identifies configuration drift
- Good for knowledge transfer and team education
- Finds parameters in all states (active, dormant, planned)

### Cons
- Extremely time-consuming without automation
- Easy to miss services or configurations
- Requires deep knowledge of each AWS service
- Repetitive and tedious work
- Prone to human error
- May need to revisit as infrastructure changes

### Best For
- Small to medium environments
- Initial comprehensive audit
- Creating infrastructure documentation
- Teams new to the environment

### Time Estimate
- **Per Service Review:** 30-60 minutes
- **Complete Review (8-10 services):** 6-12 hours
- **Ongoing Updates:** 1-2 hours per quarter

---

## Solution 4: Infrastructure-as-Code Repository Search

### Description
Search your Terraform, CloudFormation, CDK, or other IaC repositories for parameter references using code search tools.

### How It Helps
- Finds parameters defined in code even if not yet deployed
- Shows infrastructure intent and planned usage
- Faster than manual service checking
- Reveals parameters in development/feature branches
- Documents parameter lineage

### Search Patterns by IaC Tool

#### Terraform
```
aws_ssm_parameter
data.aws_ssm_parameter
var.ssm_parameter
/your-app-prefix/
```

#### CloudFormation
```
AWS::SSM::Parameter
{{resolve:ssm:
{{resolve:ssm-secure:
Fn::Sub
```

#### AWS CDK (TypeScript/Python)
```
ssm.StringParameter.fromStringParameterName
ssm.getParameter
ParameterStore.
```

#### Pulumi
```
aws.ssm.Parameter
getParameter
```

#### Ansible
```
aws_ssm_parameter_store
ssm_parameter_store
```

### Search Tools
- **GitHub/GitLab Search:** Built-in repository search
- **grep/ripgrep:** Command-line search across repositories
- **IDE Find in Files:** Visual Studio Code, IntelliJ
- **Code scanning tools:** SonarQube, CodeQL

### Pros
- Very fast with proper search tools
- Shows infrastructure intent
- Catches parameters not yet deployed
- Documents IaC dependencies
- Can search across multiple repositories simultaneously
- Finds parameters in all environments (dev, staging, prod)
- Good for change impact analysis

### Cons
- Only works if you use Infrastructure as Code
- Doesn't catch manually created resources (ClickOps)
- Code might be outdated vs actual deployment
- Doesn't show runtime behavior
- May miss dynamic parameter names (constructed at runtime)
- Requires access to all repositories

### Best For
- Organizations with strong IaC practices
- Quick initial scans
- Change impact analysis before migration
- Documentation generation

### Time Estimate
- **Initial Setup:** 15-30 minutes
- **Search Execution:** 5-15 minutes
- **Result Analysis:** 30-60 minutes

---

## Solution 5: Application Code Repository Search

### Description
Search application source code repositories for AWS SDK calls that retrieve SSM parameters.

### How It Helps
- Direct source of truth for parameter usage
- Finds hardcoded parameter names in application code
- Works across all programming languages
- Identifies which applications own which parameters
- Shows how parameters are being used (read-only, cached, etc.)

### Search Patterns by Language

#### Python (boto3)
```python
get_parameter(
ssm.get_parameter
get_parameters(
get_parameters_by_path(
client('ssm')
```

#### JavaScript/TypeScript (AWS SDK v2/v3)
```javascript
getParameter(
SSM.getParameter
GetParameterCommand
SSMClient
new AWS.SSM()
```

#### Java
```java
getParameter(
GetParameterRequest
AWSSimpleSystemsManagement
SsmClient
```

#### Go
```go
GetParameter
GetParameterInput
ssm.New
```

#### C# / .NET
```csharp
GetParameterAsync
AmazonSimpleSystemsManagementClient
GetParameterRequest
```

#### Ruby
```ruby
get_parameter
Aws::SSM::Client
```

### Additional Search Terms
- Parameter names themselves (e.g., `/myapp/db/password`)
- Environment variable names that might reference parameters
- Configuration file keys

### Pros
- Direct source of truth
- Works across all languages and frameworks
- Can find dynamic parameter construction logic
- Helps identify which team/application owns parameters
- Shows usage context (how parameters are used)
- Good for refactoring and modernization efforts

### Cons
- Requires access to all source code repositories
- Dynamic parameter names are harder to track
- Runtime-constructed names may be missed
- Doesn't catch parameters used by AWS-managed services
- May need to search multiple repos/branches
- Obfuscated or compiled code is difficult

### Best For
- Application-centric organizations
- Microservices architectures
- Understanding parameter ownership
- Code refactoring projects

### Time Estimate
- **Per Repository:** 10-20 minutes
- **Multiple Repositories (10+):** 2-4 hours
- **Complex Codebases:** 4-8 hours

---

## Solution 6: Tag-Based Inventory

### Description
Leverage AWS resource tags on parameters to identify ownership, usage, and relationships.

### How It Helps
- Quick filtering and grouping of parameters
- Identifies ownership and accountability
- Enables bulk operations on tagged resources
- Supports cost allocation and chargeback
- Facilitates automated compliance checks

### Recommended Tag Schema

#### Essential Tags
- **Environment:** dev, staging, uat, prod
- **Application:** app-name or service-name
- **Owner:** team-name or email
- **ManagedBy:** terraform, cloudformation, manual
- **CostCenter:** for billing/chargeback

#### Operational Tags
- **LastReviewedDate:** YYYY-MM-DD
- **ScheduledForMigration:** true/false
- **DataClassification:** public, internal, confidential, restricted
- **Compliance:** HIPAA, PCI-DSS, SOC2

#### Migration-Specific Tags
- **MigrationStatus:** pending, in-progress, completed, failed
- **OriginalType:** String, SecureString
- **MigrationDate:** YYYY-MM-DD

### Implementation Approaches

#### Retroactive Tagging
- Use CloudTrail to identify who created each parameter
- Tag based on IAM role or user patterns
- Correlate with service usage from other solutions

#### Automated Tagging
- Lambda function triggered on parameter creation
- AWS Config rules to enforce tagging
- Tag policies in AWS Organizations

### Pros
- Fast filtering and reporting once tags are in place
- Enables automation and bulk operations
- Good for ongoing governance
- Supports cost allocation
- Easy to understand and maintain
- Built-in AWS Console support

### Cons
- Only works if tagging discipline exists
- Historical parameters likely untagged
- Doesn't show actual runtime usage
- Requires manual tagging effort initially
- Tags can become stale or inaccurate
- No enforcement without AWS Organizations policies

### Best For
- Organizations with existing tagging standards
- Ongoing governance and compliance
- Cost allocation and chargeback
- Environments with hundreds of parameters

### Time Estimate
- **Initial Tagging (50 parameters):** 2-3 hours
- **Creating Tagging Strategy:** 1-2 hours
- **Automation Setup:** 2-4 hours
- **Ongoing Maintenance:** 30 minutes per month

---

## Solution 7: Controlled Deprecation Testing (Canary Approach)

### Description
Temporarily modify parameter values to intentionally break dependencies, then monitor what fails to identify active usage.

### How It Works

#### Phase 1: Preparation
- Identify parameter to test
- Document current value
- Set up monitoring (CloudWatch alarms, log aggregation)
- Notify relevant teams (optional, depending on environment)
- Create rollback plan

#### Phase 2: Value Modification
- Change parameter value to obvious test string (e.g., "DEPRECATED-TEST-VALUE-DO-NOT-USE")
- Or append "-DEPRECATED" to current value
- Document change timestamp

#### Phase 3: Observation Period
- Monitor for 24-48 hours minimum
- Check CloudWatch logs for errors
- Review application metrics
- Monitor support tickets/Slack channels
- Catch periodic jobs (daily, weekly cron jobs)

#### Phase 4: Analysis
- Document which services failed
- Note error messages and timestamps
- Map parameters to services definitively

#### Phase 5: Restoration
- Restore original value
- Verify services recover
- Document findings

### Pros
- Most definitive test of actual usage
- Catches infrequent and scheduled jobs
- Reveals real-world dependencies
- Tests actual runtime behavior
- Uncovers hidden dependencies
- Creates accurate dependency map

### Cons
- ⚠️ **Risky even in UAT/non-production**
- May impact testing, QA, or demo activities
- Requires comprehensive monitoring
- Time-consuming (need observation period)
- Can cause alert fatigue
- Requires coordination with teams
- May miss very infrequent jobs (monthly, quarterly)

### Safety Requirements
- **ONLY for non-production environments**
- Must have rollback plan
- Need monitoring infrastructure in place
- Should test during off-hours
- Requires approval from stakeholders
- Must document everything

### Best For
- Critical parameters where certainty is required
- UAT environments with good monitoring
- Parameters suspected to be unused
- Final validation before production migration

### Time Estimate
- **Per Parameter Test:** 24-72 hours (mostly waiting)
- **Setup and Monitoring:** 1-2 hours
- **Analysis:** 30-60 minutes
- **Total for 10 parameters:** 1-2 weeks

---

## Solution 8: VPC Flow Logs + Endpoint Analysis

### Description
Monitor network traffic to AWS Systems Manager VPC endpoints to identify which resources are making SSM API calls.

### How It Works
- Enable VPC Flow Logs on VPC
- Configure SSM VPC endpoints (if not using public endpoints)
- Analyze flow logs to see which ENIs/instances communicate with SSM
- Correlate source IPs with EC2 instances, Lambda functions, or other resources

### Data Points Captured
- Source IP address (which instance/service)
- Destination (SSM endpoint)
- Traffic volume
- Timestamps
- Accept/reject status

### Analysis Methods
- **CloudWatch Logs Insights:** Query flow logs directly
- **Athena:** SQL queries on S3-stored flow logs
- **Third-party tools:** VPC Flow Logs analyzers

### Pros
- Network-level visibility independent of IAM/CloudTrail
- Can identify unexpected access patterns
- Works even if CloudTrail is disabled
- Shows actual network communication
- Helps with security posture assessment
- Can identify traffic from unexpected sources

### Cons
- Doesn't show which specific parameters accessed
- High volume of logs to analyze
- Requires VPC endpoints to be configured
- Complex correlation with actual resources
- Doesn't work for services using public endpoints
- Additional costs for flow log storage
- NAT-ed traffic harder to trace

### Best For
- Security-focused audits
- Identifying unauthorized access attempts
- Environments with VPC endpoints already configured
- Supplement to other methods

### Time Estimate
- **VPC Flow Logs Setup:** 30-60 minutes
- **Data Collection Period:** 7-14 days
- **Analysis:** 2-4 hours
- **Correlation with Resources:** 2-3 hours

---

## Solution 9: AWS Config Resource Inventory

### Description
Use AWS Config to capture and query current state of all resources and their configurations, including relationships.

### How It Works
- AWS Config continuously records resource configurations
- Can query configurations using SQL-like syntax
- Shows point-in-time snapshots of resource states
- Tracks configuration changes over time

### Queryable Information
- Lambda function environment variables
- ECS task definition configurations
- EC2 instance user data
- IAM role policies and attachments
- Resource tags and metadata
- Configuration compliance status

### Query Capabilities
- **Advanced Queries:** SQL-like queries across resource types
- **Aggregators:** Cross-account and cross-region queries
- **Configuration History:** Track changes over time
- **Relationship Mapping:** See which resources reference others

### Sample Use Cases
- Find all Lambda functions with environment variables containing "SSM" or parameter paths
- Identify all ECS task definitions with secrets configurations
- List all EC2 instances with specific IAM instance profiles

### Pros
- Point-in-time configuration snapshots
- Can query relationships between resources
- Good for compliance documentation
- Shows configuration drift
- Historical tracking of changes
- Works across accounts with aggregators
- Supports custom compliance rules

### Cons
- Requires AWS Config to be enabled (costs money)
- Shows configuration, not actual runtime behavior
- Complex query syntax requires learning
- Historical data increases costs
- Doesn't show parameter values, only references
- May have lag in configuration updates (minutes)

### Best For
- Organizations already using AWS Config
- Compliance and audit requirements
- Large environments needing automation
- Cross-account visibility needs

### Time Estimate
- **AWS Config Setup:** 1-2 hours
- **Query Development:** 1-2 hours
- **Data Analysis:** 2-3 hours
- **Ongoing Queries:** 15-30 minutes per run

### Cost Considerations
- Configuration items recorded: ~$0.003 per item
- Configuration item delivered: ~$0.001 per delivery
- Can add up quickly in large environments

---

## Solution 10: CloudWatch Logs Insights

### Description
Query application logs across all CloudWatch log groups to find parameter references and SSM API calls.

### How It Works
- Applications write logs to CloudWatch Logs
- Use CloudWatch Logs Insights to run queries across log groups
- Search for patterns indicating parameter access
- Analyze results to identify parameter usage

### Query Patterns

#### Generic SSM Access
```
fields @timestamp, @message
| filter @message like /GetParameter|ssm|parameter/
| sort @timestamp desc
```

#### Specific Parameter Names
```
fields @timestamp, @message
| filter @message like /\/myapp\/db\/password/
| stats count() by bin(5m)
```

#### Error Patterns
```
fields @timestamp, @message
| filter @message like /SSM.*error|parameter.*failed/
| sort @timestamp desc
```

### Information Gathered
- When parameters were accessed
- Which functions/applications accessed them
- Error patterns and failures
- Usage frequency and patterns
- Request/response logging (if enabled)

### Pros
- Searches actual application runtime behavior
- Works if applications log SSM operations
- Can search across multiple log groups simultaneously
- Shows real usage patterns with timestamps
- Can correlate with application errors
- Fast query execution

### Cons
- Only works if applications log SSM calls
- Most applications don't log this by default
- Log retention limits (typically 1-90 days)
- Costs increase with data scanned
- May miss silent failures
- Requires consistent logging practices
- Query syntax learning curve

### Best For
- Applications with comprehensive logging
- Troubleshooting specific parameters
- Short-term usage analysis
- Correlating parameter access with application behavior

### Time Estimate
- **Query Development:** 30-60 minutes
- **Per Query Execution:** 2-5 minutes
- **Analysis of Results:** 30-60 minutes
- **Multiple Log Groups:** 1-2 hours

### Cost Considerations
- Queries charged per GB of data scanned
- Typical cost: $0.005 per GB scanned
- Can add up with large log volumes

---

## Recommended Hybrid Approach

For the fastest and most comprehensive results, combine multiple solutions strategically.

### Recommended Combination

#### Step 1: Quick Wins (Day 1 - 2 hours)
**Use Solution 1: CloudTrail Analysis**
- Identify parameters accessed in last 90 days
- Get list of active users/roles
- Quick baseline understanding

**Output:** List of actively-used parameters and their consumers

#### Step 2: Deep Dive (Day 1-2 - 4 hours)
**Use Solution 3: Service-by-Service Review**
- Focus on Lambda functions (environment variables)
- Check ECS task definitions (secrets section)
- Review CloudFormation stacks ({{resolve:ssm:}})

**Output:** Static configuration references

#### Step 3: Code Analysis (Day 2-3 - 2 hours)
**Use Solution 4: IaC Repository Search**
- Search Terraform/CloudFormation repos
- Find all parameter references in code
- Identify parameters in feature branches

**Output:** Complete IaC parameter inventory

#### Step 4: Documentation & Tagging (Day 3-4 - 2 hours)
**Use Solution 6: Tag-Based Inventory**
- Tag all identified parameters with owners
- Add migration status tags
- Document findings

**Output:** Tagged, organized parameter inventory

### Total Time Investment
- **Small Environment:** 4-6 hours
- **Medium Environment:** 8-12 hours
- **Large Environment:** 16-24 hours

### Expected Coverage
- **Active Parameters:** 95-100%
- **Dormant Parameters:** 70-85%
- **Planned/Future Parameters:** 60-80%

### What Gets Missed
- Parameters accessed only before 90-day window
- Very infrequent scheduled jobs (quarterly, annual)
- Parameters in code not yet deployed
- Manually created resources without documentation

### Validation Step
After completing the hybrid approach, consider using **Solution 7: Controlled Deprecation Testing** on high-value or uncertain parameters to validate findings.

---

## Migration Strategy

Once dependencies are identified, follow this structured migration approach.

### Pre-Migration Checklist

- [ ] Complete dependency analysis
- [ ] Create KMS key for encryption
- [ ] Document all parameters and their consumers
- [ ] Identify parameter owners/stakeholders
- [ ] Set up monitoring and alerting
- [ ] Create rollback procedures
- [ ] Schedule migration windows
- [ ] Communicate with affected teams

### KMS Key Setup

#### Create Customer-Managed KMS Key
1. Navigate to AWS KMS console
2. Create new symmetric key
3. Define key policy granting decrypt permissions to:
   - Lambda execution roles
   - ECS task roles
   - EC2 instance profiles
   - Any other consumers identified
4. Create alias: `alias/parameter-store-key`
5. Enable automatic rotation (optional, recommended)

#### Key Policy Considerations
- Grant `kms:Decrypt` to all parameter consumers
- Grant `kms:Encrypt` to parameter creators/updaters
- Grant `kms:DescribeKey` for validation
- Consider separate keys for different environments/applications

### IAM Policy Updates

**CRITICAL: Update IAM policies BEFORE migrating parameters**

#### For Each Consumer Role/User
Add KMS decrypt permission to their IAM policy

#### Validate Policy Updates
- Test in development first
- Verify policies propagate (can take 1-2 minutes)
- Check IAM policy simulator

### Migration Phases

#### Phase 1: Preparation (Week 1)
- [ ] Backup all parameter values
- [ ] Create migration tracking spreadsheet
- [ ] Set up KMS key
- [ ] Update all IAM policies
- [ ] Test KMS access in development
- [ ] Create parameter migration script template
- [ ] Set up monitoring dashboards

#### Phase 2: Development/Test Environment (Week 2)
- [ ] Migrate development parameters first
- [ ] Test all applications
- [ ] Validate KMS decryption works
- [ ] Adjust any issues
- [ ] Document lessons learned

#### Phase 3: UAT Environment (Week 3-4)
- [ ] Create migration plan with prioritization
- [ ] Migrate low-risk parameters first
- [ ] Monitor for 24-48 hours between batches
- [ ] Validate application functionality
- [ ] Adjust approach based on issues

#### Phase 4: Production (Week 5-6)
- [ ] Schedule during maintenance window
- [ ] Use blue/green deployment if possible
- [ ] Migrate in small batches
- [ ] Monitor each batch for 48 hours
- [ ] Keep old parameters for 30 days as backup

#### Phase 5: Cleanup (Week 7)
- [ ] Verify all services working correctly
- [ ] Remove backup plain-text parameters
- [ ] Update documentation
- [ ] Final security scan
- [ ] Retrospective meeting

### Migration Methods

#### Method 1: In-Place Overwrite (Recommended)
**Process:**
1. Get current parameter value
2. Create new SecureString parameter with same name (overwrite)
3. Applications automatically use new encrypted version
4. No code changes needed (if IAM updated)

**Advantages:**
- No code changes
- Minimal downtime
- Simple rollback (overwrite again)

**Disadvantages:**
- Loses parameter history
- One-shot migration

#### Method 2: Parallel Parameters
**Process:**
1. Create new SecureString parameter with different name (`-secure` suffix)
2. Update application code to use new name
3. Deploy application changes
4. Verify new parameter works
5. Delete old parameter

**Advantages:**
- Can test before cutover
- Preserves history
- Easy rollback (revert code)

**Disadvantages:**
- Requires code changes
- More complex coordination
- Temporary duplication

#### Method 3: Blue/Green Deployment
**Process:**
1. Create new parameters in "green" environment
2. Deploy applications to green environment
3. Test thoroughly
4. Switch traffic to green
5. Decommission blue

**Advantages:**
- Zero-downtime migration
- Full testing before cutover
- Easy rollback

**Disadvantages:**
- Requires infrastructure duplication
- More complex setup
- Higher temporary costs

### Rollback Procedures

#### If Migration Fails

**Immediate Rollback:**
1. Restore original String parameter value
2. Check application recovery
3. Review error logs
4. Identify root cause

**Common Failure Scenarios:**
- Missing KMS decrypt permission → Add IAM policy
- Wrong KMS key used → Update parameter with correct key
- Application not configured for decryption → Update app code
- Network connectivity issues → Check VPC endpoints

### Post-Migration Validation

#### Application Testing
- [ ] Run smoke tests on all applications
- [ ] Verify parameter retrieval works
- [ ] Check application logs for errors
- [ ] Monitor performance metrics
- [ ] Test edge cases and error handling

#### Security Validation
- [ ] Verify parameters encrypted at rest
- [ ] Check CloudTrail for decrypt operations
- [ ] Validate IAM policies are least-privilege
- [ ] Run security scanning tools
- [ ] Audit access logs

#### Documentation Updates
- [ ] Update runbooks and procedures
- [ ] Update architecture diagrams
- [ ] Document new parameter naming conventions
- [ ] Create troubleshooting guides
- [ ] Update team knowledge base

---

## Decision Matrix

Use this matrix to select the best solutions for your specific situation.

### By Environment Size

| Environment Size | Recommended Solutions | Time Investment | Coverage |
|-----------------|----------------------|-----------------|----------|
| **Small** (< 20 parameters, < 10 services) | 1 + 3 | 4-6 hours | 90%+ |
| **Medium** (20-100 parameters, 10-50 services) | 1 + 3 + 4 | 8-12 hours | 85-90% |
| **Large** (100+ parameters, 50+ services) | 1 + 2 + 4 + 9 | 16-24 hours | 80-85% |
| **Enterprise** (500+ parameters, 100+ services) | All + Automation | 40+ hours | 75-80% |

### By Primary Use Case

| Use Case | Recommended Solutions | Priority Order |
|----------|----------------------|----------------|
| **Quick audit before migration** | 1, 3, 4 | High-speed, good coverage |
| **Compliance documentation** | 2, 6, 9 | Formal, auditable |
| **Security hardening** | 1, 2, 8 | Permission-focused |
| **Cost optimization** | 1, 6, 7 | Identify unused |
| **Application modernization** | 4, 5, 10 | Code-centric |

### By Resource Availability

| Available Resources | Recommended Approach | Rationale |
|--------------------|---------------------|-----------|
| **1 person, 1 day** | Solutions 1 + 3 (Lambda/ECS only) | Quick wins |
| **1 person, 1 week** | Solutions 1 + 3 + 4 | Comprehensive |
| **Team, 2 weeks** | Solutions 1-6 | Very thorough |
| **Ongoing program** | All solutions + automation | Complete coverage |

### By Technical Maturity

| Maturity Level | Recommended Solutions | Notes |
|----------------|----------------------|-------|
| **Low** (mostly ClickOps) | 1, 3, 10 | Manual, service-focused |
| **Medium** (some IaC) | 1, 3, 4, 6 | Balanced approach |
| **High** (full IaC, CI/CD) | 1, 4, 5, 9 | Code and automation |
| **Very High** (platform engineering) | Automated 1, 2, 4, 5, 9 | Fully programmatic |

---

## Common Pitfalls and Solutions

### Pitfall 1: Missing KMS Permissions
**Symptom:** Applications fail after migration with "Access Denied"  
**Solution:** Update IAM policies with `kms:Decrypt` BEFORE migration  
**Prevention:** Test in dev first, validate policies

### Pitfall 2: Overlooking Scheduled Jobs
**Symptom:** Daily/weekly jobs fail after migration  
**Solution:** Run monitoring for full cycle period (7+ days)  
**Prevention:** Document all cron jobs and scheduled tasks

### Pitfall 3: Dynamic Parameter Names
**Symptom:** Code-generated parameter names not identified  
**Solution:** Deep code analysis, review application logic  
**Prevention:** Code search for parameter construction patterns

### Pitfall 4: Cross-Account Access
**Symptom:** Parameters accessed from other AWS accounts fail  
**Solution:** Update KMS key policy for cross-account access  
**Prevention:** Document all cross-account integrations

### Pitfall 5: Inadequate Rollback Testing
**Symptom:** Can't quickly recover from failed migration  
**Solution:** Test rollback procedures before migration  
**Prevention:** Document and practice rollback process

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-21 | Claude AI | Initial document creation |

---

## Next Steps

1. **Review this guide** with your security and operations teams
2. **Select appropriate solutions** based on decision matrix
3. **Obtain necessary AWS permissions** for audit activities
4. **Schedule audit window** with minimal business impact
5. **Execute selected solutions** sequentially or in parallel
6. **Document all findings** in centralized tracker
7. **Create detailed migration plan** based on findings
8. **Execute migration** with proper testing and rollback procedures
9. **Validate success** against defined metrics
10. **Update documentation** and processes for ongoing governance

---

**Remember:** This is a security improvement initiative. Take the time to do it right, test thoroughly, and maintain good documentation throughout the process.
