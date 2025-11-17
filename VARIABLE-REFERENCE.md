# IdM Variable Reference - rhis-builder-idm

**Purpose:** Explain IdM/IPA variable naming conventions and collection requirements
**Last Updated:** 2025-11-16

---

## Table of Contents
1. [Variable Naming Overview](#variable-naming-overview)
2. [Collection-Required Variables](#collection-required-variables-do-not-rename)
3. [Custom Variables](#custom-variables-can-be-standardized)
4. [Shared Variables](#shared-variables-used-by-other-repos)
5. [Variable Reference Table](#variable-reference-table)

---

## Variable Naming Overview

This repository uses **four different variable prefixes** for Identity Management:

| Prefix | Purpose | Can Change? | Example |
|--------|---------|-------------|---------|
| `ipaserver_*` | Red Hat IdM collection - server installation | **NO** | `ipaserver_domain` |
| `ipaadmin_*` | Red Hat IdM collection - admin authentication | **NO** | `ipaadmin_principal` |
| `ipa_*` | Custom project variables | **Maybe** | `ipa_admin_password_vault` |
| `idm_*` | Alternative custom prefix (not currently used) | **Yes** | `idm_custom_var` |

**Why so many prefixes?**

The Red Hat `redhat.rhel_idm` Ansible collection requires specific variable names for its roles and modules. We cannot change these without breaking the collection. Our custom variables use the `ipa_*` prefix for historical consistency.

---

## Collection-Required Variables (DO NOT RENAME!)

These variables come from the `redhat.rhel_idm` Ansible collection and **MUST** keep their exact names.

### ipaserver_* Variables (Server Installation)

Used by the `redhat.rhel_idm.ipaserver` role for installing the IdM server.

| Variable | Required? | Description | Example Value |
|----------|-----------|-------------|---------------|
| `ipaserver_domain` | **YES** | DNS domain for IdM | `"example.ca"` |
| `ipaserver_realm` | **YES** | Kerberos realm | `"EXAMPLE.CA"` |
| `ipaserver_hostname` | NO | Server hostname (auto-detected if omitted) | `"idm1.example.ca"` |
| `ipaserver_ip_addresses` | NO | IP addresses for the server | `["192.168.140.5"]` |
| `ipaserver_setup_dns` | NO | Install DNS server | `true` |
| `ipaserver_setup_ca` | NO | Install CA | `true` |
| `ipaserver_setup_kra` | NO | Install KRA (key recovery) | `true` |
| `ipaserver_setup_adtrust` | NO | Install AD trust | `true` |
| `ipaserver_auto_forwarders` | NO | Auto-configure DNS forwarders | `true` |
| `ipaserver_no_dnssec_validation` | NO | Disable DNSSEC validation | `true` |
| `ipaserver_forwarders` | NO | DNS forwarders | `["8.8.8.8", "8.8.4.4"]` |

**Source:** `redhat.rhel_idm.ipaserver` role
**Documentation:** https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/installing_identity_management/assembly_installing-an-ipa-server-using-an-ansible-playbook_installing-identity-management

---

### ipaadmin_* Variables (Authentication)

Used by IdM Ansible modules for authentication.

| Variable | Required? | Description | Example Value |
|----------|-----------|-------------|---------------|
| `ipaadmin_principal` | **YES** | Admin principal name | `"admin"` |
| `ipaadmin_password` | **YES** | Admin password | `"SecurePassword123"` |

**Source:** All `redhat.rhel_idm` modules (ipauser, ipagroup, ipahbacrule, etc.)
**Documentation:** https://github.com/freeipa/ansible-freeipa

**Example usage in tasks:**
```yaml
- name: Ensure user exists
  redhat.rhel_idm.ipauser:
    ipaadmin_principal: "{{ ipaadmin_principal }}"
    ipaadmin_password: "{{ ipaadmin_password }}"
    name: testuser
    state: present
```

---

## Custom Variables (Can Be Standardized)

These are **project-specific** variables we control. They could be renamed, but have high impact due to widespread use.

### ipa_* Custom Variables

| Variable | Type | Used By | Description | Recommendation |
|----------|------|---------|-------------|----------------|
| `ipa_admin_password_vault` | vault | IdM, Satellite, AAP, Day-2 | Admin password from vault | Keep (widely used) |
| `ipa_dm_password_vault` | vault | IdM | Directory Manager password | Keep (widely used) |
| `ipa_server_fqdn` | string | IdM, Satellite, AAP, Day-2 | IdM server hostname | Keep (widely used) |
| `ipa_principal_username_vault` | vault | Various | Alias for ipaadmin_principal | Keep (compatibility) |
| `ipa_principal_password_vault` | vault | Various | Alias for ipaadmin_password | Keep (compatibility) |

**Why keep `ipa_*` instead of renaming to `idm_*`?**

1. **Widespread Use:** These variables are referenced across 4 repositories (IdM, Satellite, AAP, Day-2-Ops)
2. **Low Confusion:** While we have both `ipa_*` and `ipaserver_*`, the context makes it clear which is which
3. **Minimal Benefit:** Renaming would require changes in 4 repos for minimal clarity gain
4. **Breaking Changes:** High risk of missing references during migration

**Decision for MVP:** Keep existing `ipa_*` custom variables. For **new** custom variables, prefer `idm_*` prefix to distinguish from collection variables.

---

## Shared Variables (Used by Other Repos)

These variables are defined in IdM but used by other repositories:

### ipa_server_fqdn

**Defined in:** rhis-builder-idm
**Used by:** Satellite, AAP, Day-2-Ops

**Purpose:** Other components need to know the IdM server address for:
- Satellite: IdM realm integration
- AAP: LDAP authentication
- Day-2-Ops: Ongoing management

**Example:**
```yaml
# In rhis-builder-idm/host_vars/idm.example.ca/main.yml
ipa_server_fqdn: "idm1.example.ca"

# Referenced in rhis-builder-satellite for realm integration
# Referenced in rhis-builder-aap for LDAP configuration
```

---

### ipa_admin_password_vault

**Defined in:** Vault file
**Used by:** IdM, Satellite, AAP, Day-2-Ops

**Purpose:**
- IdM: Sets admin password during installation
- Satellite: Authenticate to IdM for realm integration
- AAP: LDAP bind password for authentication
- Day-2-Ops: Ongoing IdM management

**Flow:**
```
vault file: ipa_admin_password_vault
    ↓
IdM: Sets admin@EXAMPLE.CA password
    ↓
Satellite: Uses to join IdM realm
    ↓
AAP: Uses for LDAP authentication
```

---

## Variable Reference Table

### Complete Variable List by Category

#### Installation (Collection-Required)

| Variable | Type | Source | Can Rename? |
|----------|------|--------|-------------|
| `ipaserver_domain` | string | redhat.rhel_idm | **NO** |
| `ipaserver_realm` | string | redhat.rhel_idm | **NO** |
| `ipaserver_hostname` | string | redhat.rhel_idm | **NO** |
| `ipaserver_setup_dns` | boolean | redhat.rhel_idm | **NO** |
| `ipaserver_setup_ca` | boolean | redhat.rhel_idm | **NO** |
| `ipaserver_setup_kra` | boolean | redhat.rhel_idm | **NO** |
| `ipaserver_setup_adtrust` | boolean | redhat.rhel_idm | **NO** |
| `ipaserver_auto_forwarders` | boolean | redhat.rhel_idm | **NO** |
| `ipaserver_forwarders` | list | redhat.rhel_idm | **NO** |

#### Authentication (Collection-Required)

| Variable | Type | Source | Can Rename? |
|----------|------|--------|-------------|
| `ipaadmin_principal` | string | redhat.rhel_idm | **NO** |
| `ipaadmin_password` | string | redhat.rhel_idm | **NO** |

#### Custom (Project-Specific)

| Variable | Type | Source | Can Rename? | Recommendation |
|----------|------|--------|-------------|----------------|
| `ipa_admin_password_vault` | vault | Custom | Yes | Keep (widely used) |
| `ipa_dm_password_vault` | vault | Custom | Yes | Keep (widely used) |
| `ipa_server_fqdn` | string | Custom | Yes | Keep (widely used) |
| `ipa_principal_username_vault` | vault | Custom | Yes | Keep (alias) |
| `ipa_principal_password_vault` | vault | Custom | Yes | Keep (alias) |

---

## Best Practices Going Forward

### For New Variables

When adding **new** variables to this repository:

1. **Collection variables:** Use exact names required by `redhat.rhel_idm`
2. **Custom variables:** Prefer `idm_*` prefix to distinguish from collection
3. **Internal variables:** Use `__idm_*` prefix (double underscore per Red Hat CoP)

**Example:**
```yaml
# Collection-required (must use exact name)
ipaserver_domain: "example.ca"

# Custom user-facing variable (new standard)
idm_custom_setting: "value"

# Internal/private variable (implementation detail)
__idm_temporary_fact: "internal-use-only"
```

---

## Red Hat Community of Practice Compliance

This variable structure aligns with Red Hat CoP guidelines:

✅ **Collection compatibility:** Preserves required variable names
✅ **Clear distinction:** Different prefixes indicate different sources
✅ **No single underscore:** User-facing variables don't use `_` prefix
✅ **Documentation:** This file explains the complexity

**Reference:** https://redhat-cop.github.io/automation-good-practices/#_supporting_multiple_distributions_and_versions

---

## Troubleshooting

### Error: "ipaserver_domain is required"

**Cause:** Missing or misspelled collection-required variable

**Solution:** Ensure exact variable name in your host_vars:
```yaml
ipaserver_domain: "your-domain.com"  # NOT idm_domain or ipa_domain
```

### Error: "ipaadmin_password is required"

**Cause:** IdM modules need authentication

**Solution:** Ensure you're setting the collection variable:
```yaml
ipaadmin_principal: "admin"
ipaadmin_password: "{{ ipa_admin_password_vault }}"
```

Note: We map our vault variable to the collection variable.

---

## References

- [Red Hat IdM Ansible Collection Docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/installing_identity_management/assembly_installing-an-ipa-server-using-an-ansible-playbook_installing-identity-management)
- [ansible-freeipa GitHub](https://github.com/freeipa/ansible-freeipa)
- [Red Hat CoP - Automation Good Practices](https://redhat-cop.github.io/automation-good-practices)
- [Variable Migration Plan](../VARIABLE-MIGRATION-PLAN.md)
- [Vault Variables Registry](../VAULT-VARIABLES.md)

---

**End of Variable Reference**
