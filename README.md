# Clone Cisco Intersight Server Profiles with Ansible

This repository is focused on one workflow: cloning an existing Cisco Intersight server profile to a new profile name.

## What the playbook does

1. Prompts for a source server profile name (exact match)
2. Prompts for a target profile name (new clone name)
3. Optionally prompts for a target blade serial number (leave blank for assign later)
4. Resolves the source profile via `server/Profiles`
5. Verifies the target profile name does not already exist
6. Sends a clone request to `bulk/MoCloners`
7. If serial is provided, requests profile Deploy via `ScheduledActions`
8. Polls profile RunningWorkflows, then monitors workflow status every 60 seconds until success or failure
9. Prints completion details for clone/deploy actions

## Requirements

- ansible-core >= 2.16
- Python >= 3.12
- `cisco.intersight` Ansible collection

Install the collection:

```bash
ansible-galaxy collection install cisco.intersight
```

## Authentication

Generate an API key pair in the Intersight UI (**Settings -> API Keys**), then export the credentials before running the playbook:

```bash
export INTERSIGHT_API_KEY_ID="<your-api-key-id>"
export INTERSIGHT_API_PRIVATE_KEY="<absolute-path-to-your-Secretkey.txt>"
```

If you use a region-specific Intersight endpoint, override `api_uri` in the playbook vars.

## Files

| File | Description |
|---|---|
| `clone_server_profile.yml` | Main playbook for cloning server profiles |
| `inventory` | Minimal inventory declaring localhost |
| `APIKeyID.txt` | Local key ID file (do not commit) |
| `Secretkey.txt` | Local private key file (do not commit) |

## Usage

Run:

```bash
ansible-playbook -i inventory clone_server_profile.yml
```

You will be prompted for:

- Source server profile name
- Target (new) profile name
- Optional target blade serial number

Assignment behavior:

- If serial is blank: clone is created with Assign Later
- If serial is provided: clone is pre-assigned by serial using Static assignment mode
- If serial is provided: playbook also requests Deploy on the cloned profile
- After deploy request, the playbook checks RunningWorkflows and polls the workflow status once per minute until deploy succeeds or fails

## Edge cases handled

- Source profile not found: fails with a clear message
- Multiple source profiles match: warns and uses the first result
- Target profile already exists: fails and asks for a different target name

## Running in AWX or Tower

`vars_prompt` is not supported in non-interactive job execution. Supply values as extra vars or survey fields:

- `source_profile_name`
- `target_profile_name`
- `target_server_serial` (optional; leave empty for assign later)

Provide Intersight credentials through a secure credential type or encrypted variables.
