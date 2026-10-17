# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49.0
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->253.6<!--/answer--> | <!--answer:vm_warm_median_ms-->211.9<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->1879.6<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->636.5<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1577.1<!--/answer--> | <!--answer:function_warm_median_ms-->641.5<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
VM          211.9 ms  #############
Cloud Run   636.5 ms  ########################################
Function    641.5 ms  ########################################
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The IaaS VM does not create any additional resources automatically when load increases. All requests are handled by the single e2-micro VM, so its hard limit comes from the CPU and memory capacity of that machine. Cloud Run automatically creates additional container instances as demand rises and can scale back to zero when idle. Its service is capped at 3 instances, and each instance allows up to 80 concurrent requests. The gen-2 Cloud Function also scales automatically by creating Cloud Run-backed instances and can scale to zero when idle.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
The VM required the most manual configuration: creating a custom VPC and subnet, firewall rules, an e2-micro instance, IAM permissions, installing Docker, authenticating to Artifact Registry, pulling the image, and starting the container manually. Cloud Run required building and pushing the container image and then deploying that image with public access and scaling settings. The Cloud Function required the least infrastructure work because the Python source was deployed directly with an HTTP trigger, while Google handled the build, containerization, and the underlying runtime infrastructure.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-988477193196.us-central1.run.app
LAST DEPLOYED BY: service-988477193196@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-17T16:34:03.742400Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-988477193196.us-central1.run.app
LAST DEPLOYED BY: Ali.Hamidovic@eurecom.fr
LAST DEPLOYED AT: 2026-10-17T16:30:15.795462Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows that both echo-bot and echo appear as Cloud Run services. The echo service was deployed by the Cloud Functions service agent, which indicates that the second-generation Cloud Function is ultimately running on Cloud Run infrastructure. This explains why the warm latencies are almost identical: Cloud Run measured 636.5 ms and the Function measured 641.5 ms. Their cold starts are also similar because both may need to start Cloud Run-backed instances after scaling to zero. The main difference is the deployment model: Cloud Run deploys a container image directly, while Cloud Functions accepts source code and Google performs the build and Cloud Run deployment automatically.
<!--/answer-->
