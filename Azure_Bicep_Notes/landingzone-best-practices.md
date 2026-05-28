<h1 align="center">🚀 Best Practices Landing Zone Setup Using BICEP</h1>

<p align="center">
<img src="https://skillicons.dev/icons?i=azure,docker,terraform,github,vscode" />
</p>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=24&duration=3000&pause=1000&color=0078D4&center=true&width=700&lines=Azure+BICEP+Templates;Landing+Zone+Architecture;Secure+%26+Scalable+Infrastructure" />
</p>

# Best Practices Landing Zone Setup Using BICEP


Today I'm going to walk you through how we can deploy a **complete Azure Landing Zone** using **Bicep** — Microsoft's native Infrastructure-as-Code language.

By the end of this notes, you will clearly understand:

- What an Azure Landing Zone actually is
- Why Bicep is the right tool for this job
- The **best practices** we must follow to build it correctly
- And the key areas — networking, security, monitoring, IAM — that we need to cover

So let's get started."


---

## 🟦 SECTION 1 — What is an Azure Landing Zone?

---

"Before we jump into code, let me quickly explain — **what is a Landing Zone?**

Think of it like this — before a plane lands, the airport needs to be fully ready. Runways, air traffic control, fuel, security — everything in place.

Similarly, an **Azure Landing Zone** is a pre-configured, well-governed Azure environment that is **ready to host your workloads** — whether that's virtual machines, containers, databases, or microservices.

It covers five key pillars:

First — **Identity and Access Management.** Who can access what, and with what level of permission.

Second — **Networking.** How resources communicate with each other and with the outside world — securely.

Third — **Security and Compliance.** Policies, Defender, Key Vault — making sure nothing is exposed.

Fourth — **Monitoring and Observability.** Logs, metrics, alerts — so we always know what's happening.

And fifth — **Governance.** Naming conventions, tagging, resource locks — so the environment stays clean and manageable over time.

A Landing Zone is NOT just one resource group. It is an **entire foundation** — spanning management groups, subscriptions, resource groups, and resources."


---

## 🟦 SECTION 2 — Why Bicep?

---

"Now — why are we using **Bicep** for this?

We have several options — ARM templates, Terraform, Pulumi. So why Bicep?

Bicep is Microsoft's **first-class, native IaC language** for Azure. Here's what makes it the right choice:

**Number one — it's simpler than ARM templates.** ARM JSON is verbose and hard to read. Bicep is clean, concise, and much easier to write and maintain.

**Number two — it has full Azure API coverage.** The moment Azure releases a new resource type, Bicep supports it. You don't have to wait for a third-party provider to be updated.

**Number three — modules.** Bicep has a powerful module system, which means we can break our infrastructure into small, reusable, independently testable pieces.

**Number four — tight CI/CD integration.** Bicep works natively with Azure DevOps and GitHub Actions, with built-in linting, what-if analysis, and deployment commands.

And **number five — it compiles to ARM templates.** So under the hood, Azure still uses ARM, but we write in a much friendlier syntax.

Think of Bicep as the developer-friendly face of Azure's native IaC."


---

## 🟦 SECTION 3 — Folder Structure and Modularity

---

"Alright — let's now talk about **how we organize our Bicep code.**

The single most important principle is: **modularity.**

One module. One responsibility. Never put everything in one giant file.

Here is the folder structure we follow:

```
infra/
├── main.bicep                  ← Entry point — orchestrates everything
├── main.bicepparam             ← Environment-specific parameters
├── modules/
│   ├── networking/             ← VNet, NSG, Route Tables
│   ├── security/               ← Key Vault, Defender
│   ├── compute/                ← VMs, AKS clusters
│   ├── storage/                ← Storage Accounts
│   ├── monitoring/             ← Log Analytics, Diagnostics
│   └── iam/                    ← Role Assignments
└── shared/
    └── naming.bicep            ← Centralized naming conventions
```

Your `main.bicep` is the **conductor**. It calls all the modules in the right order, passes parameters, and wires everything together.

Each module inside the `modules/` folder handles exactly one thing.

This approach gives us three huge benefits:

**Reusability** — the same networking module can be reused across dev, staging, and production.

**Testability** — we can test each module independently before deploying the full stack.

**Maintainability** — when something changes, we only touch one file, not the entire codebase."


---

## 🟦 SECTION 4 — Naming Conventions

---

"Next — **naming conventions.** This sounds simple, but bad naming is one of the biggest causes of confusion in large Azure environments.

We follow Microsoft's **Cloud Adoption Framework** naming standards. Here are some examples:

- Resource Group → `rg-`
- Virtual Network → `vnet-`
- Subnet → `snet-`
- Network Security Group → `nsg-`
- Key Vault → `kv-`
- Log Analytics Workspace → `law-`
- Public IP → `pip-`

And we always include the environment, location, and application name:

So a production VNet in East US 2 for our payments app would be named: `vnet-prod-eus2-payments`

We create a **shared naming module** in Bicep that generates all these names consistently. Every other module imports from this shared naming module.

This means — if tomorrow we decide to change the naming pattern — we change it in **one place**, and it propagates everywhere automatically.

Consistency in naming saves hours of debugging and makes your Azure portal much easier to navigate."


---

## 🟦 SECTION 5 — Networking (Hub-Spoke Architecture)

---

"Now we come to the **backbone of the Landing Zone — Networking.**

The architecture we follow is called **Hub and Spoke.**

Think of a bicycle wheel. The **hub** is in the center, and multiple **spokes** extend outward.

In Azure terms:

The **Hub VNet** is the central, shared network. It contains:
- **Azure Firewall** — all traffic flows through here for inspection
- **Azure Bastion** — for secure, browser-based access to VMs without exposing RDP or SSH
- **VPN or ExpressRoute Gateway** — for connectivity back to on-premises
- **Management Subnet** — for administrative resources

Each **Spoke VNet** represents a workload — maybe one spoke for your e-commerce application, another for your data platform, another for DevOps tooling.

Spokes are connected to the Hub using **VNet Peering.**

Here are the key rules for networking in Bicep:

**Rule one — every subnet must have an NSG.** Network Security Groups are your traffic filters. Apply them everywhere, and follow the principle of least privilege.

**Rule two — use Route Tables (UDR) to force traffic through Azure Firewall.** Don't let traffic go directly between spokes — inspect it at the firewall first.

**Rule three — use Private Endpoints for all PaaS services.** Key Vault, Storage Accounts, SQL Databases, Service Bus — none of these should be accessible over the public internet. Private Endpoints give each service a private IP address inside your VNet.

This network design ensures **complete traffic control, zero implicit trust, and full auditability.**"


---

## 🟦 SECTION 6 — Security Best Practices

---

"Security is not something we add at the end — it must be **baked in from day one.**

Here are the key security practices we implement in our Bicep templates:

**Key Vault configuration:**
We always enable RBAC authorization — not Access Policies. We enable soft delete with a 90-day retention period. We enable purge protection. And we set public network access to disabled, with private endpoints only.

```bicep
enableRbacAuthorization: true
enableSoftDelete: true
softDeleteRetentionInDays: 90
enablePurgeProtection: true
publicNetworkAccess: 'Disabled'
```

**Storage Accounts:**
Public blob access is always disabled. HTTPS-only is enforced. Minimum TLS version is set to TLS 1.2.

**Microsoft Defender for Cloud:**
We enable Defender for all resource types — servers, storage, SQL, Key Vault, containers, App Service. This gives us continuous security assessment, threat detection, and compliance scoring.

**Managed Identities over Service Principals:**
Whenever a resource needs to authenticate to another Azure resource, we use Managed Identity. No passwords, no certificates, no client secrets to rotate. Azure handles all of that automatically.

**Azure Policy:**
We assign policies at the subscription level to enforce our security standards. For example — a policy that prevents creation of any storage account with public access, or a policy that requires all resources to have specific tags.

The golden rule of security is — **deny by default, allow explicitly.**"


---

## 🟦 SECTION 7 — Monitoring and Observability

---

"You cannot manage what you cannot see. That's why **monitoring** is a non-negotiable part of the Landing Zone.

Our monitoring stack has three components:

**Component one — Log Analytics Workspace.**
This is the central repository for all logs across the entire landing zone. Every resource ships its logs here.

**Component two — Diagnostic Settings.**
For every single resource we deploy — VMs, Key Vault, Storage, VNet — we attach a Diagnostic Settings configuration that sends logs and metrics to Log Analytics.

We build this as a **reusable Bicep module** that we call from every resource deployment:

```bicep
resource diagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'diag-${resourceName}'
  scope: targetResource
  properties: {
    workspaceId: logAnalyticsWorkspaceId
    // ... logs and metrics config
  }
}
```

**Component three — Azure Monitor Alerts.**
We configure alerts for critical conditions — high CPU, low memory, failed authentication attempts, Key Vault access denied events, and application errors.

The goal is simple: **we should know about a problem before our users do.**

Additionally, we export the Azure Activity Log to Log Analytics — this gives us a complete audit trail of every action taken in the subscription — who deployed what, when, and from where."


---

## 🟦 SECTION 8 — IAM and RBAC

---

"Identity is the new perimeter. In a cloud environment, **who has access to what** is just as important as network firewalls.

Our IAM strategy is built on three principles:

**Principle one — Least Privilege.**
Give every identity only the minimum permissions it needs to do its job. An application that only reads from a storage account should have the Storage Blob Data Reader role — not Contributor, not Owner.

**Principle two — No standing access for humans.**
For administrative tasks, we use **Privileged Identity Management (PIM)**. Admins request elevated access when they need it, it's granted for a limited time — say, 4 hours — and it's logged. No permanent Owner assignments.

**Principle three — Managed Identities for everything non-human.**
Applications, pipelines, automation scripts — all use Managed Identities. Zero credentials to manage, zero credentials to leak.

In Bicep, all role assignments use the `guid()` function to generate deterministic, idempotent assignment IDs:

```bicep
name: guid(resourceGroup().id, principalId, roleDefinitionId)
```

This ensures that re-running the deployment doesn't create duplicate role assignments.

We also apply **Resource Locks** to production resources — specifically the `CanNotDelete` lock — so that no one can accidentally delete a critical resource."


---

## 🟦 SECTION 9 — Parameters and Secrets Management

---

"One of the most common mistakes in IaC is **hardcoding values** — especially secrets.

Here is the rule: **no secret, password, connection string, or API key should ever appear in a Bicep file or a parameters file.**

In Bicep, we use `.bicepparam` files for environment-specific values:

```bicep
param environment = 'prod'
param location = 'eastus2'
param skuName = 'Standard'
```

For secrets, we use **Key Vault references** directly in the parameter file:

```bicep
param sqlPassword = getSecret(
  subscriptionId, 'rg-shared', 'kv-prod', 'sql-password'
)
```

This means the secret is pulled from Key Vault at deployment time — it never touches our code repository.

We also maintain **separate parameter files per environment**:
- `main.dev.bicepparam`
- `main.staging.bicepparam`
- `main.prod.bicepparam`

The same Bicep templates deploy to all environments — only the parameters change. This is the foundation of a **consistent, repeatable deployment process.**"


---

## 🟦 SECTION 10 — Tagging Strategy

---

"Tags are the **metadata layer** of your Azure environment. Without them, you lose visibility into cost, ownership, and compliance.

We enforce a minimum set of tags on every single resource:

| Tag | Value Example | Purpose |
|-----|--------------|---------|
| `Environment` | prod / dev / staging | Environment identification |
| `Application` | payments-service | Which app owns this resource |
| `CostCenter` | CC-1234 | Finance chargeback |
| `Owner` | team@company.com | Who is responsible |
| `ManagedBy` | Bicep | How it was deployed |
| `DeployedDate` | 2025-01-15 | When it was created |

In Bicep, we define a `commonTags` variable once and pass it to every resource:

```bicep
var commonTags = {
  Environment:  environmentName
  Application:  applicationName
  CostCenter:   costCenter
  Owner:        ownerEmail
  ManagedBy:    'Bicep'
}
```

We also use **Azure Policy** to enforce tagging — if a resource is deployed without required tags, the policy automatically denies the deployment."


---

## 🟦 SECTION 11 — Deployment Order and Scopes

---

"In Bicep, different resources are deployed at different **scopes** — Management Group, Subscription, or Resource Group level.

It's critical to deploy in the correct order:

**Step 1 — Management Group level:**
Management Group structure, Policy definitions, top-level RBAC assignments.

**Step 2 — Subscription level:**
Resource Groups creation, Subscription-level Policy assignments, Defender enablement.

**Step 3 — Resource Group level — Networking first:**
Hub VNet, Azure Firewall, Bastion, VPN Gateway. Then Spoke VNets and Peering.

**Step 4 — Security layer:**
Key Vault, Managed Identities, Private Endpoints.

**Step 5 — Monitoring layer:**
Log Analytics Workspace, Diagnostic Settings, Monitor Alerts.

**Step 6 — Application resources:**
Virtual Machines, AKS, App Services, Databases — whatever your workloads require.

The reason for this order is **dependency management.** Your application VMs need subnets. Subnets need NSGs. NSGs need to exist before the VNet. And so on.

Bicep handles explicit dependencies with the `dependsOn` keyword, but most of the time, using **resource references** automatically creates the correct dependency order."


---

## 🟦 SECTION 12 — CI/CD Pipeline Integration

---

"The final piece — and arguably the most important for a production Landing Zone — is the **deployment pipeline.**

Infrastructure should never be deployed manually from someone's laptop. Every change goes through a pipeline.

Our pipeline has three stages:

**Stage one — Lint:**
We run `az bicep lint` to catch syntax errors and code quality issues before anything is deployed.

**Stage two — What-If:**
We run `az deployment sub what-if` which gives us a **dry run preview** — showing exactly what resources will be created, modified, or deleted — without actually making any changes. This is your safety net.

**Stage three — Deploy:**
Only after the What-If output is reviewed and approved do we run the actual deployment.

```yaml
- name: Lint
  run: az bicep lint --file main.bicep

- name: What-If Preview
  run: az deployment sub what-if \
         --template-file main.bicep \
         --parameters main.prod.bicepparam

- name: Deploy
  run: az deployment sub create \
         --template-file main.bicep \
         --parameters main.prod.bicepparam
```

For production environments, we also add a **manual approval gate** between What-If and Deploy. This ensures a human reviews the changes before they go live.

We store all Bicep files in Git, with branch protection rules — no direct commits to main, everything goes through a pull request."


---

## 🟦 CLOSING — Summary and Key Takeaways

---

"Let me quickly summarize everything we've covered today.

Building a proper Azure Landing Zone with Bicep is about **discipline and structure.**

Here are the key takeaways:

**One — Modular code.** One module, one responsibility. Small, reusable, testable pieces.

**Two — CAF naming.** Consistent naming using Microsoft's Cloud Adoption Framework standards.

**Three — Hub-Spoke networking.** All traffic through Azure Firewall, Private Endpoints for all PaaS, NSG on every subnet.

**Four — Security by default.** Defender enabled everywhere, Key Vault for all secrets, Managed Identities, no public access.

**Five — Monitoring on everything.** Log Analytics Workspace as the central hub, Diagnostic Settings on every resource.

**Six — Least privilege IAM.** PIM for humans, Managed Identities for applications, resource locks on production.

**Seven — No hardcoded secrets.** Key Vault references in parameter files, environment-specific param files.

**Eight — Tags on everything.** Environment, Owner, CostCenter — enforced via Azure Policy.

**Nine — Deploy in the right order.** Management Groups → Subscriptions → Networking → Security → Monitoring → Applications.

**Ten — Everything through a pipeline.** Lint, What-If, manual approval, then Deploy. No manual deployments.

If you follow these principles, you will have a Landing Zone that is **secure, observable, governable, and maintainable** — not just for today, but as your organization scales.

A great reference to start from is the **Azure Landing Zones Bicep** repository on GitHub — `Azure/ALZ-Bicep`. It implements all of these patterns and is maintained by Microsoft's CAF team directly.

Thank you. I'm happy to take any questions."

---

## 📋 Quick Reference Cheat Sheet

| Area | Key Practice |
|------|-------------|
| **Structure** | Modular Bicep — one file per resource type |
| **Naming** | CAF convention — `rg-`, `vnet-`, `snet-`, `kv-` |
| **Networking** | Hub-Spoke, Firewall, Bastion, Private Endpoints |
| **Security** | Defender, RBAC, No public access, Managed Identity |
| **Monitoring** | Log Analytics + Diagnostics on every resource |
| **IAM** | Least Privilege, PIM, Resource Locks |
| **Secrets** | Key Vault references — nothing hardcoded |
| **Tags** | Env, Owner, CostCenter on every resource |
| **Order** | Mgmt Group → Subscription → Network → Security → Monitoring → App |
| **Pipeline** | Lint → What-If → Approve → Deploy |

---

