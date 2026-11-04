# Project: Malachite

This is the current working brief. The latest version supersedes earlier versions.

## How to help

Act as a technical design partner for Malachite. Help evaluate options, identify dependencies and trade-offs, and plan implementation in manageable steps.

Keep **confirmed decisions**, **proposals**, and **open questions** distinct. Do not present a proposal as settled or fill gaps with unstated assumptions. Preserve decisions already made when updating this brief.

## Goal

Design and build Malachite: a local-first, self-hosted multi-agent system.

Malachite will have five or six fixed agent personas, each with a distinct role and its own Ollama model. The agents will cross-check one another’s work; no agent may review its own work.

There will be no animated talking head. Each persona should have a distinct, memorable voice so I can identify the speaker. The system is primarily for spoken notifications, not voice chat.

## Confirmed direction

- Use a local-first design. Local operation should remain useful without internet access.
- Start with the available Intel NUC hardware and accept slower responses.
- I may temporarily delegate selected tasks to Claude when local hardware is too constrained.
- No remote VPS is planned.
- The system should have five or six fixed roles, each using a different model in Ollama.
- Every agent’s work must be reviewed by another agent, never by its author.
- The target phone is Android.
- Notifications should initially cover task completion and commit creation.
- The animated talking head is out of scope.
- Each persona should eventually have its own distinct voice. The actor/character voice choice is undecided.

## Design inspiration

Use [Léo’s AI Studio](https://pages.laporte.cloud/my-setup/) as inspiration for operating principles:

- Keep agent roles stable while allowing their models to change.
- Use independent checks instead of letting a model grade its own work.
- Keep durable project memory in human-readable files that do not depend on one model.
- Keep final approval for consequential actions with me.

Do not assume the same machines, Apple devices, or budget. Malachite should start with the hardware available and grow only when measurements justify it.

## Local and cloud boundaries

Malachite is local-first, not fully air-gapped:

- Local agents and core services should work without internet access.
- Claude may be used for explicitly delegated tasks. Claude is an external service, not one of the fixed local agent roles.
- Send Claude only the context needed for the delegated task. Do not include credentials or sensitive personal or project information unless I explicitly authorize it.
- Record which work was delegated and what came back.
- Remote phone notifications may also require an internet-connected push service. This service and the information it receives are not yet chosen.

## Hardware and performance

The intended initial platform is a local cluster of approximately three to four Intel NUC units. Their exact specifications are unknown, and more capable hardware may be needed later.

Slow responses are acceptable at first. Measure performance before recommending upgrades. Do not assume that all agent models can remain loaded or run concurrently.

Do not assume an Apple-based setup or NVIDIA DGX Spark-class hardware.

## Proposed agent team

Six roles are a starting proposal; the final team may have five or six agents.

1. **Coordinator:** Breaks requests into tasks, assigns work, tracks dependencies, and combines reviewed results.
2. **Systems Architect:** Defines the system structure, interfaces, and design trade-offs.
3. **Cluster Engineer:** Handles host setup, K3s, storage, networking, and Ansible.
4. **Model and Runtime Engineer:** Selects Ollama models and configures serving, resource allocation, and model routing.
5. **Agent Engineer:** Implements agent workflows and their connections to Ollama and local services.
6. **Quality and Offline Assurance:** Defines checks and evaluates evidence, privacy, offline behavior, and reliability.

Roles should remain stable even if their assigned models change. Each local agent must have a distinct model assignment in Ollama. Model names, sizes, and whether models can stay loaded together remain undecided.

## Cross-review policy

Every agent deliverable must be reviewed by a different agent. The author must never review their own work, including revisions.

Proposed default review cycle:

- Coordinator reviews Quality and Offline Assurance deliverables.
- Quality and Offline Assurance reviews Agent Engineer deliverables.
- Agent Engineer reviews Model and Runtime Engineer deliverables.
- Model and Runtime Engineer reviews Cluster Engineer deliverables.
- Cluster Engineer reviews Systems Architect deliverables.
- Systems Architect reviews Coordinator deliverables.

Assign another qualified reviewer if the default reviewer lacks the expertise for a particular deliverable. Record the author and reviewer. A review is a check that may find problems, not proof that the work is correct.

## Android spoken notifications

Malachite is notification-first rather than primarily a voice-chat system. Voice input is not an initial priority.

When a configured event occurs—initially task completion or commit creation—the responsible agent should prepare a concise update in that persona’s voice. Keep text available, and make notifications configurable by event type. Options to mute, queue, or replay notifications should be considered.

The target phone is Android. The intended behavior is to speak an update even when the app is not open. If Android or the delivery mechanism prevents background speech, show a normal notification with a one-tap option to play it.

A small Android companion app may be needed for background speech. A web app may be used for an earlier prototype, but it must be tested on the actual phone before being treated as sufficient. Validate the design against Android’s current background execution and media playback requirements:

- [Android foreground service types](https://developer.android.com/develop/background-work/services/fgs/service-types)
- [Restrictions on starting foreground services from the background](https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start)

Generate speech locally where practical. Keep notification payloads minimal, and do not send sensitive task details or generated speech to a push service without explicit approval.

The actor/character voices are undecided. Choose appropriate licensed or authorised voice options before implementing them.

## Proposed architecture

The following is a proposal to evaluate, not a confirmed design:

- **Cluster:** K3s across the NUCs, with one control-plane node and the remaining nodes as workers.
- **Storage:** Longhorn for shared or replicated storage, if it fits the final hardware and workload.
- **LLM serving:** Ollama deployed in the cluster and exposed through an internal Kubernetes service.
- **Agent layer:** Python services that route work to the assigned Ollama models. LangChain and Agno are candidates; the framework is undecided.
- **Deployment:** Ansible for repeatable provisioning and deployment.

Validate model storage, resource requirements, scheduling, concurrency, and failure handling before deciding how Ollama should run across the cluster. Do not assume that multiple Ollama replicas will improve performance.

## Open questions

### Product and privacy

1. What exact offline boundary is required for normal operation, Claude delegation, and remote phone notifications?
2. Which task information, if any, may be sent to Claude?
3. Which notification events should speak, and what should quiet hours and interruption levels be?
4. Which licensed or authorised voice option should be used for each persona?

### Agents and models

5. Should the team have five or six agents, and which roles should be combined if there are five?
6. Which Ollama model should serve each role?
7. What response quality, latency, and concurrency are acceptable?
8. Should models be loaded on demand or kept resident when hardware permits?

### Hardware and platform

9. What are the NUC CPU, RAM, storage, and accelerator specifications?
10. Which Linux distribution should run on the NUCs? Ubuntu Server and Debian are candidates. Void Linux is preferred by the project owner but may have compatibility or support trade-offs to evaluate.
11. Is K3s appropriate for the first version, or should a simpler deployment be tested first?
12. What storage needs to be shared or replicated, and what failure tolerance is required?
13. Which Android notification delivery mechanism should be used, especially when the phone is away from home?
14. Can the Android phone reliably play persona-specific speech while the app is closed?

## Suggested order of work

1. Inventory the NUCs and define the offline, Claude, and notification boundaries.
2. Confirm the number and responsibilities of the agents.
3. Select candidate models and estimate compute, memory, storage, and latency needs.
4. Choose the Linux distribution and deployment approach.
5. Prototype local inference and cross-review on the available hardware.
6. Prototype Android notifications and test spoken playback with the app closed.
7. Write the Ansible playbooks and Kubernetes manifests after the prototypes validate the main choices.