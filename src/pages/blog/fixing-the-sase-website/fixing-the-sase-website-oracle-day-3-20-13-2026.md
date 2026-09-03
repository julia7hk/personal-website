---
title: "Fixing the SASE Website - Oracle Day 3"
date: "2026-02-13"
description: "Automating Oracle Cloud A1 instance creation and setting up API credentials for the SASE Purdue website"
dayNumber: 9
milestone: 3
milestoneTitle: "oracle cloud hosting"
---

[02/13/2026](#february-13-2026):
i cant get the oracle cloud ampere a1, looking into automating the request with a redditor's 24/7 script

[02/14/2026](#february-14-2026):
getting API credentials from sasepurdue oracle cloud account and considering the pay as you go trick

---

# **February 13, 2026**

woah friday the 13th, also i woke up at 7am for some reason

i cant get the oracle cloud ampere a1

### Automate the Request

redditor's 24/7 script to try it

https://www.reddit.com/r/oraclecloud/comments/1mpysur/finally_got_my_oracle_cloud_a1_flex_instance/

> Instead of manually trying to create instances through the web console, I wrote a script that uses **Oracle Resource Manager (ORM) Stacks** to automate the entire process:
>

> **Stack-based deployment** - More reliable than individual instance creation
>

> **Automatic retries** - Runs 24/7 until successful
>

> **Uses existing E2 Micro** - Leverages your current always-free instance as the automation runner
>

# **February 14, 2026**

woah valentines day

1. get API credentials from sasepurdue oracle cloud account

log in → user settings →

tokens and keys →

add API keys

### **Configuration file preview**

**Note:**

This configuration file snippet includes the basic authentication information you'll need to use the SDK, CLI, or other OCI developer tool. Paste the contents of the text box into your ~/.oci/config file and update the key_file parameter with the file path to your private key.

[Learn more](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/sdkconfig.htm)

**Fingerprint**

`<your API key fingerprint>`

```
[DEFAULT]
user=<your user OCID>
fingerprint=<your API key fingerprint>
tenancy=<your tenancy OCID>
region=us-chicago-1
key_file=<path to your private keyfile> # TODO
```

---

### last resort

lowk lets just do the Pay As You Go trick

its just gonna take my credit card a sec 🥀
