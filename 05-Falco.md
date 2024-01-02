### 1. Behaviour Analytics

## Falco Overiew

## Falco Installation
In K8s, Falco can be installed as a service in the nodes or it can be run as a DaemonSet.

When run as a package on the nodes, use the command to view the alerts generated
```
journalctl -fu falco
```

For installing as DaemonSets, we can utilize falco helm chart

```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco
```
## Falco Rules
Format of a rule in rules.yaml
```
- rule: <name of the rule>
  desc: <description>
  condition: <filter events to matching the rule>
  output: <output to be logged when event matches>
  priority: <severity>

- rule: Detect shell inside container
  desc: Alert if a shell like bash was spawned inside the container
  condition: container (//This is a macro) and proc.name in (linux_shells)  --> Matches value in list defined below. We have used macro & list defined below
  output: Bash opened (user=%user.name container=%container.id)
  priority: WARNING

- list: linux_shells  --> This is a LIST
  items: [bash, sh, zsh, ksh]

- macro: container
  condition: container.id != host
```

The supported rule fields can be found at https://falco.org/docs/reference/rules/supported-fields/

Falco implements several default rules. 

## Falco COnfiguration File
The main Falco configuration file is defined in the `/etc/falco/falco.yaml`
This configuration has all the info such as:
 - location of the rules file
 - formatting for the output
 - output channels

Rules:

```
rules_file:
  - /etc/falco/falco_rules.yaml  ---> DEFAULT rules file
  - /etc/falco/falco_rules.local.yaml
  - /etc/falco/rules.d
```

If same rules are defined in multiple files, the one the bottom takes precedence. As such, if we want to modify a built-in rule, instead of modifying `falco_rules.yaml`, create a new file and update there.

Output Channels:
```
stdout_output:   --> DEFAULT
  enabled: true

file_output:
  enabled: true
  filename: /tmp/falco.log

program_output: --> POST to external progreams
  enabled: true
  program: "jq '{ text: .output} | curl -d @- -X POST https://hooks.slack.com/svc/xxxx'"

http_output:
  enabled: true
  url: https://url/path
```

For any changes to rule/configuration to take effect, Falco must be restarted.

