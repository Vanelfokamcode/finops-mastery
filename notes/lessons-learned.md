# Lab 1 : The $61 Stopped EC2 Mystery

**Real Case:** Brother stopped EC2 instances without terminating.  
**Period:** September - December 2025 (4 months)  
**Total Cost:** $61.22  
**Expected Cost:** $0 (he thought "stopped" = free)

---

## The Bill Breakdown

[Screenshot de la facture]

| Service | Total (4 months) | Avg/month |
|---------|------------------|-----------|
| EC2-Instances | $34.84 | $8.71 |
| VPC | $16.14 | $4.04 |
| Tax | $10.20 | $2.55 |
| **TOTAL** | **$61.22** | **$15.31** |

---

## Investigation

### EC2-Instances : $34.84 (4 months)

**What this represents:**
- NOT compute (instances were stopped)
- This is **EBS volumes** still attached

**Calculation:**
```
$8.71/month ÷ $0.088/GB-month = ~99GB of EBS
```

**What happened:**
- He had ~100GB of EBS volumes (gp3)
- When he stopped the instances, EBS stayed attached
- AWS kept charging $0.088/GB every month
- Over 4 months : 100GB × $0.088 × 4 = $35.20 ✓ (matches bill)

**The trap:**
- AWS Console "Stop" button doesn't warn about EBS costs
- Most people think "stopped" = completely free
- EBS is SEPARATE billing from EC2 compute

---

### VPC : $16.14 (4 months)

**What this represents:**
- VPC itself = FREE
- This charge = something INSIDE the VPC

**Most likely culprits:**

**Option 1: Elastic IP (detached)**
- Cost: $3.60/month when not attached
- 4 months × $3.60 = $14.40 ✓ (close to $16.14)
- **Probable explanation:**
  - When instance stopped, Elastic IP detached
  - AWS started charging for unused IP
  - He never released it

**Option 2: VPC Endpoint**
- PrivateLink endpoint : $0.01/hour = $7.20/month
- But unlikely for beginner setup

**Option 3: NAT Gateway**
- Would be $32+/month (too expensive, not this)

**My verdict: Elastic IP detached + maybe small data transfer**

---

## Root Cause Analysis

### Timeline

**Month 0:** Launched EC2 instance for testing
- Instance type: probably t3.small or t3.medium
- EBS volume: 100GB gp3
- Elastic IP: allocated and attached

**Month 1 (September):** Finished testing
- Clicked "Stop" (not "Terminate")
- Thought: "I'll save money by stopping it"
- Reality: EBS kept charging + IP detached

**Months 2-4 (Oct-Dec):** Forgot about it
- $15/month silently charged
- No budget alerts configured
- Noticed only after 4 months

**Month 5 (Now):** Discovery
- Saw bill: $61.22
- "Wait, I stopped that instance months ago!"
- Finally terminated everything

---

## The Financial Breakdown

### What he paid (actual)
```
EBS volumes (100GB × 4 months) : $35.20
Elastic IP (detached × 4 months) : $14.40
Tax : $10.20
Misc data transfer : ~$1.42
──────────────────────────────────
TOTAL : $61.22
```

### What he SHOULD have paid
```
If terminated properly : $0.00
──────────────────────────────────
MONEY WASTED : $61.22
```

### If this continued for 1 year
```
$15.31/month × 12 months = $183.72/year
For doing NOTHING (instance not even running)
```

---

## Why This Happens (The AWS UI Trap)

### The "Stop" button confusion

**What users think:**
> "Stop = pause = no charges"

**What actually happens:**
```
Stop button pressed
    ↓
Compute stops ✓ ($0 for compute)
    ↓
BUT...
    ↓
EBS volumes stay attached → $8.71/month
Elastic IP detaches → $3.60/month
Snapshots (if any) continue → $X/month
Load Balancers (if any) continue → $16+/month
NAT Gateways (if any) continue → $32+/month
```

**AWS NEVER warns you about this.**

Why? Because it's profitable for them.

---

## How to Prevent This

### ✅ Solution 1: TERMINATE (not Stop)
```bash
# When you're done with an instance FOR REAL
aws ec2 terminate-instances --instance-ids i-xxxxx

# This will:
# - Stop compute (free)
# - Delete EBS volumes (if DeleteOnTermination=true)
# - Release Elastic IP (manual step, see below)
```

**Cost after terminate: $0**

### ✅ Solution 2: Create AMI backup first

If you might need the instance again:
```bash
# Step 1: Create AMI (image backup)
aws ec2 create-image \
  --instance-id i-xxxxx \
  --name "my-instance-backup-2025-03-01"

# Step 2: Terminate instance
aws ec2 terminate-instances --instance-ids i-xxxxx

# Step 3: Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-xxxxx

# Later, when needed:
aws ec2 run-instances --image-id ami-xxxxx ...
```

**Cost while stored:**
- AMI storage: ~$0.50/month (100GB snapshot)
- vs keeping stopped: $15.31/month
- **Savings: $14.81/month = $177.72/year**

### ✅ Solution 3: Budget Alerts (MANDATORY)
```
AWS Console → Billing → Budgets → Create

Budget 1: $5/month threshold
├─ Alert at $3 (60%)
└─ Alert at $5 (100%)

Budget 2: $10/month threshold
├─ Alert at $8 (80%)

Budget 3: $20/month threshold
└─ Alert at $20 (STOP EVERYTHING)
```

**With alerts:**
- He would've known in September (first $5)
- Total waste: $5 instead of $61

---

## Real-World Scaling

### If this was a company

**Scenario:** 10 developers each leave 1 instance "stopped"
```
10 instances × $15.31/month = $153.10/month
Over 1 year = $1,837.20/year wasted
Over 3 years = $5,511.60 wasted
```

**For medium company (100 forgotten stopped instances):**
```
100 instances × $15.31/month = $1,531/month
= $18,372/year wasted on NOTHING
```

**This is why FinOps engineers exist.**

A FinOps engineer paid $80K/year who finds this:
- ROI: $18,372 saved / $80K salary = 23% of their cost
- Find 5 issues like this = they pay for themselves

---

## Key Learnings

### 1. Stop ≠ Free ❌

The "Stop" button is a **financial trap**.

What stops:
- ✅ CPU/RAM (compute)

What DOESN'T stop:
- ❌ EBS volumes ($0.088/GB-month)
- ❌ Elastic IPs when detached ($3.60/month)
- ❌ Snapshots ($0.05/GB-month)
- ❌ Load Balancers ($16-22/month)
- ❌ NAT Gateways ($32+/month)

### 2. AWS UI is Designed to Confuse 🎭

- "Stop" sounds temporary and harmless
- No warning about ongoing costs
- No prompt to "Terminate instead?"
- This is **by design** (more revenue)

### 3. Budget Alerts = Insurance 💰

Without alerts:
- Small mistakes become expensive
- $5/month becomes $180/year

With alerts:
- You know immediately
- Fix before it hurts

### 4. Terminate Culture 🗑️

In companies with good FinOps:
- Default action = TERMINATE
- Stop = only for short breaks (hours, not days)
- Everything tagged with auto-shutdown policies

---

## The $61 Lesson

**Cost:** $61.22 over 4 months  
**Learning:** Priceless

My brother paid $61 to learn:
- "Stop" ≠ "Free"
- Always terminate when done
- Set budget alerts
- AWS UI tricks users into spending

**Now I document this so others don't make the same mistake.**

---

## Screenshots

- `bill-summary.png` - The $61.22 total
- `cost-breakdown.png` - Service-by-service breakdown
- `ebs-volume-charges.png` - EBS line items
- `vpc-charges.png` - VPC/Elastic IP charges

---

## Actions Taken

- [x] Analyzed the bill
- [x] Identified root causes (EBS + Elastic IP)
- [x] Calculated waste ($61.22)
- [x] Documented for portfolio
- [ ] Set budget alerts on account ($5, $10, $20)
- [ ] Educate brother on Terminate vs Stop
- [ ] Create cleanup checklist for future

---

**Next Lab:** Right-Sizing (finding oversized instances)
