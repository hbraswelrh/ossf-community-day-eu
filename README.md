# ossf-community-day-eu

**Talk:** From First PR to Hardening Guide: Structured Security with Gemara

## 2026 Security Slam 

Participants were tasked with a number of objectives to obtain badges. The Inspector Badge could be obtained by performing a Gemara-compatible Threat Assessment and documenting the artifact(s) in SECURITY_INSIGHTS.yml. 

### The result? 
1. Strengthened security posture through security self-assessment.
2. Project maintainer collaboration and critical thinking to nail-down their project `Capabilities` and `Threats`.
3. Established a foundation for new contributors and project adopters to reference existing security considerations without the need for seasoned expertise.
4. Inspector badge, naturally. 

## How they did it?

1. Defined what their project could do in a Capability Catalog.

<details>
  <summary>Capability Catalog Entry</summary>

```yaml
  - id: CNPG.CP04
    title: Declarative Database Objects
    group: data-management
    description: |
      The operator manages declarative databases, publications, and
      subscriptions as dedicated CRDs, and managed roles within the
      Cluster spec. These are reconciled continuously to match the
      desired state.
```

</details>
</n>

2. Defined what could potentially go wrong based on those Capabilities in a Threat Catalog.

<details>
  <summary>Threat Catalog Entry</summary> 

```yaml

  - id: CNPG.TH02
    title: Kubernetes Secret Exposure
    group: credential-and-cryptography
    description: |
      Database credentials, replication certificates, cloud provider
      credentials for object store access (AWS keys, Azure storage
      keys, GCP application credentials), and other sensitive material
      are stored in Kubernetes Secrets, which are base64-encoded but
      not encrypted at the application level. Compromise of etcd, RBAC
      misconfiguration, or excessive permissions on secrets in the cluster
      namespace could expose all credentials simultaneously. The operator does
      not provide built-in credential rotation. Users are responsible for
      enabling etcd encryption at rest, restricting RBAC on secrets, using
      external secret managers if needed, and implementing credential rotation
      procedures.
    capabilities:
      - reference-id: CNPG.CAPABILITIES
        entries:
          - reference-id: CNPG.CP01
          - reference-id: CNPG.CP03
          - reference-id: CNPG.CP04 # From Capability Catalog Entry
      - reference-id: CCC
        entries:
          - reference-id: CCC.Core.CP06
```
</details>
</n>

3. Provided the foundation for Control Catalog development as a Hardening Guide to demonstrate their project aligns with compliance frameworks for regulated end-users.

<details>
  <summary>Control Catalog Entry</summary>

```yaml
- id: CNPG.HC05
    title: Protect and Rotate Kubernetes Secrets
    objective: >
      Kubernetes Secrets containing database credentials, cloud provider
      keys, and TLS material are encrypted at rest, access-controlled,
      and subject to periodic rotation.
    group: credential-management
    state: Active
    guidelines:
      - reference-id: CNSC
        entries:
          - reference-id: CNSC-1
            remarks: >
              Secrets are injected at runtime.
          - reference-id: CNSC-16
            remarks: >
              Secrets should have a short expiration period or TTL.
          - reference-id: CNSC-19
            remarks: >
              Long-lived secrets adhere to periodic rotation and revocation.
          - reference-id: CNSC-20
            remarks: >
              Secrets distributed through secured channels commensurate
              with the data they protect.
          - reference-id: CNSC-21
            remarks: >
              Runtime-injected secrets are masked or dropped from logs.
          - reference-id: CNSC-47
            remarks: >
              Native secret stores are not configured for base64
              encoding or stored in clear-text.
    threats:
      - reference-id: CNPG.THREATS
        entries:
          - reference-id: CNPG.TH02 # From Threat Catalog Entry
            remarks: >
              All credentials stored in Kubernetes Secrets (base64-encoded,
              not encrypted at application level). Compromise of etcd
              or RBAC misconfiguration exposes all credentials.
      - reference-id: CCC
        entries:
          - reference-id: CCC.Core.TH18
            remarks: >
              Key generation may not be under operator control when
              using BYO certificates or cert-manager.
    assessment-requirements:
      - id: CNPG.HC05.AR01
        text: >
          The Kubernetes cluster MUST have etcd encryption at rest
          enabled for Secret resources.
        applicability:
          - production
        state: Active
        recommendation: >
          Configure the EncryptionConfiguration resource with an
          aescbc or secretbox provider for the secrets resource.
      - id: CNPG.HC05.AR02
        text: >
          Database credential Secrets MUST be rotated at least every
          90 days, and a documented rotation procedure MUST exist.
        applicability:
          - regulated
        state: Active
        recommendation: >
          Use an external secrets operator (e.g., External Secrets
          Operator) or a documented manual rotation procedure that
          updates the Secret and performs ALTER ROLE to change
          the password.
      - id: CNPG.HC05.AR03
        text: >
          Cloud provider credential Secrets used for object store
          access (backup/WAL archiving) MUST use short-lived
          credentials or workload identity where the cloud provider
          supports it.
        applicability:
          - production
        state: Active
        recommendation: >
          Use IAM Roles for Service Accounts (IRSA) on AWS,
          Workload Identity on GCP, or Azure AD Workload Identity
          instead of static access keys.

```
</details> 
</n>


## Now what?

## How this impacts the contributor journey

1. When a new contributor looks at the project they can look at the running catalog of capabilities.
    * The `CapabilityCatalog` serves as a consistent and well-maintained source-of-truth for how the project has evolved over time. What already exists, alignment with the documentation, and supporting capabilites from other projects.
2. Awareness of potential threats to the project based on the capabilities it has
    * The `ThreatCatalog` provides insight on the threats that could be introduced as a result of the capabilities further strengthening the contributor's understanding of the project and its security considerations. 

## Resources

* Gemara MCP Server:
* Cloud Native PostGreSQL GitHub Repository:
* CNCF Cloud Native Security Controls Catalog Initative

## Related Talks

* OpenSSF Community Day NA Minneapolis, MN 
    * Speakers: Jenn Power, Red Hat & Hannah Braswell, Red Hat 
    * Recording: [The GRC Architecture You Didn't Know You Built](https://www.youtube.com/watch?v=eGWm29YtEJw)