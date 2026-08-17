# Waves testing

This file is identical across all `vedicon/*` repositories - it gives the
same cross-repo view from wherever you land. It indexes the umbrella
tracking issues per release wave; each umbrella contains the full detail
(PRs, sub-issues, commits, integration test plan) for its repo.

A WAVE is a deploy-test identifier, not a release version. It groups issues
that were validated end-to-end by deploying one or more scenarios on a real
Proxmox - not just quick fixes merged without test coverage. Date = final
test sign-off (YYYY-MM-DD). No date = wave still in progress.

A wave can have 1 or several umbrellas - typically one per repo touched by
the wave (e.g. WAVE_01 has 3 umbrellas across 3 repos). A single umbrella
covering cross-repo work is also valid (e.g. WAVE_02 has 1 umbrella on
vedicon_playbook that also tracks devkit work via SHA refs in its body).

## WAVE_03 - 2026-06-04

- vedicon/vedicon-vedicon_playbook#62
- vedicon/vedicon#174
- vedicon/vedicon-ansible_roles-debug-devkit#110
- vedicon/vedicon-catalog#164

## WAVE_02 - 2026-05-22

- vedicon/vedicon-vedicon_playbook#55

## WAVE_01 - 2026-05-20

- vedicon/vedicon#162
- vedicon/vedicon-vedicon_playbook#49
- vedicon/vedicon-ansible_roles-proxmox_controller#98
