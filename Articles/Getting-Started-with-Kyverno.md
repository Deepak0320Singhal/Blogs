![My image](https://miro.medium.com/v2/format:webp/1*JkrotaYDUU4O0wtumTjTxw.avif)
# Getting Started with Kyverno: Your First Policy in Action

> A hands-on tutorial demonstrating how to deploy your first Kubernetes policy — written from a learner's perspective for the Kyverno community

## About This Tutorial

**Target Audience:** Kubernetes users new to policy management and governance

**Prerequisites:**
- Basic kubectl knowledge
- A running Kubernetes cluster (Minikube, Kind, Docker Desktop, or cloud-based)
- Familiarity with YAML syntax

**What You'll Learn:**
- What Kyverno does and why it matters
- How to install Kyverno in your cluster
- Writing and deploying your first validation policy
- Testing policies with real examples

## What Problem Does Kyverno Actually Solve?

### The Real-World Scenario

Imagine this: Your teammate is deploying a new microservice to your Kubernetes cluster. They're under pressure to ship a feature by end-of-day. In their rush, they forget to set resource limits on the container.

That container gets deployed. Everything seems fine initially. Then, due to a memory leak or unexpected load, the container starts consuming all available memory on the node. Other applications get starved of resources. Pods start getting evicted. Your monitoring system explodes with alerts. It's 2 AM, and you're in an incident channel trying to figure out what happened.

Or this: Someone pulls a container image from an untrusted registry to test something quickly. That image contains a vulnerability — or worse, malware. Your security scan catches it three days later. Now you have to track down what was deployed, assess the damage, and figure out how it got through.

Or this: Your team agreed on naming conventions and labeling standards for resources. Half the developers follow them religiously. The other half forget or don't know about them. Your monitoring dashboards show inconsistent data. Your billing reports are impossible to parse by team. Nobody knows which team owns which resources.

### The Root Problem

Rules and standards exist — in Confluence pages, in Slack messages, in team meetings, in README files. But they're not enforced automatically. Developers have to remember them. And humans forget, especially under pressure or when moving fast.

Manual code reviews catch some issues, but:
- They're slow and don't scale
- Reviewers can miss things
- They happen before merge, not at deployment time
- They rely on human vigilance

### Without Kyverno
- Rules live in documentation that people may or may not read
- Standards are enforced through manual review (slow, error-prone)
- Problems surface in production, not during deployment
- Security teams have to manually check every resource
- Inconsistent configurations across teams and environments
- You need to learn complex policy languages like Rego or write custom admission webhooks

### With Kyverno
- Rules are enforced automatically before resources enter the cluster
- Bad configurations are blocked or auto-fixed immediately
- Everyone follows the same standards — no exceptions, no memory required
- Policies are written in YAML (the same format you already use for Kubernetes)
- Security and compliance become automatic, not manual
- Developers get instant, actionable feedback on what needs fixing

**Bottom line:** Kyverno transforms cluster governance from "please remember to do this" to "the system won't let you do it wrong." It's like moving from gentlemen's agreements to actual guardrails.

## What Actually IS Kyverno?

**Simple definition:** Kyverno is a policy engine for Kubernetes that validates, mutates, and generates resources based on rules you define — ensuring every resource in your cluster follows your standards.

### The Analogy That Helped Me Understand

Think of Kyverno as a security guard at a club with a detailed rulebook:
- Someone tries to enter the club (a developer deploys a resource)
- The security guard checks their credentials against the rulebook (Kyverno validates the resource against your policies)
- If they meet all requirements, they get in (resource is created in the cluster)
- If not, they're turned away with a clear explanation of what's wrong (resource is rejected with an error message)
- Sometimes the guard can fix minor issues on the spot, like adjusting a dress code violation (mutation)

The guard doesn't make arbitrary decisions — they follow the rulebook consistently. You write the rulebook (your policies), and Kyverno enforces it automatically for every resource, every time.

**Etymology:** "Kyverno" comes from the Greek word "kyverno", meaning "to govern" or "to steer a ship." The name tells you exactly what it does — it governs your Kubernetes resources and helps you steer your cluster toward compliance and security.

### What Makes Kyverno Different from Other Policy Engines

**1. Kubernetes-Native Design**  
Kyverno was built specifically for Kubernetes from the ground up. Policies are Kubernetes Custom Resources, just like Deployments or Services. They're managed with kubectl, stored in Git, and follow the same patterns you already know.

**2. YAML-Based Policies**  
No new language to learn. If you can write a Pod spec, you can write a Kyverno policy. Compare this to alternatives like OPA/Gatekeeper (which use Rego) or Kyverno alternatives that require learning CEL or other domain-specific languages.

**3. Multiple Enforcement Modes**  
Kyverno can:
- **Validate:** Check if resources meet requirements (block if they don't)
- **Mutate:** Automatically fix or enhance resources (add missing labels, inject sidecars)
- **Generate:** Automatically create related resources (create default NetworkPolicies for new namespaces)

**4. Gradual Adoption**  
You can start in "Audit" mode (just report violations, don't block) and move to "Enforce" mode when you're confident. This makes it safe to experiment and learn.

### What Kyverno is NOT

- **Not a monitoring/observability tool:** It doesn't watch metrics, logs, or runtime behavior
- **Not a CI/CD pipeline tool:** It operates inside Kubernetes, not in your build/deploy pipeline
- **Not a replacement for RBAC:** It complements access controls, doesn't replace them
- **Not only for security experts:** While it helps with security, it's equally useful for operational standards and best practices

## Do I Need to Be a Kubernetes Expert?

**Short answer:** No.

When I started learning Kyverno, I was worried I didn't know enough about Kubernetes internals. Turns out, that worry was unfounded.

### What You DO Need to Know
- Basic Kubernetes concepts: What pods, deployments, services, and namespaces are
- kubectl basics: How to apply YAML files and check resource status
- YAML syntax: How to read and write YAML (indentation, lists, key-value pairs)
- Your use case: What rules or standards you want to enforce

### What You DON'T Need to Know
- Deep Kubernetes architecture (API server internals, etcd, controllers)
- How admission webhooks work under the hood
- Security frameworks and compliance standards (NIST, CIS, etc.)
- Other policy languages (Rego, CEL)
- Go programming or custom webhook development
- Advanced networking or storage concepts

**My promise to you:** You won't break anything. Kyverno policies only affect NEW resources you try to create or update. Everything already running in your cluster stays completely untouched. If you make a mistake in a policy, you just delete it and everything returns to normal. There's no risk of accidentally taking down your cluster.

Ready? Let's get hands-on.

## What We're Building Today

Let's set crystal-clear expectations for this tutorial.

In the next 15–20 minutes, you will:
1. Install Kyverno in your cluster with one command
2. Verify the installation is working correctly
3. Create ONE simple policy that enforces ONE rule
4. Test that policy by attempting to create non-compliant resources (and see them get blocked)
5. Create a compliant resource and see it succeed
6. Understand what just happened and why it matters

**Our specific goal:** Create a policy that requires every pod to have a label called `team` with a non-empty value.

Why this specific rule?

Labels are the foundation of resource organization in Kubernetes. They help you:
- Track which team owns which resources
- Build monitoring dashboards grouped by team
- Generate accurate cost reports
- Debug issues faster (you know who to ask)
- Apply team-specific policies or quotas

Requiring a `team` label is simple, immediately practical, and you'll see results in seconds. It's the perfect first policy.

By the end of this tutorial, you'll have a working Kyverno policy running in your cluster, actively enforcing this rule on every pod creation attempt. You'll see it block non-compliant pods and allow compliant ones — real enforcement, in real-time.

## Prerequisites Check: Are You Ready?

Before we begin, let's verify you have everything you need.

**The Quick Answer:** You need a Kubernetes cluster on your laptop. Don't worry — it's free and runs locally. No cloud account needed.

### Installation: Pick Your System

I'll walk you through installing Kind — the easiest way to run Kubernetes on your laptop.

#### macOS
```bash
# 1. Install Docker Desktop
brew install --cask docker

# 2. Open Docker Desktop (wait for it to start)

# 3. Install kubectl and Kind
brew install kubectl kind

# 4. Create your cluster
kind create cluster --name kyverno-learning
```

#### Windows
```powershell
# 1. Download and install Docker Desktop from:
# https://www.docker.com/products/docker-desktop/

# 2. Open PowerShell as Administrator, then:
choco install kubernetes-cli kind

# 3. Create your cluster
kind create cluster --name kyverno-learning
```

#### Linux
```bash
# 1. Install Docker
sudo apt-get update && sudo apt-get install -y docker.io
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and back in

# 2. Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl
# If stuck then use 
sudo snap install kubectl --classic # this command worked for me 
kubectl version --client

# 3. Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

# 4. Create your cluster
kind create cluster --name kyverno-learning
```

### Verify It Works

Run this command:
```bash
kubectl get nodes
```

Expected output:
```
NAME                            STATUS   ROLES           AGE
kyverno-learning-control-plane   Ready    control-plane   1m
```

✅ See "Ready"? You're done! Move to Step 1.

❌ Having issues? Check the troubleshooting guide below.

### Can't Install? Use This Instead

If installation doesn't work (corporate laptop, firewall, etc.):

**Option: Free Online Playground**
1. Go to https://killercoda.com/playgrounds/scenario/kubernetes
2. Click "Start"
3. Wait 30 seconds
4. You have Kubernetes in your browser!

⚠️ **Limitation:** Session expires after 1 hour.

### Troubleshooting Quick Fixes

- **"Docker is not running"** → Open Docker Desktop and wait for the whale icon to be solid (not animated)
- **"kind: command not found"** → Close and reopen your terminal
- **"Cannot connect to Docker daemon"** → Restart Docker Desktop
- **Still stuck?** → Use the online playground option above

### Already Have Kubernetes?

If you already have Minikube, k3s, or a cloud cluster — perfect! Just make sure:
```bash
kubectl get nodes
```
Shows at least one node with STATUS: Ready.

## Step 1: Install Kyverno (One Command, One Minute)

This is probably the easiest installation you'll ever do in Kubernetes.

### The Installation Command

Copy this command and run it in your terminal:
```bash
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.12.0/install.yaml
```

**What this command does:**
- Downloads the official Kyverno installation manifest
- Creates a namespace called `kyverno`
- Installs Kyverno's components (Custom Resource Definitions, ServiceAccounts, Deployments, etc.)
- Sets up admission webhooks (the mechanism Kyverno uses to intercept resource creation)

### What You'll See

After you run the command, you'll see a long list of output like this:
```
namespace/kyverno created
customresourcedefinition.apiextensions.k8s.io/admissionreports.kyverno.io created
customresourcedefinition.apiextensions.k8s.io/backgroundscanreports.kyverno.io created
customresourcedefinition.apiextensions.k8s.io/clusteradmissionreports.kyverno.io created
customresourcedefinition.apiextensions.k8s.io/clusterpolicies.kyverno.io created
...
serviceaccount/kyverno-service-account created
configmap/kyverno created
...
deployment.apps/kyverno-admission-controller created
service/kyverno-svc created
...
```

Don't worry about understanding every line. The key thing is seeing "created" appear many times. This means Kyverno is installing itself into your cluster.

### Wait for Kyverno to Start

Kyverno's pods need 30–60 seconds to start up and become ready. This is normal.

Wait about 30 seconds, then verify the installation:
```bash
kubectl get pods -n kyverno
```

What you're looking for:
```
NAME                                             READY   STATUS    RESTARTS   AGE
kyverno-admission-controller-xxxxxxxxxx-xxxxx    1/1     Running   0          45s
kyverno-background-controller-xxxxxxxxxx-xxxxx   1/1     Running   0          45s
kyverno-cleanup-controller-xxxxxxxxxx-xxxxx      1/1     Running   0          45s
kyverno-reports-controller-xxxxxxxxxx-xxxxx      1/1     Running   0          45s
```

Key indicators of success:
- READY column shows `1/1` for all pods
- STATUS column shows `Running` for all pods
- No errors or `CrashLoopBackOff`

**If you see "ContainerCreating" or "Pending":** Wait another 30 seconds and check again. Depending on your cluster and internet speed, it might take a minute.

**If you see "ImagePullBackOff" or "CrashLoopBackOff":** Your cluster might have resource constraints or network issues. Check `kubectl describe pod -n kyverno <pod-name>` for details.

### What Just Happened?

Kyverno is now installed and running in your cluster. It created:
- **A dedicated namespace:** All Kyverno components live in the `kyverno` namespace
- **Custom Resource Definitions (CRDs):** These allow you to create `ClusterPolicy` and `Policy` resources
- **Admission webhooks:** These intercept resource creation requests and send them to Kyverno for validation
- **Controllers:** Background processes that watch for policies and resources

Think of it like this: You just installed a security system in your house. The system is powered on and monitoring, but it's not enforcing any rules yet because you haven't configured any. That's what we're doing next.

## Step 2: Understanding What You Installed (Quick Context)

Before we write our first policy, let's take 60 seconds to understand what's now running in your cluster.

### The Components

When you installed Kyverno, several controllers were deployed:

**1. Admission Controller**  
This is the main component that intercepts resource creation and update requests. When you run `kubectl apply`, your request goes to the Kubernetes API server, which then asks Kyverno "Is this okay?" before creating the resource.

**2. Background Controller**  
This scans existing resources in your cluster and generates reports on policy compliance. It's useful for auditing what's already running.

**3. Cleanup Controller**  
This handles time-based resource cleanup (an advanced feature we won't use today).

**4. Reports Controller**  
This generates detailed compliance reports showing which resources pass or fail policies.

### How It Actually Works (Simplified)

Here's the flow when you try to create a resource:

1. You run `kubectl apply -f pod.yaml`
2. kubectl sends the request to the Kubernetes API server
3. Before creating the pod, the API server asks Kyverno: "Is this okay?"
4. Kyverno checks the pod against all your policies
5. If the pod passes all policies, Kyverno says "approved" and the pod is created
6. If the pod fails a policy, Kyverno says "rejected" and returns an error to you
7. The pod never gets created if it violates a policy

This all happens in milliseconds. From your perspective as a user, it feels instant — you just see either a success message or an error.

### Important: Existing Resources Are Untouched

Kyverno only validates NEW resources and UPDATED resources. Everything already running in your cluster is not affected by Kyverno policies (unless you specifically configure background scanning, which is optional).

This means you can't break existing workloads by installing Kyverno or creating policies. It only affects future deployments.

Okay, enough theory. Let's write some policy code.

## Step 3: Writing Your First Policy

This is where things get exciting. We're going to write a Kyverno policy from scratch.

### Our Goal

Create a policy that says: "Every pod must have a label called `team` with a non-empty value."

If someone tries to create a pod without this label, Kyverno will block it and explain why.

### The Complete Policy

Here's the full policy YAML. Yes, it looks like a lot at first glance, but I promise we'll break down every single line:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-team-label
  annotations:
    policies.kyverno.io/title: Require Team Label
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/description: >-
      Every pod must have a 'team' label to ensure proper resource tracking and ownership.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-for-team-label
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "All pods must have a 'team' label. Add metadata.labels.team to your pod spec."
      pattern:
        metadata:
          labels:
            team: "?*"
```

Don't panic. Let's translate this into plain English, section by section.

## Step 4: Understanding the Policy (Plain English Translation)

Let me walk you through this policy line by line, explaining what each part does and why it matters.

### Header Section
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
```

**What it means:** This tells Kubernetes "Hey, this is a Kyverno ClusterPolicy resource."

- `apiVersion`: Defines which API group this resource belongs to (Kyverno's API)
- `kind: ClusterPolicy`: Means this policy applies to the entire cluster, across all namespaces

**Alternative:** You could use `kind: Policy` (without "Cluster") if you wanted the policy to only apply to a single namespace.

### Metadata Section
```yaml
metadata:
  name: require-team-label
  annotations:
    policies.kyverno.io/title: Require Team Label
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/description: >-
      Every pod must have a 'team' label to ensure proper resource tracking and ownership.
```

**What it means:**
- `name`: A unique identifier for this policy. Use descriptive names like "require-team-label" instead of "policy1"
- `annotations`: Optional metadata that helps with documentation and reporting
  - `title`: Human-readable name
  - `category`: Helps organize policies (Security, Best Practices, etc.)
  - `severity`: Indicates importance (low, medium, high, critical)
  - `description`: Explains why this policy exists

**Why annotations matter:** They show up in policy reports and help your team understand what policies do without reading code.

### Spec Section (The Important Part)
```yaml
spec:
  validationFailureAction: Enforce
  background: true
```

**What it means:**
- `validationFailureAction: Enforce`: This is critical.
  - `Enforce` = Block resources that violate this policy
  - `Audit` = Just report violations, don't block anything
  - We're using `Enforce` because we want to see Kyverno actually stop non-compliant pods
- `background: true`: Run this policy against existing resources too (for reporting). When true, Kyverno will scan existing pods and generate compliance reports.

### The Rule
```yaml
rules:
- name: check-for-team-label
```

A policy can contain multiple rules. Each rule is evaluated independently. We have just one rule here.

Rule name: `check-for-team-label` - descriptive so we know what it does.

### Match Section (What Does This Apply To?)
```yaml
match:
  any:
  - resources:
      kinds:
      - Pod
```

**What it means:** "This rule applies to any resource of kind Pod."

**Translation:** When someone tries to create or update a Pod, run this rule.

You could also match:
- Specific namespaces: `namespaces: [production, staging]`
- Specific names: `name: "nginx-*"`
- Multiple resource types: `kinds: [Pod, Deployment]`

### Validate Section (The Actual Check)
```yaml
validate:
  message: "All pods must have a 'team' label. Add metadata.labels.team to your pod spec."
  pattern:
    metadata:
      labels:
        team: "?*"
```

This is where the magic happens.

- `message`: The error message users see when their pod is rejected. Make it helpful and actionable!
- `pattern`: Defines what we're looking for in the resource
  - `metadata.labels.team`: Navigate to the labels section and look for a key called `team`
  - `"?*"`: This is a wildcard meaning "any non-empty string"
    - `?` = at least one character
    - `*` = followed by zero or more characters
    - Combined: `?*` = "any string with at least one character"

**Translation of the pattern:** "Check if the pod has a label called `team`, and make sure it has a value (not empty)."

### Putting It All Together

In plain English, this policy says:

> "For any Pod that someone tries to create in this cluster, check if it has a metadata label called `team` with a non-empty value. If it doesn't, block the pod from being created and show the user this error message: 'All pods must have a team label. Add metadata.labels.team to your pod spec.'"

That's it. That's what 30 lines of YAML accomplish.

## Step 5: Deploy Your Policy

Now let's put this policy into action in your cluster.

### Create the Policy File

1. Open your favorite text editor (VS Code, vim, nano, whatever you prefer)
2. Create a new file called `require-team-label.yaml`
3. Copy the complete policy YAML from Step 3 above
4. Save the file

### Apply the Policy

In your terminal, navigate to the directory where you saved the file and run:
```bash
kubectl apply -f require-team-label.yaml
```

Expected output:
```
clusterpolicy.kyverno.io/require-team-label created
```

That one line means: Your policy is now active and enforcing rules in your cluster!

### Verify the Policy Exists

Let's confirm Kyverno registered your policy:
```bash
kubectl get clusterpolicy
```

Expected output:
```
NAME                  BACKGROUND   VALIDATE ACTION   READY   AGE
require-team-label    true         Enforce           True    15s
```

Key things to check:
- Your policy name appears in the list
- VALIDATE ACTION shows `Enforce`
- READY shows `True` (if `False`, there's a configuration issue)

You can also see more details:
```bash
kubectl describe clusterpolicy require-team-label
```

This shows the full policy definition and any status information.

Your cluster is now enforcing this rule. Any pod created from this moment forward must have a `team` label. Let's test it!

## Step 6: Testing Time! (The Fun Part)

This is where you see Kyverno in action and everything clicks into place. We're going to create two pods:

1. A "bad" pod without the required label → should be blocked
2. A "good" pod with the required label → should succeed

### Test 1: The Non-Compliant Pod (We Want This to Fail)

Let's try to create a pod that violates our policy.

Create a file called `bad-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-without-label
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

**Notice:** There are NO labels in the metadata section. This pod violates our policy.

Now try to create it:
```bash
kubectl apply -f bad-pod.yaml
```

### The Error You WANT to See

You should get an error that looks like this:
```
Error from server: error when creating "bad-pod.yaml": admission webhook "validate.kyverno.svc-fail" denied the request: 
resource Pod/default/nginx-without-label was blocked due to the following policies 
require-team-label:
  check-for-team-label: 'validation error: All pods must have a ''team'' label. Add metadata.labels.team to your pod spec. rule check-for-team-label failed at path /metadata/labels/team/'
```

### Let's Celebrate This Error!

This error is EXACTLY what we want! Here's what just happened:

1. You tried to create a pod without a `team` label
2. kubectl sent the request to the Kubernetes API server
3. The API server asked Kyverno "Should I allow this?"
4. Kyverno checked the pod against your `require-team-label` policy
5. Kyverno found that the pod is missing the required label
6. Kyverno said "REJECT this pod" and provided your custom error message
7. The API server blocked the pod and showed you the error
8. The pod was never created

Verify this:
```bash
kubectl get pods
```

You should see no pod named `nginx-without-label`. It was rejected before it could be created.

This is Kyverno working exactly as intended. Your cluster just got more secure and more compliant — automatically.

### Test 2: The Compliant Pod (This Should Work)

Now let's create a pod that follows the rules.

Create a file called `good-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-label
  labels:
    team: platform-engineering
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

**Notice the difference:** We added a `labels` section with `team: platform-engineering`. This satisfies our policy.

Apply it:
```bash
kubectl apply -f good-pod.yaml
```

Expected output:
```
pod/nginx-with-label created
```

Success! The pod was created because it has the required `team` label.

Verify it's running:
```bash
kubectl get pods
```

Expected output:
```
NAME                 READY   STATUS    RESTARTS   AGE
nginx-with-label     1/1     Running   0          10s
```

You can also see the labels:
```bash
kubectl get pod nginx-with-label --show-labels
```

Output:
```
NAME               READY   STATUS    RESTARTS   AGE   LABELS
nginx-with-label   1/1     Running   0          30s   app=nginx,team=platform-engineering
```

### What You Just Accomplished

This is huge. In the span of a few minutes, you:

1. Created a policy from scratch
2. Deployed it to your cluster
3. Saw it actively block a non-compliant resource in real-time
4. Saw it allow a compliant resource through
5. Enforced a meaningful standard (resource labeling) automatically

You're officially using Kyverno to govern your Kubernetes cluster. Everything you deploy from now on will be checked against this policy.

## Step 7: What Just Happened? (Behind the Scenes)

Let me walk you through the entire flow so you understand exactly what's happening when Kyverno validates resources.

### The Journey of the Bad Pod

Here's what happened when you tried to create `nginx-without-label`:

1. You ran: `kubectl apply -f bad-pod.yaml`
2. kubectl sent the pod definition to the Kubernetes API server over HTTPS
3. The API server received the request and started processing it
4. Before creating the pod, the API server checked its admission webhooks and found Kyverno registered
5. The API server sent the pod definition to Kyverno's webhook endpoint: "Hey Kyverno, should I allow this?"
6. Kyverno received the pod and started evaluating it
7. Kyverno found your `require-team-label` policy
8. Kyverno checked if the policy's match criteria applied (yes, it's a Pod)
9. Kyverno ran the validation pattern: looking for `metadata.labels.team`
10. Kyverno didn't find the `team` label in the pod's metadata
11. Kyverno responded to the API server: "REJECT this pod. Validation failed."
12. The API server rejected the request and returned your custom error message to kubectl
13. kubectl displayed the error to you
14. The pod was never created (it never made it to etcd, never got scheduled)

Total time elapsed: ~50–100 milliseconds. It's instant from your perspective.

### The Journey of the Good Pod

Here's what happened when you created `nginx-with-label`:

1. Steps 1–9: Same as above (request sent, webhook called, policy matched, validation started)
2. Kyverno found the `team` label with value `platform-engineering`
3. The pattern matched: `team: "?*"` (any non-empty string) ✓
4. Kyverno responded to the API server: "APPROVE this pod. Validation passed."
5. The API server created the pod object in etcd
6. The Kubernetes scheduler assigned the pod to a node
7. The kubelet on that node pulled the nginx image and started the container
8. kubectl displayed the success message
9. The pod is now running and serving traffic

### The Key Insight

Kyverno sits in the critical path between your `kubectl apply` command and the actual resource creation. It's like a checkpoint that every resource must pass through.

This architecture means:
- Non-compliant resources are caught immediately
- Developers get instant feedback
- Bad configurations never make it to production
- No need to scan and fix resources after deployment

This is policy-as-code working exactly as intended.

## Step 8: Understanding the Three Things Kyverno Can Do

We just used "validation" to check and block resources. But Kyverno can actually do three different types of policy enforcement. Let me explain each one with simple examples.

### 1. Validate (What We Just Did)

**What it does:** Checks if a resource meets your requirements. If not, blocks it.

**Think of it as:** A bouncer at a club checking IDs. No valid ID? You're not getting in.

**Example we used:** "Every pod must have a team label"

**Real-world examples:**
- "Containers must not run as root user"
- "Images must come from approved registries only"
- "All deployments must have resource limits set"

### 2. Mutate (Automatic Fixes)

**What it does:** Automatically modifies resources to add missing fields or fix configurations.

**Think of it as:** Autocorrect on your phone. You type "teh" and it changes it to "the" automatically.

**Example:** "If a pod doesn't have a team label, automatically add `team: unknown`"

With mutation, that bad pod we tried to create earlier would have been automatically fixed instead of rejected. Kyverno would have:
1. Seen the pod is missing the `team` label
2. Added `team: unknown` automatically
3. Allowed the pod to be created

**Real-world examples:**
- Add a sidecar container to every pod in the production namespace
- Automatically inject environment variables
- Add default resource requests/limits if missing
- Append standard labels for cost tracking

### 3. Generate (Create Related Resources)

**What it does:** Automatically creates new resources when certain conditions are met.

**Think of it as:** A "copy to all" button. When you do one thing, Kyverno does several related things automatically.

**Example:** "When a new namespace is created, automatically create a default NetworkPolicy and ResourceQuota in that namespace"

This ensures every namespace starts with security policies and resource limits — without anyone having to remember to create them manually.

**Real-world examples:**
- Create default RBAC roles when a new namespace appears
- Generate ConfigMaps for each new deployment
- Auto-create backup CronJobs for StatefulSets
- Provision default network policies for isolation

All three types work the same way:
1. Write a YAML policy defining the rule
2. Apply it to your cluster
3. Kyverno enforces it automatically

Today we focused on validation because it's the easiest to understand and gives immediate, visible feedback. But mutation and generation use the exact same pattern — you just change one field in the policy spec.

## Step 9: Real-World Use Cases

Now that you understand how Kyverno works, let's talk about how teams actually use it in production. These aren't hypothetical — these are real policies running in real clusters.

### Security Policies

**Problem:** Containers running as root are a security risk. If compromised, an attacker has root access.

**Kyverno solution:** Validate that all containers run as non-root users.

```yaml
# Simplified example
validate:
  message: "Containers must run as non-root"
  pattern:
    spec:
      securityContext:
        runAsNonRoot: true
```

### Cost Management

**Problem:** Developers accidentally create expensive resources in non-production environments, racking up cloud bills.

**Kyverno solution:** Enforce strict resource limits in development namespaces.

```yaml
# Simplified example
validate:
  message: "Dev pods must have CPU under 1 core, memory under 2Gi"
  pattern:
    spec:
      containers:
      - resources:
          limits:
            cpu: "<=1"
            memory: "<=2Gi"
```

This prevents someone from spinning up a 32-core instance in the dev environment.

### Naming Standards

**Problem:** Teams naming resources inconsistently. Monitoring becomes difficult. Billing reports are messy.

**Kyverno solution:** Enforce naming conventions.

```yaml
# Simplified example
validate:
  message: "Deployment names must start with team prefix: 'platform-', 'data-', or 'ml-'"
  pattern:
    metadata:
      name: "platform-* | data-* | ml-*"
```

### Image Security

**Problem:** Developers pull images from Docker Hub or unknown registries. Supply chain security risk.

**Kyverno solution:** Only allow images from approved registries.

```yaml
# Simplified example
validate:
  message: "Images must come from company-registry.io or gcr.io/our-project"
  pattern:
    spec:
      containers:
      - image: "company-registry.io/* | gcr.io/our-project/*"
```

### Automatic Label Injection

**Problem:** Developers forget to add labels needed for monitoring, billing, and compliance.

**Kyverno solution:** Automatically inject required labels using mutation.

```yaml
# Simplified example
mutate:
  patchStrategicMerge:
    metadata:
      labels:
        environment: production
        cost-center: engineering
        managed-by: kyverno
```

Now every resource automatically gets tagged for billing and monitoring.

These policies are running in production right now at companies managing thousands of pods. And you now have the skills to create policies just like these.

## Cleaning Up Your Experiment

When you're done learning and want to remove everything:

### Delete Test Pods
```bash
kubectl delete pod nginx-with-label
# If you created others, delete them too
```

### Delete Your Policy
```bash
kubectl delete clusterpolicy require-team-label
```

### Remove Policy Files
```bash
kubectl delete -f require-team-label.yaml
kubectl delete -f bad-pod.yaml
kubectl delete -f good-pod.yaml
```

### Uninstall Kyverno Completely
```bash
kubectl delete -f https://github.com/kyverno/kyverno/releases/download/v1.12.0/install.yaml
```

This removes all Kyverno components from your cluster.

**Note:** You can always reinstall and start fresh:
```bash
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.12.0/install.yaml
```

Nothing is permanent. Kubernetes makes it safe to experiment.

## Resources for Continued Learning

Not all documentation is created equal. Here are resources specifically useful for beginners, with context on what each is good for.

### Official Kyverno Documentation
👉 https://kyverno.io/docs/

### Kyverno Policy Library
👉 https://kyverno.io/policies/

### Kyverno Playground
👉 https://playground.kyverno.io/

### Community Slack
👉 Join Kubernetes Slack → #kyverno channel

### YouTube Tutorials
👉 Search: "Kyverno tutorial" or "Kyverno deep dive"

### GitHub Discussions
👉 https://github.com/kyverno/kyverno/discussions

### Policy Reporter
👉 https://github.com/kyverno/policy-reporter

Not needed for beginners, but good to know it exists.

**My learning advice:** Don't try to read everything at once. Start with the Policy Library (deploy some pre-written policies), then experiment in the Playground, then dive into documentation when you have specific questions.

Learn by doing, not by reading.

---

## Contributing

This tutorial is a living document. If you find errors, have suggestions, or want to contribute improvements, please open an issue or pull request.

## License

This work is licensed under [MIT License](LICENSE)

---

⭐ **If you found this tutorial helpful, please star this repository!**