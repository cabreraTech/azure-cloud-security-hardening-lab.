# azure-cloud-security-hardening-lab.
Hands-on Azure project: deploy a small cloud estate, deliberately misconfigure it, then detect and fix the misconfigurations the way a cloud security engineer would in a real audit.

## Overview

This project deploys a small Azure environment, deliberately misconfigures it, then hardens it back to a secure state, the same cycle a cloud security engineer runs in a real audit. The goal was hands-on practice with the misconfigurations that cause real breaches (public storage, over-permissioned identities) and the tools used to find and fix them.

**Tools used:** Microsoft Azure, Microsoft Defender for Cloud, Azure Monitor / Log Analytics

## Architecture

A small estate, deliberately simple so the focus stays on configuration, not scale.

- Resource group: `rg-cloud-security-lab`
- Linux virtual machine (`security-lab-vm`), with a public IP, network security group, and network interface
- Storage account (Standard, LRS) with a blob container (`lab-data`) holding a test file
- Log Analytics workspace (`cloudlab-logs`) for diagnostic data

## Breaking It On Purpose

Two real-world misconfigurations were introduced deliberately, to have genuine findings to detect and fix rather than staged ones.

| Misconfiguration | How it was introduced | Why it's a real risk |
|---|---|---|
| Public blob container | Allow Blob anonymous access enabled at the account level, container access level set to Container (anonymous read) | Exposed storage buckets are one of the most common real-world breach causes; this makes the test file readable by anyone with the URL, no login required |
| Over-permissioned identity | System-assigned managed identity enabled on the VM, then assigned the Owner role at the subscription scope | A compromised VM with this identity would give an attacker full control — create, delete, modify permissions — over every resource in the subscription, not just the VM itself |

Both were verified as working before moving to the hardening phase: the blob loaded in a private browser window with no authentication, and the role assignment showed Owner at subscription scope in Access control (IAM).

## Hardening

| Finding | Fix | Principle applied |
|---|---|---|
| Public blob container | Container access level reset to Private (no anonymous access); verified the test file URL no longer loads in a private browser window | Default deny: storage should never be world-readable unless there's a specific, reviewed reason |
| Over-permissioned VM identity | Removed the Owner role assignment at subscription scope; replaced it with Storage Blob Data Reader scoped to only the one storage account the VM actually needs | Least privilege: grant the minimum access a resource needs to do its job |

After the fix, the VM's identity can read blobs in that single storage account and nothing else in the subscription — a direct, verifiable reduction in blast radius if the VM is ever compromised.

## Logging and Encryption

**Encryption at rest:** confirmed enabled by default on the storage account (Microsoft-managed keys).

**Logging:** a Log Analytics workspace (`cloudlab-logs`) was created, then a diagnostic setting (`storage-logs`) was added to the storage account's blob service, forwarding read, write, and transaction logs to that workspace. This closes a real gap: without it, there would be no way to tell afterward whether the exposed container had actually been accessed while it was public.

## Skills Demonstrated

- Deploying and configuring core Azure resources: virtual machines, storage accounts, blob containers, resource groups
- Identifying and reproducing two high-frequency real-world cloud misconfigurations: public object storage and over-permissioned managed identities
- Applying least-privilege access control: replacing broad, standing Owner access with a narrowly scoped role
- Configuring diagnostic logging and forwarding it to Log Analytics for auditability
- Verifying fixes rather than assuming them
- Working directly in the Azure Portal across IAM, storage configuration, and monitoring
