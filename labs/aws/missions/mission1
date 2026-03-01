# Mission 1: SaaS Startup EC2 Audit

**Client Code:** TF-2024-01 (TechFlow - anonymized from real r/aws case)  
**Industry:** SaaS B2B (Project Management Tool)  
**Engagement Type:** 3-hour FinOps consultation  
**Your Rate:** $150/hour (you're learning, but let's pretend)  
**Date:** March 1, 2025

---

## 📋 PRE-MISSION BRIEFING

### Client Background

**TechFlow Inc.**
- Founded: September 2023 (18 months ago)
- Product: Agile project management tool (Jira competitor)
- Team: 12 people
  - 3 developers (full-stack)
  - 1 DevOps (part-time, junior level)
  - 8 business/sales/support
- Customers: 200 paying (SMBs, 10-50 employees each)
- Revenue: $15,000/month ($180K annual run rate)
- AWS spend: Started at $150/month, now $500/month
- Profitability: Breaking even (AWS is 3.3% of revenue)

### Why They Hired You

**Email from CEO (Sarah):**

> "Hi,
> 
> Our AWS bill has tripled in 18 months but our customers only doubled. 
> Our dev team says 'everything is running fine' but I'm worried we're 
> wasting money.
> 
> We're not AWS experts. We just want things to work and not cost too much.
> Can you take a quick look and tell us if we're doing something stupid?
> 
> Budget: $500 for this consultation (3 hours max).
> 
> Thanks,
> Sarah (CEO)"

---

## 🔍 PHASE 1: INITIAL DATA GATHERING (15 min)

### What You Receive

Sarah gives you:
1. **Read-only AWS Console access** (IAM user: finops-consultant)
2. **Last 3 months of AWS bills** (PDF)
3. **Architecture diagram** (hand-drawn by their DevOps)

### AWS Bill Summary (Last Month)
```
December 2024 AWS Bill: $487.34

Breakdown:
- EC2-Instances: $187.23 (38%)
- VPC: $89.45 (18%)
- RDS: $156.89 (32%)
- S3: $23.12 (5%)
- CloudFront: $18.76 (4%)
- Other: $11.89 (3%)
```

**Your focus today:** EC2-Instances ($187.23)

---

## 💻 PHASE 2: EC2 AUDIT (30 min)

### What You Find in AWS Console

**Region:** us-east-1 (N. Virginia)

You run this command:
```bash
aws ec2 describe-instances --region us-east-1 --output table
```

**Results:**

### Instance 1: production-app-server
```
Instance ID: i-0a1b2c3d4e5f67890
Name: production-app-server
Type: t3.large
State: running
vCPU: 2
RAM: 8 GB
Launch time: 2023-09-15 (547 days ago)
Uptime: 547 days continuous

Storage:
- Root volume: 100 GB gp3
- Additional volume: None

Network:
- Elastic IP: 54.123.45.67 (allocated, attached)
- Public IPv4: Yes

Tags:
- Environment: production
- Owner: dev-team
```

**Your notes:** 
- Has been running since company launch
- Never been stopped or resized
- Chosen size: t3.large (DevOps said "we need power for production")

---

### Instance 2: production-worker
```
Instance ID: i-0b2c3d4e5f6789012
Name: production-worker
Type: t3.medium
State: running
vCPU: 2
RAM: 4 GB
Launch time: 2023-09-20
Uptime: 542 days

Storage:
- Root volume: 50 GB gp3

Network:
- Public IPv4: Yes (no Elastic IP)

Tags:
- Environment: production
- Purpose: background-jobs
```

**Your notes:**
- Runs Sidekiq (Ruby background job processor)
- Processes emails, notifications, reports
- Utilization unknown (no monitoring visible)

---

### Instance 3: staging-server
```
Instance ID: i-0c3d4e5f678901234
Name: staging-server
Type: t3.medium
State: stopped ⚠️
vCPU: 2
RAM: 4 GB
Launch time: 2023-11-10
Stop time: 2024-08-15 (120 days ago!)

Storage:
- Root volume: 100 GB gp3 (still attached!)

Network:
- Elastic IP: 54.123.45.68 (allocated, detached)

Tags:
- Environment: staging
- Owner: bob
```

**Your notes:**
- Stopped 4 months ago
- Bob left the company 3 months ago (Sarah mentions this)
- No one knows why it's still there
- **RED FLAG: This is costing money!**

---

### Instance 4: old-migration-server
```
Instance ID: i-0d4e5f67890123456
Name: old-migration-server
Type: m5.large ⚠️
State: stopped ⚠️
vCPU: 2
RAM: 8 GB
Launch time: 2023-09-18
Stop time: 2023-11-01 (450 days ago!!)

Storage:
- Root volume: 200 GB gp3 (still attached!)
- Additional volume: 100 GB gp2 (legacy)

Network:
- Public IPv4: Yes

Tags:
- Purpose: data-migration-from-old-system
- Owner: dev-team
```

**Your notes:**
- Used for one-time data migration when launching
- Migration completed November 2023
- Stopped since then but never terminated
- **HUGE RED FLAG: 15 months of waste!**

---

### Instance 5: alice-test-server
```
Instance ID: i-0e5f678901234567
Name: alice-test-server
Type: t3.small
State: stopped ⚠️
vCPU: 2
RAM: 2 GB
Launch time: 2024-09-20
Stop time: 2024-12-01 (90 days ago)

Storage:
- Root volume: 30 GB gp3

Network:
- Public IPv4: Yes

Tags:
- Owner: alice
- Purpose: testing-new-feature
```

**Your notes:**
- Alice is still with company (you can ask her about this)
- Probably forgot about it after feature was deployed
- **Smaller but still waste**

---

## 📊 PHASE 3: YOUR ANALYSIS (20 min)

### Your Tasks

#### Task 1: Calculate Stopped Instance Costs

For **EACH** stopped instance (3, 4, 5), calculate:

1. **Monthly EBS cost**
   - Formula: Volume size (GB) × $0.10/GB-month (gp3 in us-east-1)
   - Note: Instance 4 has 2 volumes!

2. **Monthly IPv4 cost** (if applicable)
   - Formula: $0.005/hour × 730 hours = $3.65/month per IP

3. **Monthly Elastic IP cost** (if detached)
   - Formula: $3.60/month per IP

4. **Total monthly waste** per instance

5. **Total waste since stopped**
   - Monthly waste × months stopped

**Hint for Instance 3:**
```
EBS: 100 GB × $0.10 = $10/month
IPv4: $3.65/month
Elastic IP (detached): $3.60/month
Total: $17.25/month
Duration: 4 months
Wasted so far: $17.25 × 4 = $69
```

**Your job:** Calculate for instances 4 and 5 yourself.

---

#### Task 2: Identify the "WHY"

For each stopped instance, determine:
- Why was it stopped instead of terminated?
- What SHOULD have been done?
- Is there any valid reason to keep it?

**Example for Instance 4:**
```
Why stopped: Migration was done, DevOps clicked "Stop" instead of "Terminate"
Why not terminated: Probably fear of "what if we need it" + lack of knowledge
Valid reason to keep: NONE. Migration was 15 months ago.
Should have: Terminated immediately after migration complete.
Alternative: Create AMI if paranoid, then terminate.
```

---

#### Task 3: Calculate Running Instance Costs (Bonus)

For instances 1 and 2 (running), calculate:

**Instance 1 (t3.large running):**
```
Compute: $0.0832/hour × 730 hours = ?
EBS: 100 GB × $0.10 = ?
IPv4: $3.65/month
Elastic IP: $0 (attached to running)
Total: ?
```

**Question:** Is t3.large necessary? (You don't have CloudWatch data yet, but note this for future investigation)

---

## 📝 PHASE 4: YOUR DELIVERABLE (15 min)

### What to Write

Create a file: `takeaways/mission-01-solution.md`

**Structure:**
```markdown
# Mission 1: TechFlow EC2 Audit Report

**Consultant:** [Your name]
**Date:** March 1, 2025
**Client:** TechFlow Inc.
**Time spent:** [Your actual time]

---

## Executive Summary

[2-3 sentences: What you found, total waste, recommended actions]

Example:
"Found 3 stopped EC2 instances costing $XX/month with no business value. 
Total waste: $XXX over past 15 months. Immediate termination will save 
$XX/month = $XXX/year going forward."

---

## Findings

### 1. Stopped Instance Waste

**Instance 3: staging-server**
- Type: t3.medium
- Stopped: 120 days ago
- Monthly cost: $XX (breakdown: EBS $X + IPv4 $X + Elastic IP $X)
- Total wasted: $XX
- Recommendation: [Your recommendation]

**Instance 4: old-migration-server**
- Type: m5.large
- Stopped: 450 days ago
- Monthly cost: $XX
- Total wasted: $XX
- Recommendation: [Your recommendation]

**Instance 5: alice-test-server**
- [Fill in]

**TOTAL MONTHLY WASTE:** $XX/month  
**TOTAL HISTORICAL WASTE:** $XXX  
**ANNUAL SAVINGS IF FIXED:** $XXX/year

---

## Recommendations

### Immediate Actions (Do Today - 15 minutes)

1. **Terminate Instance 4** (old-migration-server)
   - Command: `aws ec2 terminate-instances --instance-ids i-0d4e5f678...`
   - Savings: $XX/month
   - Risk: None (migration was 15 months ago)

2. **Ask Alice about Instance 5**
   - If she doesn't need it: Terminate
   - If she might need config: Create AMI first, then terminate
   - Savings: $X/month

3. **Check with team about Instance 3**
   - Does anyone use staging environment?
   - If yes: Restart when needed, don't keep stopped
   - If no: Terminate
   - Savings: $XX/month

### Short-term Actions (This Week)

4. **Set up AWS Budget Alerts**
   - Threshold: $550/month (current + 10%)
   - Alert emails to: CEO + DevOps
   - Purpose: Catch future waste within 30 days

5. **Review Running Instances**
   - Check CloudWatch CPU/RAM metrics (need monitoring enabled first)
   - Instance 1 (t3.large) might be oversized (future investigation)

### Long-term Actions (This Month)

6. **Implement Tagging Policy**
   - All resources must have: Owner, Environment, Purpose, DeleteAfter
   - Allows cost tracking per team/project

7. **Scheduled Start/Stop for Staging**
   - If staging is recreated, use Lambda to auto-stop at night
   - Savings: ~60% on staging costs

---

## Cost-Benefit Analysis

**Investment:**
- Your consultation: $500 (3 hours × $150)
- Implementation time: 30 minutes (free, internal)
- Total: $500

**Savings:**
- Monthly: $XX
- Annual: $XXX
- 3-year: $XXX

**ROI: XXX% in first year**

---

## Questions for Client

[List any questions you have for Sarah or the team]

Example:
1. Does anyone actively use the staging environment?
2. Is there a reason Instance 1 is t3.large and not t3.medium?
3. Who has permission to launch instances? (governance issue)

---

## Lessons Learned

[Personal notes - what this taught YOU]

Example:
"The old-migration-server being stopped for 15 months shows lack of 
cleanup process after one-time tasks. This is common. Need to build 
culture of 'delete after use' or at least 'set expiry dates on resources'."
```

---

## 🎯 SELF-CHECK QUESTIONS (Answer These BEFORE Writing Report)

### Question 1: Basic Calculation
```
Instance 4 has:
- Root volume: 200 GB gp3
- Additional volume: 100 GB gp2 (older type, $0.10/GB too)
- Stopped for 15 months

What is the TOTAL waste on this instance since it was stopped?

A) $150
B) $450
C) $657
D) $900

[Calculate before looking at answer]
```

### Question 2: Strategy Decision
```
Alice (developer) says about Instance 5:
"Oh yeah, I might need that next month for another test."

What do you recommend?

A) Keep it stopped (she might need it)
B) Terminate it now (she can create new one)
C) Create AMI, then terminate
D) Start it now so it's ready for her

[Think about cost vs benefit before answering]
```

### Question 3: Prioritization
```
You only have 15 minutes to fix ONE thing before your call ends.
Which instance do you terminate first and why?

A) Instance 3 (staging, 4 months stopped, owner left company)
B) Instance 4 (migration, 15 months stopped, largest waste)
C) Instance 5 (test, 3 months stopped, owner still here)

[Consider: waste amount, business risk, speed of decision]
```

### Question 4: Communication
```
Sarah (CEO) asks: "In simple terms, what happened here?"

Which explanation is BEST?

A) "Your team made mistakes and wasted money on stopped instances."

B) "AWS's 'Stop' button is misleading. Your team thought stopped = free, 
    but storage keeps charging. This is a common trap."

C) "You need better governance and monitoring to prevent this."

D) "The old-migration-server has been stopped for 450 days costing $X/month."

[Which is most helpful and professional?]
```

### Question 5: Trick Question (Gotcha!)
```
You calculate total savings as $67/month from terminating stopped instances.

Sarah says: "Great! So we'll save $67/month. But wait... the EC2 part 
of our bill is $187/month. Why is it so high if the waste is only $67?"

What's your answer?

Hint: Look at what's RUNNING. What might be the other issue?

[This tests if you understand stopped waste is just ONE type of waste]
```

---

## 📚 REFERENCE DATA (For Your Calculations)

### EC2 Pricing (us-east-1, On-Demand)

| Instance Type | vCPU | RAM | $/hour | $/month |
|---------------|------|-----|--------|---------|
| t3.small | 2 | 2 GB | $0.0208 | $15.18 |
| t3.medium | 2 | 4 GB | $0.0416 | $30.37 |
| t3.large | 2 | 8 GB | $0.0832 | $60.74 |
| m5.large | 2 | 8 GB | $0.096 | $70.08 |

### EBS Pricing (us-east-1)

| Type | $/GB-month |
|------|------------|
| gp3 (default) | $0.08 |
| gp2 (legacy) | $0.10 |

### Other Costs

| Item | Cost |
|------|------|
| Public IPv4 address | $0.005/hour = $3.65/month |
| Elastic IP (attached) | $0 |
| Elastic IP (detached) | $3.60/month |

---

## ✅ DELIVERABLES CHECKLIST

Before you submit your report:

- [ ] Calculated monthly cost for each stopped instance
- [ ] Calculated total waste since each was stopped
- [ ] Provided specific termination commands
- [ ] Prioritized recommendations (immediate vs later)
- [ ] Included cost-benefit analysis
- [ ] Wrote in clear, non-technical language (CEO will read this)
- [ ] Answered all 5 self-check questions honestly
- [ ] Took notes on what YOU learned (for your takeaway)

---

## 🎓 GRADING CRITERIA (Self-Assessment)

Rate yourself:

**Calculations (40 points):**
- [ ] All costs calculated correctly (15 pts)
- [ ] Showed formulas/work (10 pts)
- [ ] Caught the 2-volume trick on Instance 4 (15 pts)

**Recommendations (30 points):**
- [ ] Prioritized by impact (10 pts)
- [ ] Specific and actionable (10 pts)
- [ ] Considered business context (10 pts)

**Communication (20 points):**
- [ ] Clear executive summary (10 pts)
- [ ] Professional tone (10 pts)

**Critical Thinking (10 points):**
- [ ] Identified root causes (5 pts)
- [ ] Suggested prevention (5 pts)

**Total: ___/100**

- 90-100: Senior FinOps level
- 75-89: Mid-level, good work
- 60-74: Junior, needs improvement
- <60: Review the lesson again

---

**Time to complete: 40 minutes**

**Good luck! 🚀**

---

## 💡 HINTS (Only Read If Stuck)

<details>
<summary>Hint for Instance 4 calculation</summary>

Instance 4 has TWO volumes:
- 200 GB gp3: 200 × $0.08 = $16/month
- 100 GB gp2: 100 × $0.10 = $10/month
- IPv4: $3.65/month
- Total: $29.65/month
- Duration: 15 months
- Total waste: $444.75

This is the BIGGEST waste.
</details>

<details>
<summary>Hint for Question 2 (Alice's instance)</summary>

Best answer: C (Create AMI, then terminate)

Why:
- AMI = $3/month (30 GB snapshot)
- Stopped = $6/month (EBS + IPv4)
- Savings: $3/month
- If she needs it: 10 minutes to recreate from AMI
- Risk: Low (test server, not production)
</details>

<details>
<summary>Hint for Question 4 (CEO communication)</summary>

Best answer: B

Why:
- Doesn't blame team (unprofessional)
- Explains the trap (educational)
- Acknowledges it's common (not their fault)
- Sets up for recommendations

Avoid A (blaming), C (vague), D (too technical without context)
</details>
