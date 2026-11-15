# S3 – AI analysis and reasoning

<!-- Written by the AWS TnC Change Monitor. Rewritten on every run - do not edit by hand. -->

| | |
|---|---|
| Service | S3 |
| Last major assessment | 2026-05-18 (Last Major Assessment Date in s3.md) |
| Assessment file | s3.md |
| Kit decision | 5A · Perform (partial) re-assessment |
| AI risk score | 78 / 100 (High) |
| Kit version | 2.2.1 |
| Run | 2026-11-15 22:35 | <!-- tnc:volatile -->

## The run in numbers

| | |
|---|---|
| Model | OpenAI gpt-5.6-terra |
| Prompt version | 1.14-scenario-checklist-1 |
| Changes found by rules | 72 |
| Changes sent to the AI | 68 |
| Answers reused from earlier runs (same change, evidence, assessment and prompt) | 0 |
| Removed by the triage model | 0 |
| Changes asked 3× for the consistency vote | 29 |
| Changes asked again after a failed or incomplete answer / still without answer | 0 / 0 |
| AI calls repeated after a temporary fault (rate limit, server error, timeout) | 0 | <!-- tnc:volatile -->
| High findings independently reviewed / disputed | 28 / 25 |
| AI calls | 17 |
| Tokens in / out | 168038 / 52459 | <!-- tnc:volatile -->
| AI time | 440 s | <!-- tnc:volatile -->
| Calls by step | independent review: 2, service judgment: 1, service summary: 1, validate attempt 1: 9, validate attempt 2: 2, validate attempt 3: 2 | <!-- tnc:volatile -->

Every prompt and answer of this run is stored word for word in the audit log `data/ai_audit/s3_2026-11-15_223537.jsonl` on the machine that ran the kit. <!-- tnc:volatile -->

## How the AI reaches a finding

The model does **not** pick a severity itself. It answers fixed questions and must quote its evidence; the kit then checks the quotes and applies the company's severity table. Step by step:

1. **Changes found by rules.** The kit compares AWS's IAM reference and API models on the last major assessment date with today (see Changes.md). The AI never invents changes; it only judges these.
2. **What the model is given per change:** the change itself (type, name, details, actions it applies to), a passage from the official AWS documentation found through the AWS Knowledge MCP server (when one exists), the most relevant sections of this service's assessment file (chosen by keyword search, BM25), the TnC framework scenarios XS0–XS6, the current IAM reference data (so proposed policies only use real actions/keys) and the AWS Security Hub controls already available.
3. **Seven checklist questions,** each tied to one TnC scenario. For every YES the model must copy an exact quote from the evidence:

   - `network_exposure` → **XS0** (High): Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restriction option?
   - `data_sharing` → **XS1** (High): Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, replication, cross-account access)?
   - `iam_permissions` → **XS5** (High): Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-management action or authorization option?
   - `encryption_at_rest` → **XS2** (Medium): Does the change affect encryption at rest or the choice of encryption keys?
   - `encryption_in_transit` → **XS3** (Medium): Does the change affect encryption or TLS for data in transit?
   - `deletion_recovery` → **XS4** (Low): Does the change affect deletion, retention, backup, versioning or recovery of data?
   - `logging_audit` → **XS6** (Low): Does the change affect logging, monitoring, auditing or traceability?

4. **Quote check by the kit (not by AI).** A YES counts only if its quote is really in the change details or the AWS documentation passage. A YES without a matching quote does not count and is flagged.
5. **Severity from the company table:** XS0 = High, XS1 = High, XS2 = Medium, XS3 = Medium, XS4 = Low, XS5 = High, XS6 = Low. A change gets the highest level among its verified scenarios. No verified scenario: *No security risk* if the model said not security-relevant, otherwise *Uncertain* (a person decides).
6. **Consistency vote.** Every possible High or Medium finding is asked 3 times in total (attempts 2+ use a short checklist-only prompt at a higher temperature). A scenario counts only if a majority of the attempts verified it. Disagreement is flagged for review.
7. **Independent review.** Every High finding goes to a separate review prompt that tries to disprove it. If the reviewer does not uphold a High scenario, the finding is kept but marked *needs review*.
8. **Coverage.** The model says whether the existing assessment already covers the risk and must quote the assessment. *Covered* without a matching quote is downgraded to *Unknown*.
9. **Guardrails.** The model proposes protect (SCP / IAM deny) and detect (Prisma / Config) guardrails; the kit checks that every action and condition key in them exists in AWS's IAM reference.
10. **Decision.** *Perform re-assessment* when a material (Medium or higher) mapped risk is not fully covered, or a change removes something the assessment relies on. *Manual review* when findings are uncertain, disputed or not checked, or evidence is missing. The more conservative of the rule-based and the AI recommendation is used.
11. **Service judgment and ranking.** Finally the model, acting as a senior cyber security risk assessor, scores the whole service 0–100 from the verified findings, and compares all services of a batch for the after-AI ranking (it may move a score by at most 15 points).

## Results

| Outcome | Severity | Changes |
|---|---|---:|
| Risk mapped to the TnC framework | High | 28 |
| Risk mapped to the TnC framework | Medium | 1 |
| Risk mapped to the TnC framework | Low | 14 |
| Uncertain – needs a human decision | – | 4 |
| No security risk | – | 21 |

**Decision:** 5A · Perform (partial) re-assessment – rule-based check: `PERFORM_ASSESSMENT`, AI recommendation: `PERFORM_ASSESSMENT`.

- 4 material TnC-mapped risk(s) not fully covered by the existing assessment (e.g. FEAT-0004, FEAT-0020, FEAT-0021, FEAT-0023).
- 25 uncertain/unmapped risk(s) need a human decision.
- 14 change(s) affect only XS4/XS6 (14 by AI mapping): marked Low priority - recommended guardrails are listed as backlog; they do not require a re-assessment on their own.

**AI service-level recommendation:** A partial reassessment is required because multiple material changes mapped to XS0, XS1, XS2, XS3, and XS5 are not covered or only partially covered by the existing assessment. These include new S3 annotation access and deletion permissions, altered Access Grants authorization behavior, new log-delivery and cross-account aggregation paths, and new network access paths. New condition keys provide additional preventive control options, but their required request actions, approved values, and deployment coverage must be validated before relying on them as mitigations. XS4- and XS6-only changes are recorded as low priority and do not independently drive this decision.

## Service judgment (senior cyber security risk assessor)

| | |
|---|---|
| Risk score | **78 / 100** (High) |
| Re-assessment | PERFORM_ASSESSMENT |
| Headline | New S3 authorization and cross-account log-delivery paths are outside the current assessment scope. |
| TnC scenarios affected | XS0, XS1, XS2, XS3, XS4, XS5, XS6 |
| Gap in the existing assessment | PARTIALLY_COVERED |
| Confidence | 0.84 |
| Reasoning | Confirmed changes introduce or extend authorization models not covered by the assessment: Access Grants directory/IAM mappings and IAM-based authorization for S3 Tables. Server-access-log delivery to CloudWatch Logs enables cross-account aggregation and creates a new log destination requiring destination, KMS, and monitoring controls. The current assessment explicitly excludes XS5 and has incomplete preventive controls for sharing. Annotation and Object Lock changes are mostly unconfirmed or recovery-focused, but require targeted entitlement and SCP review. |
| Condition keys | Nine new keys enable finer log-delivery, annotation, and Object Lock guardrails; validate SCP and IAM policy use. |
| IAM actions | New annotation actions and Access Grants authorization changes require entitlement review; several security mappings remain unconfirmed. |
| New features | Access Grants, S3 Tables IAM authorization, and CloudWatch Logs delivery add material authorization and log-destination considerations. |

Key drivers:

- **Amazon S3 Access Grants are now available in the AWS European Sovereign Cloud (Germany) Region** – Access Grants introduces directory-to-S3 dataset authorization in a newly supported region; XS5 is out of scope.
- **Simplified permissions for Amazon S3 Tables and Iceberg materialized views are now available in AWS GovCloud (US) Regions** – IAM authorization now spans S3 Tables storage, catalog, and query permissions; not assessed.
- **Amazon S3 server access logs now deliver to Amazon CloudWatch Logs and Amazon S3 Tables** – Cross-account CloudWatch Logs aggregation can expose access logs and requires controlled encrypted destinations.
- **Amazon S3 server access log delivery to Amazon CloudWatch Logs now available in five additional AWS Regions** – Expanded log-delivery path needs KMS, destination-policy, and traceability control validation.
- **s3:deliverySourceArn** – New vended-log-delivery source restriction needs policy review; validation remains unconfirmed.

## Reasoning per change (68)

Ordered High → Medium → Low → uncertain → no risk. ✔ = the quote was found in the evidence by the kit.

### ACT-0001 · s3: 25 actions – High

*IAM action · Modified · released after 2026-05-18 (exact date not published) · source [link](https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json)*

**Change:** Access level Permissions management -> Permissions management, Write - applies to 25 actions: AssociateAccessGrantsIdentityCenter, BypassGovernanceRetention, CreateAccessGrantsInstance, DeleteAccessGrant, DeleteAccessGrantsInstance, DeleteAccessGrantsInstanceResourcePolicy, DeleteAccessGrantsLocation, DeleteAccessPointPolicy, DeleteAccessPointPolicyForObjectLambda, DeleteBucketPolicy, DissociateA…
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"Api-S3control-2018-08-20","context":"### A…](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-s3control-2018-08-20.html) – “{"content":{"result":"Table of Contents:\n- AWS S3 Control 2018-08-20 (char 6-447225)\n - Operation Summary (char 416-15693)\n - Paginators (char 15694-16407)\n - Operations (char 16408-299556)\n - AssociateAccessGrantsIdentityCenter (char 16423-18299)\n - Parameter Syntax (char 17305-17492)\n - Pa…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Access level Permissions management -> Permissions management, Write - applies to 25 actions |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Reclassification of 25 S3 actions adds Write to permission-management actions, requiring IAM entitlement review to avoid excessive administrative access.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The quote only reports a classification change affecting 25 actions. It identifies no action, new permission, principal, or authorization capability.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence identifies an access-level change but does not describe changed API authorization behavior.
- **Needs a person because:** independent review disputed High: The quote only reports a classification change affecting 25 actions. It identifies no action, new permission, principal, or authorization capability.

### ACT-0002 · s3:PutObject – High

*IAM action · Modified · released 2026-09-30 · source [link](https://github.com/iann0036/iam-dataset/commit/c62f167d8caf6ec52dac3cd6448cd68c214483d2)*

**Change:** Condition keys now supported: s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:x-amz-object-annotation-directive
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"How do I resolve the 403 Access Denied err…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3/aws.sdk.kotlin.services.s3/-s3-client/put-object.html) – “- When you enable versioning for a bucket, if Amazon S3 receives multiple write requests for the same object simultaneously, it stores all versions of the objects. For each write request that is made to the same object, Amazon S3 automatically generates a unique version ID of that object being stor…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 2/3 | Condition keys now supported: s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:x-amz-object-annotation-directive |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | UNKNOWN | 1/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Administrators can newly condition PutObject authorization using the added request attributes, changing the S3 write authorization model.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, Uncertain, High (agreement 67 %).
- **Independent review:** NOT upheld – PutObject support for additional condition keys enables policy restrictions on existing writes; it does not add a permission grant or new identity path.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not define the added object-lock or annotation-directive semantics.
- **Needs a person because:** independent review disputed High: PutObject support for additional condition keys enables policy restrictions on existing writes; it does not add a permission grant or new identity path.

### ACT-0003 · s3:DeleteObjectAnnotation – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+2 more).
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"deleteObjectAnnotation","context":"# delet…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3/aws.sdk.kotlin.services.s3/-s3-client/delete-object-annotation.html) – “{"content":{"result":"s3/aws.sdk.kotlin.services.s3/S3Client/deleteObjectAnnotation\n\n# deleteObjectAnnotation\n\nabstract suspend fun deleteObjectAnnotation(input: DeleteObjectAnnotationRequest): DeleteObjectAnnotationResponse\n\nDeletes a specific annotation from an Amazon S3 object. Use the `x-…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 2/3 | s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | To use this operation, you must have the `s3:DeleteObjectAnnotation` permission. |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 2/3 | s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:a… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 3/3 | Deleting an annotation is permanent. Annotations are not independently versioned, so there is no delete marker or way to recover a deleted annotation. |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3, XS4, XS5; severity High (TnC scenario table).
- **Risk (AI):** An identity granted DeleteObjectAnnotation can permanently delete object annotations, with no independent annotation recovery.
- **Existing assessment:** Partly covered – “An identity has more privileges than required for its operations, or has access to information not required for those operations.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The action deletes object annotations only. AccessPointNetworkOrigin is merely a condition key; no new network exposure or IAM model change is evidenced.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** independent review disputed High: The action deletes object annotations only. AccessPointNetworkOrigin is merely a condition key; no new network exposure or IAM model change is evidenced.

### ACT-0004 · s3:DeleteObjectVersionAnnotation – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+3 more).
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 2/3 | s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | New action (Write). Resources: accesspointobject, object. |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 2/3 | s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:a… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | UNKNOWN | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3, XS5; severity High (TnC scenario table).
- **Risk (AI):** Identities granted the new write action can perform its undocumented operation on object or access-point-object resources.
- **Existing assessment:** Partly covered – “An identity has more privileges than required for its operations, or has access to information not required for those operations.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – A write action on annotations does not itself create network exposure or alter identity authorization. Listed condition keys are policy constraints, not exposure capabilities.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Documentation does not describe deletion, recovery, or audit effects.
- **Needs a person because:** independent review disputed High: A write action on annotations does not itself create network exposure or alter identity authorization. Listed condition keys are policy constraints, not exposure capabilities.

### ACT-0011 · s3:CreateAccessGrant – High

*IAM action · Modified · released 2026-07-15 · source [link](https://awsiamchanges.com/changes/1784116800-s3.html)*

**Change:** Access level Permissions management -> Permissions management, Write; Resource types added: accessgrant; Resource types removed: accessgrantslocation
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"CreateAccessGrant","context":"# CreateAcce…](https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_CreateAccessGrant.html) – “{"content":{"result":"# CreateAccessGrant\n\nCreates an access grant that gives a grantee access to your S3 data. The grantee can be an IAM user or role or a directory user, or group. Before you can create a grant, you must have an S3 Access Grants instance in the same Region as the S3 data. You ca…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Access level Permissions management -> Permissions management, Write; Resource types added: accessgrant; Resource types removed: accessgrantslocation |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Principals authorized for CreateAccessGrant have an altered resource-authorization model for creating grants to S3 data.
- **Existing assessment:** Not covered – “XS5 is explicitly marked not in scope in the source.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – CreateAccessGrant inherently grants access, but the cited modification only changes resource types. Evidence does not show newly broadened grantees, sharing paths, or weakened authorization.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No configuration setting is supplied to detect IAM resource-type authorization semantics.
- **Needs a person because:** independent review disputed High: CreateAccessGrant inherently grants access, but the cited modification only changes resource types. Evidence does not show newly broadened grantees, sharing paths, or weakened authorization.

### ACT-0012 · s3:CreateAccessGrantsLocation – High

*IAM action · Modified · released 2026-06-11 · source [link](https://awsiamchanges.com/changes/1784116800-s3.html)*

**Change:** Access level Permissions management -> Permissions management, Write; Resource types added: accessgrantslocation; Resource types removed: accessgrantsinstance
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"createAccessGrantsLocation","context":"##…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3control/aws.sdk.kotlin.services.s3control/create-access-grants-location.html) – “cation(crossinline block: CreateAccessGrantsLocationRequest.Builder.() -> Unit): CreateAccessGrantsLocationResponse\n\nThe S3 data location that you would like to register in your S3 Access Grants instance. Your S3 data must be in the same Region as your S3 Access Grants instance. The location can…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Access level Permissions management -> Permissions management, Write; Resource types added: accessgrantslocation; Resource types removed: accessgrantsinstance |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Principals authorized to register Access Grants locations have an altered resource-authorization model for an IAM-role-based access-management operation.
- **Existing assessment:** Not covered – “XS5 is explicitly marked not in scope in the source.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The resource-type modification does not demonstrate changed role assumption or PassRole behavior. No new or weakened IAM authorization capability is evidenced.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No configuration setting is supplied to detect IAM resource-type authorization semantics.
- **Needs a person because:** independent review disputed High: The resource-type modification does not demonstrate changed role assumption or PassRole behavior. No new or weakened IAM authorization capability is evidenced.

### ACT-0013 · s3:GetObjectAnnotation – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Read). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+1 more).
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"GetObjectAnnotation","context":"# GetObjec…](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObjectAnnotation.html) – “{"content":{"result":"# GetObjectAnnotation\n\nRetrieves an annotation from an Amazon S3 object. To use this operation, you must have the\n`s3:GetObjectAnnotation` permission.\n\nIf checksum mode is enabled via the `x-amz-checksum-mode` header, Amazon S3\nreturns the stored checksum in the response…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | 0/3 | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:Data… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3; severity High (TnC scenario table).
- **Risk (AI):** Annotation reads can require network-origin and TLS conditions; absent equivalent authorization controls, annotations could be read outside approved access paths.
- **Existing assessment:** Partly covered – “Deny requests that do not satisfy the restricted-access conditions; deny wildcard `*` access to resource and principal.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – GetObjectAnnotation retrieves data only when explicitly authorized. AccessPointNetworkOrigin as a condition key does not establish untrusted-network exposure.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** independent review disputed High: GetObjectAnnotation retrieves data only when explicitly authorized. AccessPointNetworkOrigin as a condition key does not establish untrusted-network exposure.

### ACT-0014 · s3:GetObjectVersionAnnotation – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Read). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+2 more).
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"AmazonEC2RoleforAWSCodeDeployLimited","con…](https://docs.aws.amazon.com/AmazonS3/latest/userguide/batch-ops-iam-role-policies.html#batch-ops-iam-role-policies-perm) – “{"content":{"result":"Table of Contents:\n- Granting permissions for Batch Operations (char 0-17399)\n - Creating an S3 Batch Operations IAM role (char 727-3676)\n - Trust policy (char 3274-3676)\n - Using JSON (char 3415-3431)\n - Using (char 3432-3676)\n - Attaching permissions policies (char 367…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 2/3 | New action (Read). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey} |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:Data… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3, XS5; severity High (TnC scenario table).
- **Risk (AI):** Version-annotation reads can require network-origin and TLS conditions; absent equivalent authorization controls, version annotations could be read outside approved access paths.
- **Existing assessment:** Partly covered – “Deny requests that do not satisfy the restricted-access conditions; deny wildcard `*` access to resource and principal.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – This is a read permission for version annotations. Resource types and condition keys do not evidence public, cross-account, network, or IAM-model expansion.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Documentation excerpt does not name this action.
- **Needs a person because:** independent review disputed High: This is a read permission for version annotations. Resource types and condition keys do not evidence public, cross-account, network, or IAM-model expansion.

### ACT-0015 · s3:GetObjectVersionAnnotationForReplication – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Read). Resources: object. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion, s3:x-amz-content-sha256.
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"How do I troubleshoot cross-account Amazon…](https://docs.aws.amazon.com/AmazonS3/latest/userguide/setting-repl-config-perm-overview.html#setting-repl-config-crossacct) – “plicate\n objects or delete markers to the destination bucket. For information about delete\n markers, see How delete operations affect replication.\n\n **Note:**\n\n Permissions for the `s3:ReplicateObject` action on the\n `amzn-s3-demo-destination-bucket` bucket also allow replication of\n metada…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | YES ✘ quote not found | 0/3 | Permissions for this action on objects in the source bucket allow Amazon S3 to read annotations for replication. |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 2/3 | New action (Read). Resources: object. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey} |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion, s3:x-a… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS3, XS5; severity High (TnC scenario table).
- **Risk (AI):** Replication roles can read source-object annotations; unrestricted replication permissions could copy annotation data through unapproved replication paths.
- **Existing assessment:** Partly covered – “Ensure replication rules use a destination account that belongs to the SocGen organization.”
- **Consistency vote:** 3 attempts gave Medium, High, High (agreement 67 %).
- **Independent review:** NOT upheld – The action permits S3 replication to read annotations from an authorized source bucket. It does not create a new principal, trust relationship, or sharing mechanism.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** quote not found in the evidence for: data_sharing (XS1); possible High not backed by evidence; independent review disputed High: The action permits S3 replication to read annotations from an authorized source bucket. It does not create a new principal, trust relationship, or sharing mechanism.

### ACT-0016 · s3:ListObjectAnnotations – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (List). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:annotation-prefix, s3:authType, s3:max-annotation-results (+3 more).
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"listObjectAnnotations","context":"# listOb…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3/aws.sdk.kotlin.services.s3/-s3-client/list-object-annotations.html) – “{"content":{"result":"s3/aws.sdk.kotlin.services.s3/S3Client/listObjectAnnotations\n\n# listObjectAnnotations\n\nabstract suspend fun listObjectAnnotations(input: ListObjectAnnotationsRequest): ListObjectAnnotationsResponse\n\nLists the annotations attached to an Amazon S3 object. Results are pagin…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 2/3 | To use this operation, you must have the `s3:ListObjectAnnotations` permission. |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 3/3 | Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:Data… |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3, XS5; severity High (TnC scenario table).
- **Risk (AI):** Annotation listings can require network-origin and TLS conditions; absent equivalent authorization controls, object annotations could be enumerated outside approved access paths.
- **Existing assessment:** Partly covered – “Deny requests that do not satisfy the restricted-access conditions; deny wildcard `*` access to resource and principal.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – Listing annotations requires explicit permission. The cited access-point condition keys do not show untrusted-network access or an IAM authorization-model change.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** independent review disputed High: Listing annotations requires explicit permission. The cited access-point condition keys do not show untrusted-network access or an IAM authorization-model change.

### ACT-0017 · s3:ListObjectVersionAnnotations – High

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (List). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:annotation-prefix, s3:authType, s3:max-annotation-results (+4 more).
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | New action (List). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s… |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 2/3 | New action (List). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey} |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | YES ✔ | 3/3 | s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:annotation-prefix, s3:authType, s3:max-annotation-results |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0, XS3, XS5; severity High (TnC scenario table).
- **Risk (AI):** Version-annotation listings can require network-origin and TLS conditions; absent equivalent authorization controls, version annotations could be enumerated outside approved access paths.
- **Existing assessment:** Partly covered – “Deny requests that do not satisfy the restricted-access conditions; deny wildcard `*` access to resource and principal.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – A list action for version annotations remains subject to IAM authorization. Condition keys constrain policies and do not themselves create exposure or privilege escalation.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No AWS documentation was supplied for this action.
- **Needs a person because:** independent review disputed High: A list action for version annotations remains subject to IAM authorization. Condition keys constrain policies and do not themselves create exposure or privilege escalation.

### ACT-0018 · s3:PutObjectRetention – High

*IAM action · Modified · released 2026-09-08 · source [link](https://github.com/iann0036/iam-dataset/commit/c62f167d8caf6ec52dac3cd6448cd68c214483d2)*

**Change:** Condition keys now supported: s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | s3:DataAccessPointArn, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:object-lock-mode |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 3/3 | s3:DataAccessPointArn, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:object-lock-mode |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4, XS5; severity High (TnC scenario table).
- **Risk (AI):** Policy authors can condition object-retention updates on Object Lock event-hold attributes, improving control of retention changes that could otherwise affect recoverability.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – Added Object Lock event-hold condition keys provide policy scoping. Evidence does not show broadened permissions, new principals, or weakened IAM controls.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No AWS documentation was supplied for the new condition keys.
- **Needs a person because:** independent review disputed High: Added Object Lock event-hold condition keys provide policy scoping. Evidence does not show broadened permissions, new principals, or weakened IAM controls.

### ACT-0019 · s3:ReplicateObject – High

*IAM action · Modified · released 2026-09-09 · source [link](https://github.com/iann0036/iam-dataset/commit/c62f167d8caf6ec52dac3cd6448cd68c214483d2)*

**Change:** Condition keys now supported: s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"How can I set up cross-account replication…](https://repost.aws/knowledge-center/s3-cross-account-replication-object-lock) – “e\" \n } \n ] \n}\n```\n\nAttach the following permissions to the IAM role:\n\n```plaintext\n \n{ \"Version\": \"2012-10-17\", \n \"Statement\": [ \n { \n \"Sid\": \"SourceBucketPermissions\", \n \"Effect\": \"Allow\", \n \"Action\": [ \n \"s3:GetObjectRetention\", \n \"s3:GetObjectVersionTagging\"…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:signatureAge |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 3/3 | s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:object-lock-event-hold, s3:object-lock-event-hold-duration-days, s3:signatureAge |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4, XS5; severity High (TnC scenario table).
- **Risk (AI):** Policy authors can condition object replication on Object Lock event-hold attributes, enabling retention-aware controls over replicated objects and their recoverability.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The evidence only identifies additional condition keys for an existing replication action. It does not demonstrate altered replication trust, destination sharing, or IAM escalation.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Documentation does not describe the new condition-key semantics.
- **Needs a person because:** independent review disputed High: The evidence only identifies additional condition keys for an existing replication action. It does not demonstrate altered replication trust, destination sharing, or IAM escalation.

### ACT-0020 · s3:UpdateObjectEncryption – High

*IAM action · Modified · released 2026-05-23 · source [link](https://github.com/iann0036/iam-dataset/commit/9b3ecf9887de7760cea26a0d2c4c14ceb6baaa98)*

**Change:** Condition keys now supported: s3:ExistingObjectTag/<key>
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"Update object encryption","context":"## Re…](https://docs.aws.amazon.com/AmazonS3/latest/userguide/batch-ops-update-encryption.html) – “perform the `UpdateObjectEncryption` operation, add the following AWS Identity and Access Management (IAM)\npolicy to your IAM principal (user, role, or group). To use this policy, replace\n`amzn-s3-demo-bucket` with the name of the bucket that\ncontains the objects that you want to update encrypti…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion, s3:x-amz-content-sha256 |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | YES ✘ quote not found | 1/3 | perform the `UpdateObjectEncryption` operation, add the following AWS Identity and Access Management (IAM) policy to your IAM principal |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Policy authors can use existing object tags to constrain object-encryption updates, strengthening authorization over encryption-at-rest changes for classified objects.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The cited material describes required permissions and condition keys for encryption updates. It does not evidence weakened encryption, broadened access, or IAM-model change.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** quote not found in the evidence for: encryption_at_rest (XS2); independent review disputed High: The cited material describes required permissions and condition keys for encryption updates. It does not evidence weakened encryption, broadened access, or IAM-model change.

### CK-0001 · s3:deliverySourceArn – High

*Condition key · Added · released 2026-06-04 · source [link](https://github.com/iann0036/iam-dataset/commit/ae377b996fada60bba62ed5d5e80edfa848addfb)*

**Change:** New condition key (type ARN). Usable with: AllowVendedLogDeliveryForResource Filters access by specific delivery source Amazon Resource Name (ARN)
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Filters access by specific delivery source Amazon Resource Name (ARN) |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | 3/3 | New condition key (type ARN). Usable with: AllowVendedLogDeliveryForResource Filters access |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5, XS6; severity High (TnC scenario table).
- **Risk (AI):** An authorized principal could allow vended log delivery without a source-delivery restriction, weakening authorization control and traceability scope.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The key restricts vended-log delivery authorization to a specified source ARN; it does not itself grant permissions or introduce a new principal.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration evidence can evaluate request condition-key use.
- **Needs a person because:** independent review disputed High: The key restricts vended-log delivery authorization to a specified source ARN; it does not itself grant permissions or introduce a new principal.

### CK-0002 · s3:logType – High

*Condition key · Added · released 2026-06-04 · source [link](https://github.com/iann0036/iam-dataset/commit/ae377b996fada60bba62ed5d5e80edfa848addfb)*

**Change:** New condition key (type String). Usable with: AllowVendedLogDeliveryForResource Filters access by specific log type, currently supports S3_SERVER_ACCESS_LOGS
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Filters access by specific log type, currently supports S3_SERVER_ACCESS_LOGS |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | 3/3 | Filters access by specific log type, currently supports S3_SERVER_ACCESS_LOGS |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5, XS6; severity High (TnC scenario table).
- **Risk (AI):** An authorized principal could allow log delivery without restricting log type, weakening authorization control over delivered audit-log types.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The key narrows authorization by log type for an existing delivery action. Evidence does not show a new identity, permission grant, or access path.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration evidence can evaluate request condition-key use.
- **Needs a person because:** independent review disputed High: The key narrows authorization by log type for an existing delivery action. Evidence does not show a new identity, permission grant, or access path.

### CK-0003 · s3:object-lock-event-hold – High

*Condition key · Added · released 2026-09-08 · source [link](https://github.com/iann0036/iam-dataset/commit/c62f167d8caf6ec52dac3cd6448cd68c214483d2)*

**Change:** New condition key (type String). Usable with: PutObject, PutObjectRetention, ReplicateObject Filters access by object event hold status (ON or OFF)
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Usable with: PutObject, PutObjectRetention, ReplicateObject Filters access by object event hold status |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 3/3 | Filters access by object event hold status (ON or OFF) |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4, XS5; severity High (TnC scenario table).
- **Risk (AI):** An authorized writer could submit object operations without an event-hold restriction, reducing enforceable object-retention protection against loss or unauthorized change.
- **Existing assessment:** Partly covered – “Deny S3 buckets without Object Lock to provide data immutability and prevent unauthorized or accidental deletion.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – This is a request-attribute condition for existing object operations. It can constrain policies but does not create or alter an authorization model.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** Object Lock detection does not evidence request-level event-hold enforcement.
- **Needs a person because:** independent review disputed High: This is a request-attribute condition for existing object operations. It can constrain policies but does not create or alter an authorization model.

### CK-0004 · s3:object-lock-event-hold-duration-days – High

*Condition key · Added · released 2026-09-08 · source [link](https://github.com/iann0036/iam-dataset/commit/c62f167d8caf6ec52dac3cd6448cd68c214483d2)*

**Change:** New condition key (type Numeric). Usable with: PutObject, PutObjectRetention, ReplicateObject Filters access by object event hold duration in days
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Usable with: PutObject, PutObjectRetention, ReplicateObject Filters access by object event hold duration |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 3/3 | Filters access by object event hold duration in days |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4, XS5; severity High (TnC scenario table).
- **Risk (AI):** An authorized writer could perform object operations without supplying an event-hold duration, reducing enforceable retention safeguards against object loss.
- **Existing assessment:** Partly covered – “Deny S3 buckets without Object Lock to provide data immutability and prevent unauthorized or accidental deletion.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The duration condition key limits eligible object-lock requests under existing permissions. It does not independently grant access or enable new principals.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No approved minimum duration is supplied for a value-based denial.
- **Needs a person because:** independent review disputed High: The duration condition key limits eligible object-lock requests under existing permissions. It does not independently grant access or enable new principals.

### CK-0005 · s3:resourceArnBeingAuthorized – High

*Condition key · Added · released 2026-06-04 · source [link](https://github.com/iann0036/iam-dataset/commit/ae377b996fada60bba62ed5d5e80edfa848addfb)*

**Change:** New condition key (type ARN). Usable with: AllowVendedLogDeliveryForResource Filters access by source bucket Amazon Resource Name (ARN)
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Filters access by source bucket Amazon Resource Name (ARN) |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | 2/3 | New condition key (type ARN). Usable with: AllowVendedLogDeliveryForResource Filters access |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5, XS6; severity High (TnC scenario table).
- **Risk (AI):** An authorized principal could allow vended log delivery without a source-bucket authorization restriction, expanding the scope of authorized resource access.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The key filters an existing vended-log delivery authorization by source bucket ARN. No new sharing, principal, or permission capability is evidenced.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration evidence can evaluate request condition-key use.
- **Needs a person because:** independent review disputed High: The key filters an existing vended-log delivery authorization by source bucket ARN. No new sharing, principal, or permission capability is evidenced.

### CK-0006 · s3:x-amz-object-annotation-directive – High

*Condition key · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New condition key (type String). Usable with: PutObject Filters access by the annotation copy directive specified in the request
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Usable with: PutObject Filters access by the annotation copy directive specified in the request |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** An authorized PutObject caller could use annotation-copy directives without policy restriction, expanding request-level authorization behavior for object annotations.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – This condition key filters PutObject requests by annotation directive. It is policy-scoping functionality, not a new permission or identity path.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not identify approved annotation-directive values or a data-sharing impact.
- **Needs a person because:** independent review disputed High: This condition key filters PutObject requests by annotation directive. It is policy-scoping functionality, not a new permission or identity path.

### CK-0007 · s3:x-amz-object-if-match – High

*Condition key · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New condition key (type String). Usable with: DeleteObjectAnnotation, DeleteObjectVersionAnnotation, PutObjectAnnotation, PutObjectVersionAnnotation Filters access by the ETag of the object version specified in the request
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Filters access by the ETag of the object version specified in the request |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | 1/3 | Usable with: DeleteObjectAnnotation, DeleteObjectVersionAnnotation, PutObjectAnnotation, PutObjectVersionAnnotation Filters access |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** An authorized caller could alter or delete object annotations without ETag-based conditional authorization, increasing risk of unintended concurrent object-version changes.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – ETag-based filtering constrains annotation operations already authorized. The evidence does not establish broadened IAM permissions or a changed authorization model.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration evidence can evaluate request ETag enforcement.
- **Needs a person because:** independent review disputed High: ETag-based filtering constrains annotation operations already authorized. The evidence does not establish broadened IAM permissions or a changed authorization model.

### CK-0008 · s3:annotation-prefix – High

*Condition key · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New condition key (type String). Usable with: ListObjectAnnotations, ListObjectVersionAnnotations Filters access by the annotation name prefix specified in the request
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | New condition key (type String). Usable with: ListObjectAnnotations, ListObjectVersionAnnotations Filters access by the annotation name prefix specified in the… |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Policy administrators can restrict annotation-listing requests by annotation prefix, altering authorization for annotation metadata access.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The annotation-prefix key constrains list requests under existing permissions. No evidence shows that it grants access or changes IAM authorization.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration is supplied that records request condition-key use.
- **Needs a person because:** independent review disputed High: The annotation-prefix key constrains list requests under existing permissions. No evidence shows that it grants access or changes IAM authorization.

### CK-0009 · s3:max-annotation-results – High

*Condition key · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New condition key (type Numeric). Usable with: ListObjectAnnotations, ListObjectVersionAnnotations Filters access by the maximum number of annotation results requested
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | New condition key (type Numeric). Usable with: ListObjectAnnotations, ListObjectVersionAnnotations Filters access by the maximum number of annotation results r… |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Policy administrators can restrict annotation-listing result volumes, altering authorization for annotation metadata enumeration.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – The maximum-results key limits list response size in policies. It does not create a new permission, principal relationship, or access route.
- **Guardrails possible:** SCP: YES (check: VALID); IAM deny: YES (check: VALID); Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration is supplied that records request condition-key use.
- **Needs a person because:** independent review disputed High: The maximum-results key limits list response size in policies. It does not create a new permission, principal relationship, or access route.

### FEAT-0004 · Amazon S3 Access Grants are now available in the AWS European Sovereign Cloud (Germany) Region – High

*New feature · AWS launch (What's New) · released 2026-06-09 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-access-grants-european-sovereign-cloud-germany-region)*

**Change:** You can now create Amazon S3 Access Grants in the AWS European Sovereign Cloud (Germany) Region. Amazon S3 Access Grants map identities in directories such as Microsoft Entra ID, or AWS Identity and Access Management (IAM) principals, to datasets in S3. This helps you manage data permissions at scale by automatically granting S3 access to end users based on their corporate identity. Visit the AWS…
**AWS documentation given to the AI:** [Amazon S3 Access Grants are now available in the AWS European Sovereign Cloud (Germany) R…](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-access-grants-european-sovereign-cloud-germany-region) – “Amazon S3 Access Grants are now available in the AWS European Sovereign Cloud (Germany) Region. You can now create Amazon S3 Access Grants in the AWS European Sovereign Cloud (Germany) Region. Amazon S3 Access Grants map identities in directories such as Microsoft Entra ID, or AWS Identity and Acce…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | Amazon S3 Access Grants map identities in directories such as Microsoft Entra ID, or AWS Identity and Access Management (IAM) principals, to datasets in S3. |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** Users in the European Sovereign Cloud can map directory or IAM identities to S3 datasets, introducing the Access Grants authorization model in that Region.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** upheld (XS5) – Availability in the Germany Sovereign Cloud newly enables Access Grants there, including directory-identity-to-S3-dataset authorization and automatically granted end-user access.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not provide Access Grants resource configuration or API details.

### FEAT-0006 · Amazon S3 Access Points for Amazon FSx are now available in additional AWS Regions – High

*New feature · AWS launch (What's New) · released 2026-11-09 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/11/s3-access-points-fsx-regions/)*

**Change:** You can now attach Amazon S3 Access Points to your Amazon FSx for NetApp ONTAP file systems in 4 additional AWS Regions, and to your Amazon FSx for OpenZFS file systems in 20 additional AWS Regions. With this capability, your file data in Amazon FSx is accessible for use with the broad range of artificial intelligence, machine learning, and analytics services and applications that work with S3, w…
**AWS documentation given to the AI:** [Amazon S3 Access Points for Amazon FSx are now available in additional AWS Regions](https://aws.amazon.com/about-aws/whats-new/2026/11/s3-access-points-fsx-regions/) – “Amazon S3 Access Points for Amazon FSx are now available in additional AWS Regions. You can now attach Amazon S3 Access Points to your Amazon FSx for NetApp ONTAP file systems in 4 additional AWS Regions, and to your Amazon FSx for OpenZFS file systems in 20 additional AWS Regions. With this capabi…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | An S3 Access Point for Amazon FSx is an S3 endpoint that gives S3-based applications |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | 0/3 | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0; severity High (TnC scenario table).
- **Risk (AI):** FSx data becomes reachable through new S3 endpoints in additional Regions, expanding access paths requiring network-access review.
- **Existing assessment:** Partly covered – “Deny requests that do not satisfy the restricted-access conditions; deny wildcard `*` access to resource and principal.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – Regional availability of FSx access points adds no evidenced public or untrusted-network exposure. The quote does not describe internet access or weakened endpoint controls.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not identify an FSx access-point network-origin configuration.
- **Needs a person because:** independent review disputed High: Regional availability of FSx access points adds no evidenced public or untrusted-network exposure. The quote does not describe internet access or weakened endpoint controls.

### FEAT-0018 · Amazon S3 now supports PrivateLink for FIPS endpoints – High

*New feature · AWS launch (What's New) · released 2026-09-03 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-privatelink-fips-endpoints)*

**Change:** Amazon S3 now supports AWS PrivateLink for endpoints that have been validated under the Federal Information Processing Standard (FIPS) 140-3 program. Customers with security and compliance requirements can use FIPS-validated cryptographic modules when connecting to S3 while keeping their traffic within their Virtual Private Cloud (VPC). To get started, create a new or edit an existing interface V…
**AWS documentation given to the AI:** [Amazon S3 now supports PrivateLink for FIPS endpoints](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-privatelink-fips-endpoints) – “Amazon S3 now supports PrivateLink for FIPS endpoints. Amazon S3 now supports AWS PrivateLink for endpoints that have been validated under the Federal Information Processing Standard (FIPS) 140-3 program. Customers with security and compliance requirements can use FIPS-validated cryptographic modul…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | YES ✔ | 3/3 | create a new or edit an existing interface VPC endpoint for S3 |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | 0/3 | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 1/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS0; severity High (TnC scenario table).
- **Risk (AI):** Administrators can configure FIPS S3 interface endpoints; configurations require validation to preserve intended VPC-restricted access.
- **Existing assessment:** Partly covered – “Use resource-based policies to ensure S3 is accessible only through VPC endpoints”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** NOT upheld – PrivateLink FIPS endpoints keep S3 traffic within a VPC and use FIPS-validated cryptography. This reduces rather than introduces untrusted-network exposure.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **Needs a person because:** independent review disputed High: PrivateLink FIPS endpoints keep S3 traffic within a VPC and use FIPS-validated cryptography. This reduces rather than introduces untrusted-network exposure.

### FEAT-0021 · Amazon S3 server access logs now deliver to Amazon CloudWatch Logs and Amazon S3 Tables – High

*New feature · AWS launch (What's New) · released 2026-06-29 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-cloudwatch-logs-tables/)*

**Change:** Amazon S3 now supports delivering server access logs to Amazon CloudWatch Logs, giving you instant querying, alarms, cross-account and cross-Region aggregation, and AWS Key Management Service (KMS) encryption for your access log data. You can also mirror your logs to Amazon S3 Tables in Apache Iceberg format at no additional storage cost. These new delivery paths complement the existing free deli…
**AWS documentation given to the AI:** [Amazon S3 server access logs now deliver to Amazon CloudWatch Logs and Amazon S3 Tables](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-cloudwatch-logs-tables/) – “Amazon S3 server access logs now deliver to Amazon CloudWatch Logs and Amazon S3 Tables. Amazon S3 now supports delivering server access logs to Amazon CloudWatch Logs, giving you instant querying, alarms, cross-account and cross-Region aggregation, and AWS Key Management Service (KMS) encryption f…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | YES ✔ | 3/3 | giving you instant querying, alarms, cross-account and cross-Region aggregation |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | 0/3 | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | YES ✔ | 3/3 | AWS Key Management Service (KMS) encryption for your access log data |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | 3/3 | Amazon S3 now supports delivering server access logs to Amazon CloudWatch Logs |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS1, XS2, XS6; severity High (TnC scenario table).
- **Risk (AI):** Administrators can aggregate S3 access logs across accounts; unauthorized destinations could expose access-log data and weaken traceability.
- **Existing assessment:** Partly covered – “S3 cross-account sharing should not be allowed. Ensure access through S3 resource-based policies is limited to the same account.”
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** upheld (XS1) – The new CloudWatch Logs delivery path explicitly supports cross-account aggregation, creating a configurable cross-account path for S3 server access-log data.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0023 · Simplified permissions for Amazon S3 Tables and Iceberg materialized views are now available in AWS GovCloud (US) Regions – High

*New feature · AWS launch (What's New) · released 2026-06-05 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/gdc-s3tables-simplified-permissions-in-aws-govcloud/)*

**Change:** AWS Glue Data Catalog now supports AWS IAM-based authorization for Amazon S3 Tables and Apache Iceberg materialized views. With IAM-based authorization, you can define all necessary permissions across storage, catalog, and query engines in a single IAM policy. This capability simplifies the integration of S3 Tables or materialized views with any AWS Analytics service, including Amazon Athena, Ama…
**AWS documentation given to the AI:** [Simplified permissions for Amazon S3 Tables and Iceberg materialized views are now availa…](https://aws.amazon.com/about-aws/whats-new/2026/06/gdc-s3tables-simplified-permissions-in-aws-govcloud/) – “Simplified permissions for Amazon S3 Tables and Iceberg materialized views are now available in AWS GovCloud (US) Regions. AWS Glue Data Catalog now supports AWS IAM-based authorization for Amazon S3 Tables and Apache Iceberg materialized views. With IAM-based authorization, you can define all nece…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | YES ✔ | 3/3 | AWS Glue Data Catalog now supports AWS IAM-based authorization for Amazon S3 Tables and Apache Iceberg materialized views. |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | 0/3 | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | 0/3 | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS5; severity High (TnC scenario table).
- **Risk (AI):** IAM policies can authorize access across storage, catalog, and query engines, increasing the impact of an overly permissive policy.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave High, High, High (agreement 100 %).
- **Independent review:** upheld (XS5) – GovCloud now supports IAM-based authorization for S3 Tables and materialized views, introducing IAM policy authorization across storage, catalog, and query-engine resources.
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Specific IAM actions and configurable resource settings are not provided.

### FEAT-0020 · Amazon S3 server access log delivery to Amazon CloudWatch Logs now available in five additional AWS Regions – Medium

*New feature · AWS launch (What's New) · released 2026-10-21 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-server-access-logs-cloudwatch-logs-regions/)*

**Change:** Amazon S3 server access log delivery to Amazon CloudWatch Logs is now available in the AWS GovCloud (US) Regions, AWS European Sovereign Cloud, and AWS China Regions. With your access logs in CloudWatch Logs, you can query them as they arrive, set alarms, and encrypt them with AWS Key Management Service (AWS KMS) keys. This new delivery path complements the existing delivery of server access logs…
**AWS documentation given to the AI:** [Amazon S3 server access log delivery to Amazon CloudWatch Logs now available in five addi…](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-server-access-logs-cloudwatch-logs-regions/) – “Amazon S3 server access log delivery to Amazon CloudWatch Logs now available in five additional AWS Regions. Amazon S3 server access log delivery to Amazon CloudWatch Logs is now available in the AWS GovCloud (US) Regions, AWS European Sovereign Cloud, and AWS China Regions. With your access logs i…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | 0/3 | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | 0/3 | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | 0/3 | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | YES ✔ | 3/3 | set alarms, and encrypt them with AWS Key Management Service (AWS KMS) keys |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | 0/3 | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | 0/3 | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | 3/3 | This new delivery path complements the existing delivery of server access logs to S3 general purpose buckets |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS2, XS6; severity Medium (TnC scenario table).
- **Risk (AI):** Administrators can use a new CloudWatch Logs delivery path; log confidentiality and traceability depend on KMS encryption and monitoring configuration.
- **Existing assessment:** Not covered
- **Consistency vote:** 3 attempts gave Medium, Medium, Medium (agreement 100 %).
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### ACT-0010 · s3:AllowVendedLogDeliveryForResource – Low

*IAM action · Added · released 2026-06-04 · source [link](https://github.com/iann0036/iam-dataset/commit/ae377b996fada60bba62ed5d5e80edfa848addfb)*

**Change:** New action (Read). Resources: bucket. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:deliverySourceArn, s3:logType, s3:resourceArnBeingAuthorized.
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | New action (Read). Resources: bucket. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:deliverySourceArn, s3:logType |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** Authorized principals can use a new vended-log delivery authorization action, affecting log-delivery traceability configuration.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration or Describe API is supplied for vended-log delivery authorization.

### FEAT-0001 · AWS Backup enhances Amazon S3 backup copy performance – Low

*New feature · AWS launch (What's New) · released 2026-06-25 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/aws-backup-amazon-s3-copy-enhancement/)*

**Change:** AWS Backup now executes S3 backup copy operations up to 8x faster for buckets with millions of objects and low change rates between backup copies through enhanced change tracking. This improvement reduces the time required to copy S3 backups across accounts and AWS Regions by eliminating the need to scan all objects in the destination account or Region. With this improvement, AWS Backup records o…
**AWS documentation given to the AI:** [AWS Backup enhances Amazon S3 backup copy performance](https://aws.amazon.com/about-aws/whats-new/2026/06/aws-backup-amazon-s3-copy-enhancement/) – “AWS Backup enhances Amazon S3 backup copy performance. AWS Backup now executes S3 backup copy operations up to 8x faster for buckets with millions of objects and low change rates between backup copies through enhanced change tracking. This improvement reduces the time required to copy S3 backups ac…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | With this improvement, AWS Backup records object events as they occur, resulting in faster copy operations and reduced processing time. |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** AWS Backup newly records S3 object events during cross-account and cross-Region backup copies, changing backup-copy traceability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence provides no resource configuration for the enhanced event tracking.

### FEAT-0002 · AWS Backup for Amazon S3 now supports direct access to backup data – Low

*New feature · AWS launch (What's New) · released 2026-08-06 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/08/aws-backup-amazon-s3-direct-access/)*

**Change:** AWS Backup for Amazon S3 now supports creating S3 Access Points, providing immediate read-only access to backup data using standard S3 APIs without initiating a restore. This enables targeted file recovery, data validation, compliance auditing, and forensic investigation while your backup data remains protected in your backup vault. You can create an access point for an S3 recovery point and read…
**AWS documentation given to the AI:** [AWS Backup for Amazon S3 now supports direct access to backup data](https://aws.amazon.com/about-aws/whats-new/2026/08/aws-backup-amazon-s3-direct-access/) – “AWS Backup for Amazon S3 now supports direct access to backup data. AWS Backup for Amazon S3 now supports creating S3 Access Points, providing immediate read-only access to backup data using standard S3 APIs without initiating a restore. This enables targeted file recovery, data validation, complia…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | UNKNOWN | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | This enables targeted file recovery, data validation, compliance auditing, and forensic investigation while your backup data remains protected in your backup v… |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** Users can read backup recovery-point data through S3 Access Points without restore, changing the recovery and backup-data access process.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not specify access-point authorization or resource configuration.

### FEAT-0003 · Amazon Redshift long-term system table retention with Amazon S3 Tables is now available in additional AWS Regions – Low

*New feature · AWS launch (What's New) · released 2026-10-15 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/redshift-long-term-system-table-retention-expansion/)*

**Change:** Amazon Redshift long-term retention for system table data through native integration with Amazon S3 Tables is now available in additional AWS Regions. With this feature, you can configure your Redshift system table data retention beyond the current 7-day limit to meet your compliance, auditing, and observability requirements. Once enabled, AWS automatically writes system table data to S3 Tables i…
**AWS documentation given to the AI:** [Amazon Redshift long-term system table retention with Amazon S3 Tables is now available i…](https://aws.amazon.com/about-aws/whats-new/2026/10/redshift-long-term-system-table-retention-expansion/) – “Amazon Redshift long-term system table retention with Amazon S3 Tables is now available in additional AWS Regions. Amazon Redshift long-term retention for system table data through native integration with Amazon S3 Tables is now available in additional AWS Regions. With this feature, you can config…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | you can configure your Redshift system table data retention beyond the current 7-day limit to meet your compliance, auditing, and observability requirements |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | Customers use Redshift system tables to monitor query performance, audit data warehouse activity, and meet compliance requirements. |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4, XS6; severity Low (TnC scenario table).
- **Risk (AI):** Redshift users in additional Regions can retain system-table data beyond seven days, altering audit-data retention and deletion timing.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No S3 Tables retention configuration API or setting is supplied.

### FEAT-0007 · Amazon S3 Object Lock now supports variable retention with event holds – Low

*New feature · AWS launch (What's New) · released 2026-09-08 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-object-lock-variable-retention/)*

**Change:** Amazon S3 Object Lock now supports variable retention, allowing you to apply write-once-read-many (WORM) protection to objects whose required retention period starts with a future event, such as a contract closing or an audit completing. You place an event hold with a retention duration on an object and S3 protects the object while the hold is in place. When you release the hold, S3 retains the o…
**AWS documentation given to the AI:** [Amazon S3 Object Lock now supports variable retention with event holds](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-object-lock-variable-retention/) – “Amazon S3 Object Lock now supports variable retention with event holds. Amazon S3 Object Lock now supports variable retention, allowing you to apply write-once-read-many (WORM) protection to objects whose required retention period starts with a future event, such as a contract closing or an audit c…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | You place an event hold with a retention duration on an object and S3 protects the object while the hold is in place. |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** Event-hold configuration or release can affect the retention period and recovery availability of protected S3 objects.
- **Existing assessment:** Partly covered – “Deny S3 buckets without Object Lock to provide data immutability and prevent unauthorized or accidental deletion.”
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence provides no resource configuration or API for event-hold monitoring.

### FEAT-0008 · Amazon S3 Object Lock variable retention with event holds is now available in AWS GovCloud (US) Regions – Low

*New feature · AWS launch (What's New) · released 2026-10-01 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-object-lock-variable-retention-event-holds-aws-govcloud/)*

**Change:** Amazon S3 Object Lock support for variable retention with event holds is now available in AWS GovCloud (US-East) and AWS GovCloud (US-West). Amazon S3 Object Lock variable retention allows you to apply write-once-read-many (WORM) protection to objects whose required retention period starts with a future event, such as a contract closing or an audit completing. You place an event hold with a reten…
**AWS documentation given to the AI:** [Amazon S3 Object Lock variable retention with event holds is now available in AWS GovClou…](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-object-lock-variable-retention-event-holds-aws-govcloud/) – “Amazon S3 Object Lock variable retention with event holds is now available in AWS GovCloud (US) Regions. Amazon S3 Object Lock support for variable retention with event holds is now available in AWS GovCloud (US-East) and AWS GovCloud (US-West). Amazon S3 Object Lock variable retention allows you t…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | You place an event hold with a retention duration on an object and S3 protects the object while the hold is in place. |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** GovCloud event-hold configuration or release can affect protected-object retention periods and recovery availability.
- **Existing assessment:** Partly covered – “Deny S3 buckets without Object Lock to provide data immutability and prevent unauthorized or accidental deletion.”
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence provides no resource configuration or API for event-hold monitoring.

### FEAT-0010 · Amazon S3 Tables now support S3 server access logging – Low

*New feature · AWS launch (What's New) · released 2026-11-10 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-tables-server-access-logging/)*

**Change:** Amazon S3 Tables now support server access logging, providing you detailed records of the requests made to your table buckets. You can now track table operations, namespace management, and data access to support security audits, access analysis, and cost attribution. Log delivery is available at no additional cost, and you pay only for the storage of the log files. When you enable server access l…
**AWS documentation given to the AI:** [Amazon S3 Tables now support S3 server access logging](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-tables-server-access-logging/) – “Amazon S3 Tables now support S3 server access logging. Amazon S3 Tables now support server access logging, providing you detailed records of the requests made to your table buckets. You can now track table operations, namespace management, and data access to support security audits, access analysis…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | Amazon S3 Tables now support server access logging, providing you detailed records of the requests made to your table buckets. |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** Table-bucket server access logging can record requests, improving traceability of table operations and data access.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No table-bucket configuration API or Prisma resource schema is supplied.

### FEAT-0016 · Amazon S3 adds additional policy details to access denied error messages – Low

*New feature · AWS launch (What's New) · released 2026-08-13 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/08/s3-additional-policy-details-access-denied-error-messages/)*

**Change:** Amazon S3 now includes the specific AWS Identity and Access Management (IAM) and AWS Organizations policy Amazon Resource Name (ARN) in HTTP 403 Access Denied error messages for same-account and same-organization requests. This helps you quickly identify the exact policy responsible for a denied request and remediate the issue directly. Previously, S3 access denied error messages included the pol…
**AWS documentation given to the AI:** [Amazon S3 adds additional policy details to access denied error messages](https://aws.amazon.com/about-aws/whats-new/2026/08/s3-additional-policy-details-access-denied-error-messages/) – “Amazon S3 adds additional policy details to access denied error messages. Amazon S3 now includes the specific AWS Identity and Access Management (IAM) and AWS Organizations policy Amazon Resource Name (ARN) in HTTP 403 Access Denied error messages for same-account and same-organization requests. Th…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | This helps you quickly identify the exact policy responsible for a denied request |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** Same-account users receiving denied requests can identify specific IAM or SCP policy ARNs, exposing policy-identification details.
- **Existing assessment:** Not covered – “No XS4 or XS6 row or overall service risk rating is visible in the supplied screenshots.”
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0017 · Amazon S3 adds annotations to provide AI agents and analytics tools with context for data discovery – Low

*New feature · AWS launch (What's New) · released 2026-06-16 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-annotations-business-context)*

**Change:** Amazon S3 adds annotations, so you can attach custom metadata to your S3 objects at massive scale, giving AI agents and analytics tools the context they need to find and use the right data. Annotations are a new metadata capability purpose-built for attaching business context directly in JSON, XML, or YAML to your objects, with up to 1GB per object. Annotations can be modified or deleted at any t…
**AWS documentation given to the AI:** [Amazon S3 adds annotations to provide AI agents and analytics tools with context for data…](https://aws.amazon.com/about-aws/whats-new/2026/06/amazon-s3-annotations-business-context) – “Amazon S3 adds annotations to provide AI agents and analytics tools with context for data discovery. Amazon S3 adds annotations, so you can attach custom metadata to your S3 objects at massive scale, giving AI agents and analytics tools the context they need to find and use the right data. Annotati…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | Annotations can be modified or deleted at any time, making it easier to keep context current |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** Authorized users can delete object annotations, causing loss of business context used for data discovery and governance.
- **Existing assessment:** Not covered – “No XS4 or XS6 row or overall service risk rating is visible in the supplied screenshots.”
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0019 · Amazon S3 removes 30-day minimum for transitions to S3 Standard-IA and S3 One Zone-IA – Low

*New feature · AWS launch (What's New) · released 2026-07-16 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-removes-30-day-transitions-standard-ia-one-zone-ia)*

**Change:** You can now transition objects to S3 Standard-Infrequent Access (S3 Standard-IA) and S3 One Zone-Infrequent Access (S3 One Zone-IA) as soon as the day they are created, without the previous 30-day minimum retention in S3 Standard. These storage classes offer up to 40% lower storage costs than S3 Standard while still providing millisecond access when needed, making them ideal for backups, log anal…
**AWS documentation given to the AI:** [Amazon S3 removes 30-day minimum for transitions to S3 Standard-IA and S3 One Zone-IA](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-removes-30-day-transitions-standard-ia-one-zone-ia) – “Amazon S3 removes 30-day minimum for transitions to S3 Standard-IA and S3 One Zone-IA. You can now transition objects to S3 Standard-Infrequent Access (S3 Standard-IA) and S3 One Zone-Infrequent Access (S3 One Zone-IA) as soon as the day they are created, without the previous 30-day minimum retenti…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 3. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | as soon as the day they are created, without the previous 30-day minimum retention |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** Administrators can transition objects immediately through Lifecycle rules, requiring application-specific assessment of retention and recovery implications.
- **Existing assessment:** Not covered – “No XS4 or XS6 row or overall service risk rating is visible in the supplied screenshots.”
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0024 · s3.DeleteObjectAnnotation – Low

*New feature · New API operation · released 2026-06-16 · source [link](https://github.com/boto/botocore/blob/1.43.31/CHANGELOG.rst)*

**Change:** New API operation: Deletes a specific annotation from an Amazon S3 object.
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"deleteObjectAnnotation","context":"# delet…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3/aws.sdk.kotlin.services.s3/delete-object-annotation.html) – “{"content":{"result":"s3/aws.sdk.kotlin.services.s3/deleteObjectAnnotation\n\n# deleteObjectAnnotation\n\ninline suspend fun S3Client.deleteObjectAnnotation(crossinline block: DeleteObjectAnnotationRequest.Builder.() -> Unit): DeleteObjectAnnotationResponse\n\nDeletes a specific annotation from an…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | YES ✔ | – | Deleting an annotation is permanent. Annotations are not independently versioned, so there is no delete marker or way to recover a deleted annotation. |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS4; severity Low (TnC scenario table).
- **Risk (AI):** An identity permitted to delete object annotations can permanently remove annotation context, with no delete marker or recovery path.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration or describe API evidence supports a Prisma CONFIG rule for annotation deletion.

### FEAT-0025 · Amazon S3 Event Notifications now include system-generated tags – Low

*New feature · AWS launch (What's New) · released 2026-07-16 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/07/amazon-s3-event-notifications-system-generated-tags/)*

**Change:** Amazon S3 Event Notifications now include system-generated tags in events delivered to all destinations including Amazon EventBridge, Amazon SQS, Amazon SNS, and AWS Lambda. System-generated tags are metadata labels attached to your bucket by AWS services. You can use these tags to filter events from thousands of buckets with a single EventBridge rule, instead of listing each bucket name individu…
**AWS documentation given to the AI:** [Amazon S3 Event Notifications now include system-generated tags](https://aws.amazon.com/about-aws/whats-new/2026/07/amazon-s3-event-notifications-system-generated-tags/) – “Amazon S3 Event Notifications now include system-generated tags. Amazon S3 Event Notifications now include system-generated tags in events delivered to all destinations including Amazon EventBridge, Amazon SQS, Amazon SNS, and AWS Lambda. System-generated tags are metadata labels attached to your b…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | Amazon S3 Event Notifications now include system-generated tags in events delivered to all destinations including Amazon EventBridge, Amazon SQS, Amazon SNS, a… |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** Event notifications can now carry system-generated tags, changing event metadata available for monitoring and traceability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No resource configuration setting for system-generated event tags is evidenced.

### FEAT-0029 · Amazon S3 Metadata and annotations are now available in AWS GovCloud (US) Regions – Low

*New feature · AWS launch (What's New) · released 2026-08-18 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-s3-metadata-annotations-govcloud-regions)*

**Change:** Amazon S3 Metadata and annotations are now available in AWS GovCloud (US-East) and AWS GovCloud (US-West) Regions, giving you the easiest and fastest way to discover, understand, and enrich your S3 data. S3 Metadata automatically captures system-defined details like object size and source, and stores them in Amazon S3 Tables, optimized for near real-time tabular queries that simplify business ana…
**AWS documentation given to the AI:** [Amazon S3 Metadata and annotations are now available in AWS GovCloud (US) Regions](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-s3-metadata-annotations-govcloud-regions) – “Amazon S3 Metadata and annotations are now available in AWS GovCloud (US) Regions. Amazon S3 Metadata and annotations are now available in AWS GovCloud (US-East) and AWS GovCloud (US-West) Regions, giving you the easiest and fastest way to discover, understand, and enrich your S3 data. S3 Metadata…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | S3 Metadata automatically captures system-defined details like object size and source, and stores them in Amazon S3 Tables. |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** S3 Metadata automatically records object-source details, creating traceability data not addressed by the current assessment.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No metadata resource configuration or describe API is evidenced.

### FEAT-0030 · Amazon S3 Metadata now available in six additional AWS Regions – Low

*New feature · AWS launch (What's New) · released 2026-10-15 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-metadata-six-additional-regions/)*

**Change:** Amazon S3 Metadata is now available in six additional AWS Regions: Asia Pacific (Hyderabad), Asia Pacific (Malaysia), Asia Pacific (New Zealand), Asia Pacific (Taipei), Asia Pacific (Thailand), and Mexico (Central). S3 Metadata is a fast and easy way to instantly discover and understand your S3 data with automated, easily queried metadata that updates in near real time. This helps you curate, ide…
**AWS documentation given to the AI:** [Amazon S3 Metadata now available in six additional AWS Regions](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-metadata-six-additional-regions/) – “Amazon S3 Metadata now available in six additional AWS Regions. Amazon S3 Metadata is now available in six additional AWS Regions: Asia Pacific (Hyderabad), Asia Pacific (Malaysia), Asia Pacific (New Zealand), Asia Pacific (Taipei), Asia Pacific (Thailand), and Mexico (Central). S3 Metadata is a fa…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | YES ✔ | – | S3 Metadata is a fast and easy way to instantly discover and understand your S3 data with automated, easily queried metadata that updates in near real time. |

- **Kit's result:** Risk mapped to the TnC framework; scenarios XS6; severity Low (TnC scenario table).
- **Risk (AI):** S3 Metadata becomes available in additional Regions, extending automated metadata capabilities that can support data traceability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No metadata resource configuration or describe API is evidenced.

### ACT-0005 · s3:PutObjectAnnotation – Uncertain – needs a human decision

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+2 more).
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"How do I resolve the 403 Access Denied err…](https://docs.aws.amazon.com/AmazonS3/latest/userguide/batch-ops-iam-role-policies.html) – “{"content":{"result":"Table of Contents:\n- Granting permissions for Batch Operations (char 0-17399)\n - Creating an S3 Batch Operations IAM role (char 727-3676)\n - Trust policy (char 3274-3676)\n - Using JSON (char 3415-3431)\n - Using (char 3432-3676)\n - Attaching permissions policies (char 367…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | – | – |

- **Kit's result:** Uncertain – needs a human decision; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The security effect of writing an object annotation cannot be determined from the supplied evidence.
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence does not define object annotations or their audit effect.
- **Needs a person because:** security relevance could not be established from the evidence (unknown: logging_audit)

### ACT-0006 · s3:PutObjectVersionAnnotation – Uncertain – needs a human decision

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: accesspointobject, object. Condition keys: aws:ResourceTag/${TagKey}, s3:AccessPointNetworkOrigin, s3:AccessPointTag/${TagKey}, s3:BucketTag/${TagKey}, s3:DataAccessPointAccount, s3:DataAccessPointArn, s3:ExistingObjectTag/<key>, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion (+3 more).
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | – | – |

- **Kit's result:** Uncertain – needs a human decision; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The security effect of writing a version annotation cannot be determined from the supplied evidence.
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No documentation defines version annotations or their audit effect.
- **Needs a person because:** security relevance could not be established from the evidence (unknown: logging_audit)

### ACT-0007 · s3:ReplicateObjectAnnotation – Uncertain – needs a human decision

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: object. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion, s3:x-amz-content-sha256.
**AWS documentation given to the AI:** none found – the AI judged from the change details only.
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | UNKNOWN | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | – | – |

- **Kit's result:** Uncertain – needs a human decision; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The supplied evidence does not establish whether replicated object annotations copy data or affect audit records.
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** No documentation defines replicated object annotations or their data scope.
- **Needs a person because:** security relevance could not be established from the evidence (unknown: data_sharing, logging_audit)

### ACT-0008 · s3:UpdateBucketMetadataAnnotationTableConfiguration – Uncertain – needs a human decision

*IAM action · Added · released 2026-06-16 · source [link](https://github.com/iann0036/iam-dataset/commit/63de73515088ed65f19e854b8a783cdb2a90091d)*

**Change:** New action (Write). Resources: bucket. Condition keys: aws:ResourceTag/${TagKey}, s3:BucketTag/${TagKey}, s3:ResourceAccount, s3:TlsVersion, s3:authType, s3:signatureAge, s3:signatureversion, s3:x-amz-content-sha256.
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"Api-S3-2006-03-01","context":"### UpdateBu…](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-s3-2006-03-01.html) – “RenameObject (char 828208-835838)\n - Parameter Syntax (char 830836-831490)\n - Parameter Details (char 831491-835254)\n - Result Syntax (char 835255-835286)\n - Result Details (char 835287-835357)\n - Errors (char 835358-835838)\n - RestoreObject (char 835839-851067)\n - Parameter Syntax (char 842…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | UNKNOWN | – | – |

- **Kit's result:** Uncertain – needs a human decision; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The security effect of updating bucket metadata annotation-table configuration cannot be determined from the supplied evidence.
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence names the operation but does not describe its configuration effect.
- **Needs a person because:** security relevance could not be established from the evidence (unknown: logging_audit)

### ACT-0009 · s3: 12 actions – No security risk

*IAM action · Modified · released after 2026-05-18 (exact date not published) · source [link](https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json)*

**Change:** Access level Tagging -> Tagging, Write - applies to 12 actions: DeleteJobTagging, DeleteObjectTagging, DeleteObjectVersionTagging, DeleteStorageLensConfigurationTagging, PutBucketTagging, PutJobTagging, PutObjectTagging, PutObjectVersionTagging, PutStorageLensConfigurationTagging, ReplicateTags, TagResource, UntagResource
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"s3control","context":"### Permissions\n\nT…](https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_CreateJob.html) – “obOperation\n\n## Request Syntax\n\n```\nPOST /v20180820/jobs HTTP/1.1\nHost: s3-control.amazonaws.com\nx-amz-account-id: AccountId\n<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<CreateJobRequest xmlns=\"http://awss3control.amazonaws.com/doc/2018-08-20/\">\n <ConfirmationRequired>boolean</Confirmati…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The change reclassifies existing actions as Write and does not evidence a changed security capability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### ACT-0021 · s3express: 6 actions – No security risk

*IAM action · Modified · released after 2026-05-18 (exact date not published) · source [link](https://servicereference.us-east-1.amazonaws.com/v1/s3express/s3express.json)*

**Change:** Access level Permissions management -> Permissions management, Write - applies to 6 actions: DeleteAccessPointPolicy, DeleteAccessPointScope, DeleteBucketPolicy, PutAccessPointPolicy, PutAccessPointScope, PutBucketPolicy
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"AWS::S3Express::BucketPolicy","context":"#…](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-s3express-bucketpolicy.html) – “themselves out of\n their own buckets, the root principal in a bucket owner's AWS account can perform the `GetBucketPolicy`,\n `PutBucketPolicy`, and `DeleteBucketPolicy` API\n actions, even if their bucket policy explicitly denies the root principal's\n access. Bucket owner root principals can onl…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 6. S…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### ACT-0022 · s3express:TagResource – No security risk

*IAM action · Modified · released after 2026-05-18 (exact date not published) · source [link](https://servicereference.us-east-1.amazonaws.com/v1/s3express/s3express.json)*

**Change:** Access level Tagging -> Tagging, Write
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"tagResource","context":"## Directory bucke…](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/s3control/aws.sdk.kotlin.services.s3control/-s3-control-client/tag-resource.html) – “tag. Each tag is a label consisting of a key and value that is applied to your resource. Tags can help you organize, track costs for, and control access to your resources. You can add up to 50 Amazon Web Services resource tags for each S3 resource.\n\nThis operation is only supported for the follow…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### ACT-0023 · s3express:UntagResource – No security risk

*IAM action · Modified · released after 2026-05-18 (exact date not published) · source [link](https://servicereference.us-east-1.amazonaws.com/v1/s3express/s3express.json)*

**Change:** Access level Tagging -> Tagging, Write
**AWS documentation given to the AI:** [{"content":{"result":[{"rank_order":1,"title":"Api-S3control-2018-08-20","context":"### U…](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-s3control-2018-08-20.html) – “- PutStorageLensConfigurationTagging (char 279777-281430)\n - Parameter Syntax (char 280464-280804)\n - Parameter Details (char 280805-281263)\n - Result Syntax (char 281264-281295)\n - Result Details (char 281296-281366)\n - Errors (char 281367-281430)\n - SubmitMultiRegionAccessPointRoutes (char…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO
- **What the AI was unsure about:** Evidence only identifies UntagResource in an API index.

### FEAT-0005 · Amazon S3 Access Grants now support listing grants across multiple AWS accounts – No security risk

*New feature · AWS launch (What's New) · released 2026-11-10 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-access-grants-list-across-regions/)*

**Change:** Amazon S3 Access Grants can now discover and list all your grants across multiple AWS accounts with a single API call, eliminating the need to query each account separately. A single ListCallerAccessGrants call returns grants from your own S3 Access Grants instance and every instance shared with you, giving you a complete view of the S3 data you can access. By setting the new InstanceScope parame…
**AWS documentation given to the AI:** [Amazon S3 Access Grants now support listing grants across multiple AWS accounts](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-access-grants-list-across-regions/) – “Amazon S3 Access Grants now support listing grants across multiple AWS accounts. Amazon S3 Access Grants can now discover and list all your grants across multiple AWS accounts with a single API call, eliminating the need to query each account separately. A single ListCallerAccessGrants call returns…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The feature consolidates visibility of grants already available to the caller; evidence does not show new data access or sharing permissions.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0009 · Amazon S3 Tables now provide a simple, cost-effective path for data lake modernization – No security risk

*New feature · AWS launch (What's New) · released 2026-11-04 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-tables-managed-migration-hive-to-iceberg/)*

**Change:** Amazon S3 Tables now offer managed migration of your Apache Hive tables to Apache Iceberg. Iceberg adds ACID transactions, schema evolution, and time travel to your data lake, but moving off Hive usually means building a migration pipeline for each table, which can take weeks. You can now let S3 Tables manage your migration, resulting in query-ready Iceberg tables that stay optimized as data grow…
**AWS documentation given to the AI:** [Amazon S3 Tables now provide a simple, cost-effective path for data lake modernization](https://aws.amazon.com/about-aws/whats-new/2026/11/amazon-s3-tables-managed-migration-hive-to-iceberg/) – “Amazon S3 Tables now provide a simple, cost-effective path for data lake modernization. Amazon S3 Tables now offer managed migration of your Apache Hive tables to Apache Iceberg. Iceberg adds ACID transactions, schema evolution, and time travel to your data lake, but moving off Hive usually means b…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Managed Hive-to-Iceberg migration is a functional data-conversion capability; supplied evidence shows no new TnC scenario capability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0011 · Amazon S3 Tables now support all Apache Iceberg V3 data types – No security risk

*New feature · AWS launch (What's New) · released 2026-09-30 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-tables-iceberg-v3-data-types)*

**Change:** Amazon S3 Tables add support for geometry, geography, unknown, and nanosecond timestamp data types, along with column default values, as defined in the Apache Iceberg Version 3 (V3) specification. You can now store geospatial coordinates and nanosecond-precision event times natively instead of encoding them in strings or integers. This helps simplify data architecture while improving storage effi…
**AWS documentation given to the AI:** [Amazon S3 Tables now support all Apache Iceberg V3 data types](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-s3-tables-iceberg-v3-data-types) – “Amazon S3 Tables now support all Apache Iceberg V3 data types. Amazon S3 Tables add support for geometry, geography, unknown, and nanosecond timestamp data types, along with column default values, as defined in the Apache Iceberg Version 3 (V3) specification. You can now store geospatial coordinate…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Additional Iceberg V3 data types are a functional schema capability; supplied evidence shows no new TnC scenario capability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0012 · Amazon S3 Tables now support up to 100 table buckets per AWS Region in an AWS account – No security risk

*New feature · AWS launch (What's New) · released 2026-10-01 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-tables-table-bucket-increase)*

**Change:** Amazon S3 Tables now support up to 100 table buckets per AWS Region in an AWS account, increased from 10. This allows you to create up to 1 million tables per AWS Region in an AWS account. With a higher table bucket allowance, you can create a separate table bucket for each dataset, workload, or team, and apply table bucket-level settings such as encryption, access policies, and replication to ea…
**AWS documentation given to the AI:** [Amazon S3 Tables now support up to 100 table buckets per AWS Region in an AWS account](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-tables-table-bucket-increase) – “Amazon S3 Tables now support up to 100 table buckets per AWS Region in an AWS account. Amazon S3 Tables now support up to 100 table buckets per AWS Region in an AWS account, increased from 10. This allows you to create up to 1 million tables per AWS Region in an AWS account. With a higher table buc…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** The increased table-bucket quota expands scale only; supplied evidence shows no new sharing, encryption, retention, or access-control option.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0013 · Amazon S3 Vectors introduces metadata pre-filtering for up to 5x higher recall on filtered search – No security risk

*New feature · AWS launch (What's New) · released 2026-09-30 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/09/s3-vectors-introduces-metadata-pre-filtering/)*

**Change:** Amazon S3 Vectors now supports pre-filtering, which evaluates metadata filters before running similarity search, returning up to 5x more of the matching vectors when your filter is selective. S3 Vectors also adds a prefix match operator ($startsWith) for filtering on values like paths and URLs. Together, these improvements give your retrieval-augmented generation (RAG), agentic, and semantic-sear…
**AWS documentation given to the AI:** [Amazon S3 Vectors introduces metadata pre-filtering for up to 5x higher recall on filtere…](https://aws.amazon.com/about-aws/whats-new/2026/09/s3-vectors-introduces-metadata-pre-filtering/) – “Amazon S3 Vectors introduces metadata pre-filtering for up to 5x higher recall on filtered search. Amazon S3 Vectors now supports pre-filtering, which evaluates metadata filters before running similarity search, returning up to 5x more of the matching vectors when your filter is selective. S3 Vecto…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Vector metadata pre-filtering and prefix matching are query-result features; supplied evidence shows no new TnC scenario capability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0014 · Amazon S3 Vectors is now available in AWS GovCloud (US) Regions – No security risk

*New feature · AWS launch (What's New) · released 2026-07-07 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-vectors-available-aws-govcloud-regions/)*

**Change:** Amazon S3 Vectors is now available in AWS GovCloud (US-East) and AWS GovCloud (US-West). Amazon S3 Vectors is purpose-built vector storage for AI agents, inference, Retrieval Augmented Generation (RAG), and semantic search at billion-vector scale. S3 Vectors is designed to provide the same elasticity, durability, and availability as Amazon S3, with a dedicated set of APIs that let you store, acce…
**AWS documentation given to the AI:** [Amazon S3 Vectors is now available in AWS GovCloud (US) Regions](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-vectors-available-aws-govcloud-regions/) – “Amazon S3 Vectors is now available in AWS GovCloud (US) Regions. Amazon S3 Vectors is now available in AWS GovCloud (US-East) and AWS GovCloud (US-West). Amazon S3 Vectors is purpose-built vector storage for AI agents, inference, Retrieval Augmented Generation (RAG), and semantic search at billion-…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0015 · Amazon S3 Vectors is now available in the AWS European Sovereign Cloud (Germany) Region – No security risk

*New feature · AWS launch (What's New) · released 2026-08-04 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-s3-vectors-european-sovereign-cloud-germany/)*

**Change:** Amazon S3 Vectors is now available in the AWS European Sovereign Cloud (Germany) Region. Amazon S3 Vectors is purpose-built vector storage for AI agents, inference, Retrieval Augmented Generation (RAG), and semantic search at billion-vector scale. S3 Vectors is designed to provide the same elasticity, durability, and availability as Amazon S3, with a dedicated set of APIs that let you store, acce…
**AWS documentation given to the AI:** [Amazon S3 Vectors is now available in the AWS European Sovereign Cloud (Germany) Region](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-s3-vectors-european-sovereign-cloud-germany/) – “Amazon S3 Vectors is now available in the AWS European Sovereign Cloud (Germany) Region. Amazon S3 Vectors is now available in the AWS European Sovereign Cloud (Germany) Region. Amazon S3 Vectors is purpose-built vector storage for AI agents, inference, Retrieval Augmented Generation (RAG), and sem…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Unknown
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0022 · Mountpoint for Amazon S3 adds memory usage controls – No security risk

*New feature · AWS launch (What's New) · released 2026-08-26 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/08/mountpoint-for-S3-adds-memory-usage-controls)*

**Change:** Mountpoint for Amazon S3 can now limit memory usage, either automatically based on the environment it runs in or with a limit that you define. This lets you run Mountpoint alongside memory-intensive applications, for example, in machine learning training or analytics workloads where applications share a memory budget. Previously, Mountpoint’s memory usage could expand over time based on usage pat…
**AWS documentation given to the AI:** [Mountpoint for Amazon S3 adds memory usage controls](https://aws.amazon.com/about-aws/whats-new/2026/08/mountpoint-for-S3-adds-memory-usage-controls) – “Mountpoint for Amazon S3 adds memory usage controls. Mountpoint for Amazon S3 can now limit memory usage, either automatically based on the environment it runs in or with a limit that you define. This lets you run Mountpoint alongside memory-intensive applications, for example, in machine learning…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0026 · Amazon S3 Express One Zone is now available in 4 additional AWS Regions – No security risk

*New feature · AWS launch (What's New) · released 2026-10-12 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-express-one-zone-4-additional-regions/)*

**Change:** The Amazon S3 Express One Zone storage class is now available in 4 additional AWS Regions: Europe (London), Asia Pacific (Jakarta), Asia Pacific (Hong Kong), and Africa (Cape Town). Amazon S3 Express One Zone is a high-performance, single-Availability Zone storage class purpose-built to deliver consistent single-digit millisecond data access for your most frequently accessed data and latency-sens…
**AWS documentation given to the AI:** [Amazon S3 Express One Zone is now available in 4 additional AWS Regions](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-express-one-zone-4-additional-regions/) – “Amazon S3 Express One Zone is now available in 4 additional AWS Regions. The Amazon S3 Express One Zone storage class is now available in 4 additional AWS Regions: Europe (London), Asia Pacific (Jakarta), Asia Pacific (Hong Kong), and Africa (Cape Town). Amazon S3 Express One Zone is a high-perform…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0027 · Amazon S3 Express One Zone is now available in 7 additional AWS Regions – No security risk

*New feature · AWS launch (What's New) · released 2026-09-17 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/09/s3-express-one-zone-7-regions/)*

**Change:** The Amazon S3 Express One Zone storage class is now available in 7 additional AWS Regions: Asia Pacific (Singapore), South America (Sao Paulo), US West (N. California), Canada (Central), Europe (Paris), Asia Pacific (Sydney), and Asia Pacific (Seoul). Amazon S3 Express One Zone is a high-performance, single-Availability Zone storage class purpose-built to deliver consistent single-digit milliseco…
**AWS documentation given to the AI:** [Amazon S3 Express One Zone is now available in 7 additional AWS Regions](https://aws.amazon.com/about-aws/whats-new/2026/09/s3-express-one-zone-7-regions/) – “Amazon S3 Express One Zone is now available in 7 additional AWS Regions. The Amazon S3 Express One Zone storage class is now available in 7 additional AWS Regions: Asia Pacific (Singapore), South America (Sao Paulo), US West (N. California), Canada (Central), Europe (Paris), Asia Pacific (Sydney),…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0028 · Amazon S3 Express One Zone is now available in the AWS Europe (Frankfurt) Region – No security risk

*New feature · AWS launch (What's New) · released 2026-07-07 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-express-one-zone-europe-frankfurt/)*

**Change:** The Amazon S3 Express One Zone storage class is now available in the AWS Europe (Frankfurt) Region. Amazon S3 Express One Zone is a high-performance, single-Availability Zone storage class purpose-built to deliver consistent single-digit millisecond data access for your most frequently accessed data and latency-sensitive applications. S3 Express One Zone delivers data access speed up to 10x faste…
**AWS documentation given to the AI:** [Amazon S3 Express One Zone is now available in the AWS Europe (Frankfurt) Region](https://aws.amazon.com/about-aws/whats-new/2026/07/s3-express-one-zone-europe-frankfurt/) – “Amazon S3 Express One Zone is now available in the AWS Europe (Frankfurt) Region. The Amazon S3 Express One Zone storage class is now available in the AWS Europe (Frankfurt) Region. Amazon S3 Express One Zone is a high-performance, single-Availability Zone storage class purpose-built to deliver con…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0031 · Amazon S3 Tables are now available in two additional AWS Regions – No security risk

*New feature · AWS launch (What's New) · released 2026-05-29 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/05/amazon-s3-tables-aws-regions/)*

**Change:** Amazon S3 Tables are now available in the Asia Pacific (Taipei) and Asia Pacific (New Zealand) Regions. Amazon S3 Tables deliver the first cloud object store with built-in Apache Iceberg support, streamlining tabular data storage at scale. S3 Tables automatically perform continual table maintenance to optimize query efficiency and reduce storage costs as your data lake grows and evolves. Because…
**AWS documentation given to the AI:** [Amazon S3 Tables are now available in two additional AWS Regions](https://aws.amazon.com/about-aws/whats-new/2026/05/amazon-s3-tables-aws-regions/) – “Amazon S3 Tables are now available in two additional AWS Regions. Amazon S3 Tables are now available in the Asia Pacific (Taipei) and Asia Pacific (New Zealand) Regions. Amazon S3 Tables deliver the first cloud object store with built-in Apache Iceberg support, streamlining tabular data storage at…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Regional availability and table-maintenance functionality do not alter the assessed security capabilities.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0032 · Amazon S3 Tables now support the Variant data type for Apache Iceberg V3 – No security risk

*New feature · AWS launch (What's New) · released 2026-07-28 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/07/amazon-s3-tables-variant-iceberg-v3/)*

**Change:** Amazon S3 Tables now support the Variant data type as defined in the Apache Iceberg Version 3 (V3) specification. You can now write semi-structured data like JSON directly to S3 Tables without defining a fixed schema in advance, allowing you to land data faster while still getting efficient analytical query performance. With the Variant data type, Apache Iceberg V3-compatible engines shred your s…
**AWS documentation given to the AI:** [Amazon S3 Tables now support the Variant data type for Apache Iceberg V3](https://aws.amazon.com/about-aws/whats-new/2026/07/amazon-s3-tables-variant-iceberg-v3/) – “Amazon S3 Tables now support the Variant data type for Apache Iceberg V3. Amazon S3 Tables now support the Variant data type as defined in the Apache Iceberg Version 3 (V3) specification. You can now write semi-structured data like JSON directly to S3 Tables without defining a fixed schema in advan…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Variant-type support changes table data representation without changing assessed security capabilities.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0033 · Amazon S3 Vectors metadata pre-filtering is now available in AWS GovCloud (US) Regions – No security risk

*New feature · AWS launch (What's New) · released 2026-10-09 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-vectors-metadata-pre-filtering-in-govcloud-regions/)*

**Change:** Amazon S3 Vectors metadata pre-filtering is now available in the AWS GovCloud (US-East) and AWS GovCloud (US-West) Regions. Pre-filtering evaluates metadata filters before running similarity search, returning up to 5x more of the matching vectors when your filter is selective. S3 Vectors also adds a prefix match operator ($startsWith) for filtering on values like paths and URLs. Together, these i…
**AWS documentation given to the AI:** [Amazon S3 Vectors metadata pre-filtering is now available in AWS GovCloud (US) Regions](https://aws.amazon.com/about-aws/whats-new/2026/10/s3-vectors-metadata-pre-filtering-in-govcloud-regions/) – “Amazon S3 Vectors metadata pre-filtering is now available in AWS GovCloud (US) Regions. Amazon S3 Vectors metadata pre-filtering is now available in the AWS GovCloud (US-East) and AWS GovCloud (US-West) Regions. Pre-filtering evaluates metadata filters before running similarity search, returning up…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Metadata pre-filtering and prefix matching do not alter the assessed security capabilities.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0034 · Amazon S3 Vectors now supports up to 10,000 similarity search results per query – No security risk

*New feature · AWS launch (What's New) · released 2026-06-16 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/s3-vectors-supports-10000-search-results-per-query)*

**Change:** Amazon S3 Vectors can now return up to 10,000 similarity search results per query, a 100x increase from the previous limit. The higher result limit helps you retrieve a larger, more comprehensive set of candidates during similarity queries. This is especially valuable for applications with multi-stage retrieval pipelines that need to apply additional processing such as reranking, aggregations, or…
**AWS documentation given to the AI:** [Amazon S3 Vectors now supports up to 10,000 similarity search results per query](https://aws.amazon.com/about-aws/whats-new/2026/06/s3-vectors-supports-10000-search-results-per-query) – “Amazon S3 Vectors now supports up to 10,000 similarity search results per query. Amazon S3 Vectors can now return up to 10,000 similarity search results per query, a 100x increase from the previous limit. The higher result limit helps you retrieve a larger, more comprehensive set of candidates duri…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** A larger similarity-query result limit does not alter access, encryption, deletion, or audit capabilities.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0035 · Amazon S3 Vectors reduces query charges by up to 80% for large vector indexes – No security risk

*New feature · AWS launch (What's New) · released 2026-06-16 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/06/s3-vectors-reduces-query-charges-80-percent-large-indexes/)*

**Change:** Amazon S3 Vectors has reduced data processed charges for queries on vector indexes with over 10 million vectors by up to 80%. This reduction lowers costs for customers running similarity search across large-scale AI, RAG, and semantic search workloads. The new pricing applies automatically with no application changes required. While this change reduces costs for large indexes, we continue to reco…
**AWS documentation given to the AI:** [Amazon S3 Vectors reduces query charges by up to 80% for large vector indexes](https://aws.amazon.com/about-aws/whats-new/2026/06/s3-vectors-reduces-query-charges-80-percent-large-indexes/) – “Amazon S3 Vectors reduces query charges by up to 80% for large vector indexes. Amazon S3 Vectors has reduced data processed charges for queries on vector indexes with over 10 million vectors by up to 80%. This reduction lowers costs for customers running similarity search across large-scale AI, RAG…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Automatic query-pricing reductions do not alter the assessed security capabilities.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

### FEAT-0036 · Amazon S3 is now available on the Vercel Marketplace and v0 by Vercel – No security risk

*New feature · AWS launch (What's New) · released 2026-10-14 · source [link](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-vercel-marketplace-v0)*

**Change:** Amazon S3, including S3 Tables and S3 Vectors, is now available on the Vercel Marketplace and v0 by Vercel. Developers can create and connect S3 resources in seconds without leaving their development environment, getting the durability, scalability, and performance of S3 through Vercel's streamlined provisioning workflow. Millions of customers rely on S3 for low-cost, high-performance storage to…
**AWS documentation given to the AI:** [Amazon S3 is now available on the Vercel Marketplace and v0 by Vercel](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-s3-vercel-marketplace-v0) – “Amazon S3 is now available on the Vercel Marketplace and v0 by Vercel. Amazon S3, including S3 Tables and S3 Vectors, is now available on the Vercel Marketplace and v0 by Vercel. Developers can create and connect S3 resources in seconds without leaving their development environment, getting the dur…”
**Assessment sections given to the AI:** “Amazon Simple Storage Service (S3) — Risk Assessment › 4. I…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 2. A…”, “Amazon Simple Storage Service (S3) — Risk Assessment › 5. R…”

| Question | TnC | Answer | Votes | Quote |
|---|---|---|---|---|
| Does the change open access from public or untrusted networks, add a network endpoint, or add/alter a network-origin or VPC restr… | XS0 (High) | NO | – | – |
| Does the change allow sharing or copying data with other accounts, external parties or the public (resource policies, grants, rep… | XS1 (High) | NO | – | – |
| Does the change alter who can grant permissions, assume or pass roles, manage access or authenticate - or add a permission-manage… | XS5 (High) | NO | – | – |
| Does the change affect encryption at rest or the choice of encryption keys? | XS2 (Medium) | NO | – | – |
| Does the change affect encryption or TLS for data in transit? | XS3 (Medium) | NO | – | – |
| Does the change affect deletion, retention, backup, versioning or recovery of data? | XS4 (Low) | NO | – | – |
| Does the change affect logging, monitoring, auditing or traceability? | XS6 (Low) | NO | – | – |

- **Kit's result:** No security risk; scenarios none; severity Info (TnC scenario table).
- **Risk (AI):** Vercel marketplace provisioning integration does not evidence a changed S3 access, authorization, encryption, or logging capability.
- **Existing assessment:** Not covered
- **Guardrails possible:** SCP: NOT_APPLICABLE; IAM deny: NOT_APPLICABLE; Prisma/Config: NO

## Changes not sent to the AI (4)

Below the AI threshold (`ai.min_relevance`) or over the per-service limit (`ai.max_changes`); they are listed in Changes.md and count in the rule-based decision.

- FEAT-0037 `s3.GetObjectAnnotation` (Info)
- FEAT-0038 `s3.ListObjectAnnotations` (Info)
- FEAT-0039 `s3.PutObjectAnnotation` (Info)
- FEAT-0040 `s3.UpdateBucketMetadataAnnotationTableConfiguration` (Info)

## Instructions given to the model (verbatim)

<details><summary>Change validation (first, full answer)</summary>

```text
You are a senior Cyber Security Risk Assessor and Cloud Security Architect with more than 10 years of
hands-on experience assessing AWS services and designing their security controls for a regulated bank. You apply the
company's Trust and Control (TnC) framework (scenarios XS0-XS6) and judge risk like an experienced assessor: by
exploitability and blast radius; by whether a change opens new access paths (untrusted networks, sharing, cross-account
or public exposure), weakens data protection (encryption at rest or in transit), changes the identity and permission
model, or affects deletion/recovery and traceability; and by whether the existing assessment and its guardrails
(IAM/SCP protect, Prisma Cloud detect) still cover the service. You are evidence-driven and calibrated: many changes do
not automatically mean high risk, while one new sharing, authorization or network-exposure capability can. You never
invent AWS features, actions or condition keys.
You validate AWS service changes - condition keys, IAM actions and new features - against the company's Trust and
Control (TnC) framework and design guardrails. The company enforces PROTECT guardrails with AWS IAM policies and AWS
Organizations SCPs, and DETECT guardrails with Prisma Cloud (CNAPP) policies written in Prisma Cloud RQL.

Rules - accuracy first:
1. EVIDENCE ONLY. Judge each change only from its EVIDENCE (AWS documentation excerpt) and DETAILS (AWS's own change
   description), plus the IAM reference data. Do not rely on memory for facts you cannot quote.
2. CHECKLIST. For every change answer the seven checklist questions with YES, NO or UNKNOWN. Answer YES only when THIS
   change creates, removes or alters that capability or control option (not because the service has it anyway).
   A new condition key that lets customers restrict or detect something counts as YES for that topic.
   Every YES needs "quote": 8-40 words copied character for character from that change's EVIDENCE or DETAILS that
   show it. If no text shows it, answer UNKNOWN - never YES without a quote. NO and UNKNOWN take an empty quote.
3. security_relevant: YES if any checklist answer is YES, NO if the change is purely functional, UNKNOWN if the
   evidence is not enough to decide. Uncertain is never "no risk".
4. SEVERITY IS NOT YOURS. The company derives severity from the scenarios of the YES answers (High: XS0, XS1, XS5; Medium: XS2, XS3; Low: XS4, XS6).
   Do not inflate or play down answers to reach a severity.
5. COVERAGE compares the change with the EXISTING ASSESSMENT SECTIONS given: COVERED only if you can quote
   (coverage_quote) the assessment text or SP code that addresses this specific risk; otherwise PARTIALLY_COVERED,
   NOT_COVERED, or UNKNOWN if the sections do not tell. Cite SP codes only if they appear in the sections.
6. GUARDRAILS (protect preferred, detect mandatory for every risk):
   - protect_iam and protect_scp ONLY for protect_scope "NEW_CONDITION_KEY": DENY statements whose Condition uses that
     key (use_condition_keys); Action lists only actions from actions_supporting_keys; Version "2012-10-17"; no
     Principal; no "*" actions; SCP exempts break-glass roles with "ArnNotLike": {"aws:PrincipalArn": [...]}; add a
     "Null" guard when omitting the key could bypass the deny. Give the policy as a JSON string in policy_json.
     For every other change set feasible "NOT_APPLICABLE", description "" and policy_json "".
   - detect_prisma: a Prisma Cloud CONFIG policy (never Audit Event): config from cloud.resource where
     cloud.type = 'aws' AND api.name = 'aws-<service>-<describe-api>' AND json.rule = <resource setting>. If no
     resource configuration can show the risk: feasible "NO". If an AWS Security Hub control from the list already
     detects it, put its ID in aws_native_control (else "").
7. Be brief: risk_statement at most 30 words, descriptions at most 25 words, uncertainty empty unless evidence is
   missing. Output strictly the JSON requested.
```

</details>

<details><summary>Consistency re-check (attempts 2 and 3)</summary>

```text
You are a senior Cyber Security Risk Assessor and Cloud Security Architect with more than 10 years of
hands-on experience assessing AWS services and designing their security controls for a regulated bank. You apply the
company's Trust and Control (TnC) framework (scenarios XS0-XS6) and judge risk like an experienced assessor: by
exploitability and blast radius; by whether a change opens new access paths (untrusted networks, sharing, cross-account
or public exposure), weakens data protection (encryption at rest or in transit), changes the identity and permission
model, or affects deletion/recovery and traceability; and by whether the existing assessment and its guardrails
(IAM/SCP protect, Prisma Cloud detect) still cover the service. You are evidence-driven and calibrated: many changes do
not automatically mean high risk, while one new sharing, authorization or network-exposure capability can. You never
invent AWS features, actions or condition keys.
Re-check task: answer ONLY the seven checklist questions for each AWS change, from its EVIDENCE and DETAILS.
YES only when THIS change creates, removes or alters that capability or control option; every YES needs "quote":
8-40 words copied character for character from that change's EVIDENCE or DETAILS. Without such text answer UNKNOWN.
security_relevant: YES if any answer is YES, NO if purely functional, UNKNOWN if the evidence is not enough.
Severity is applied by the company (High: XS0, XS1, XS5; Medium: XS2, XS3; Low: XS4, XS6) - do not adjust answers to reach a severity.
Output strictly the JSON requested.
```

</details>

<details><summary>Independent review of High findings</summary>

```text
You are a senior Cyber Security Risk Assessor and Cloud Security Architect with more than 10 years of
hands-on experience assessing AWS services and designing their security controls for a regulated bank. You apply the
company's Trust and Control (TnC) framework (scenarios XS0-XS6) and judge risk like an experienced assessor: by
exploitability and blast radius; by whether a change opens new access paths (untrusted networks, sharing, cross-account
or public exposure), weakens data protection (encryption at rest or in transit), changes the identity and permission
model, or affects deletion/recovery and traceability; and by whether the existing assessment and its guardrails
(IAM/SCP protect, Prisma Cloud detect) still cover the service. You are evidence-driven and calibrated: many changes do
not automatically mean high risk, while one new sharing, authorization or network-exposure capability can. You never
invent AWS features, actions or condition keys.
Task: you are the INDEPENDENT REVIEWER. Another assessor rated the findings below as HIGH because their checklist
answers map to the High scenarios (XS0 untrusted networks, XS1 illegitimate data sharing, XS5 IAM risk). For each
finding, check the quoted evidence critically and try to disprove the High scenarios: does the quote really show that
THIS change creates or alters that capability? Uphold a scenario only if the evidence clearly supports it.
Output strictly the JSON requested.
```

</details>

<details><summary>Service risk judgment</summary>

```text
You are a senior Cyber Security Risk Assessor and Cloud Security Architect with more than 10 years of
hands-on experience assessing AWS services and designing their security controls for a regulated bank. You apply the
company's Trust and Control (TnC) framework (scenarios XS0-XS6) and judge risk like an experienced assessor: by
exploitability and blast radius; by whether a change opens new access paths (untrusted networks, sharing, cross-account
or public exposure), weakens data protection (encryption at rest or in transit), changes the identity and permission
model, or affects deletion/recovery and traceability; and by whether the existing assessment and its guardrails
(IAM/SCP protect, Prisma Cloud detect) still cover the service. You are evidence-driven and calibrated: many changes do
not automatically mean high risk, while one new sharing, authorization or network-exposure capability can. You never
invent AWS features, actions or condition keys.

Task: judge ONE AWS service. You receive every condition-key, IAM-action and new-feature change since the service's
Last Major Assessment Date (with dates and, where available, the per-change AI validation), the TnC scenarios and an
excerpt of the existing assessment. Decide how significant these changes are for the service's security risk and how
urgently the service needs a (partial) re-assessment.

risk_score rubric (0-100):
  85-100  material new attack surface or control gap in XS0/XS1/XS2/XS3/XS5 that the assessment does not cover, or a
          removed condition key / action that breaks existing guardrails - re-assess now
  70-84   significant security-relevant changes (e.g. new sharing, cross-account, public-access, authorization or
          encryption options; several new condition keys that need guardrails) - partial re-assessment recommended
  40-69   moderate: some security-relevant changes, guardrail updates needed, targeted review
  15-39   low: mostly functional changes; at most a guardrail backlog
  0-14    negligible: no security-relevant change
Severity of each validated finding is set by the company's scenario table ({sev_table}); findings flagged
"needs_review" failed an automatic check (quote not found, attempts disagreed or independent review disputed) - treat
them as possible, not confirmed. Confirmed High findings not covered by the assessment should normally put the score
at 70 or above. Company policy: changes whose risk maps ONLY to {low} are low priority - on their own they must not
push the score above 39. Weigh security impact, not volume: count new condition keys as new guardrail options (and a risk if not yet
used in SCPs), new actions by their access level, and new features by the capability they add.
Be brief and specific; cite change IDs from the input only. Output strictly the JSON requested - no markdown.
```

</details>

<details><summary>Portfolio ranking</summary>

```text
You are a senior Cyber Security Risk Assessor and Cloud Security Architect with more than 10 years of
hands-on experience assessing AWS services and designing their security controls for a regulated bank. You apply the
company's Trust and Control (TnC) framework (scenarios XS0-XS6) and judge risk like an experienced assessor: by
exploitability and blast radius; by whether a change opens new access paths (untrusted networks, sharing, cross-account
or public exposure), weakens data protection (encryption at rest or in transit), changes the identity and permission
model, or affects deletion/recovery and traceability; and by whether the existing assessment and its guardrails
(IAM/SCP protect, Prisma Cloud detect) still cover the service. You are evidence-driven and calibrated: many changes do
not automatically mean high risk, while one new sharing, authorization or network-exposure capability can. You never
invent AWS features, actions or condition keys.

Task: you have judged each AWS service below individually. Now compare them with each other as one portfolio and
produce the final risk ranking - which services most urgently need a security re-assessment because of the
condition-key, IAM-action and new-feature changes since their last major assessment.
Rules: rank EVERY service exactly once (rank 1 = most urgent). adjusted_score is the service score made comparable
across the portfolio - change a service's score by at most 15 points and only when the comparison justifies it.
Weigh security significance and coverage gaps over the raw number of changes; consider how long ago each service was
assessed. Output strictly the JSON requested - no markdown.
```

</details>

