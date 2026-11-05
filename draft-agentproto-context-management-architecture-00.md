---
title:
  "Context and Correlation Management Architecture for Distributed Agentic
  Activities"
abbrev: "Context Management Architecture"
category: info

docname: draft-agentproto-context-management-architecture-00
submissiontype: IETF
number:
date:
consensus: true
v: 0
area: Internet
workgroup: agentproto
keyword:
  - AI Network
  - Agentic Networks
  - AI inference
  - AI training

author:
  - fullname: Hesham Moussa
    organization: Huawei Canada
    email: hesham.moussa@huawei.com

  - fullname: Arashmid Akhavain
    organization: Huawei Canada
    email: arashmid.akhavain@huawei.com

contributor:
  - fullname: Hesham Moussa
    organization: Huawei Canada
    email: hesham.moussa@huawei.com
  - fullname: Arashmid Akhavain
    organization: Huawei Canada
    email: arashmid.akhavain@huawei.com
  -

normative:

informative:
---

--- abstract

Agentic applications increasingly distribute a single objective across
autonomous agents, tools, and services implemented by different vendors,
operated by different organizations, and connected through different
protocols.

A single activity may contain multiple pairwise dialogs, multiple tasks
within each dialog, tool invocations, derived resources, downstream
delegations, and later interactions that reinterpret or reuse artifacts
produced earlier in the activity. Existing protocols provide useful
local identifiers for their own interactions, but those identifiers
do not by themselves provide a common information model for correlating
independently maintained dialogs, tasks, and resources across protocol
and administrative boundaries.

This document defines an information model consisting of
Activities, Dialogs, Tasks, Resources, and typed Relationships. It distinguishes Activity correlation from protocol-local
Dialog context, and distinguishes stable resource identity and
provenance from locally defined interpretations.

As such, this document develops a context-and-correlation information-model architecture,
not a functional reference architecture or a wire protocol. It identifies relevant objects and relationships, relevant to maintaining
coherent context, and examines the implications for preserving and propagating their
correlations. It does not define the procedures by which
applications execute work. The model distinguishes the logical extent
of an Activity, which may extend beyond protocol-managed correlation; extending that
coverage is selective, while maintaining established managed
correlations entails responsibilities.

--- middle

# Introduction

Distributed agentic systems are increasingly composed of multiple independently operated participants.

A single Activity can involve:

- Invocation by users, agents, or other invoking systems;
- One agent delegating work to another;
- Several independent agent-to-agent Dialogs;
- Agent-to-tool interactions;
- Resources produced in one branch and consumed in another;
- Downstream delegation hidden from upstream participants;
- Modality transformations, such as video to audio to text;
- Multiple vendors and administrative domains;
- Local protocol constructs that do not share identifiers or state; and
- Operation over different transport protocols.

No single participant, protocol, or identifier spans all of these
interactions.

Multiple such Activities may proceed concurrently while sharing the
same underlying distributed agentic system, giving rise to multiple
intersecting interaction waves composed of Dialogs, Tasks,
conversations, artifacts, Resources, and other related objects. These
waves may overlap, share participants, pursue competing objectives,
and serve different users and goals. Under these circumstances, the
challenge is to ensure that such waves can propagate across the
system without causing significant confusion, loss, dropped
interactions, or delays in service. In essence, the objective is to
preserve sufficient correlation for independently maintained Dialogs,
Tasks, and Resources to remain meaningfully related, such that value of distributed agentic networks can be materialized.

Existing protocols provide useful local identifiers for their own
interactions, but those identifiers cannot, by themselves, correlate independently maintained objects across Dialogs and administrative domains. This document develops an information model for that problem and examines the correlations that need to be understood, preserved, or propagated across those boundaries.

This information-model architecture has a different purpose from a
functional reference architecture that positions discovery, identity,
authentication, authorization, dialog management, and transport as
building blocks of agentic communications. This document concentrates
on what must be correlated among Activities, Dialogs, Tasks, and
Resources, which are objects defined in this document as part of the architecture, and on how those correlations can remain coherent across
independently maintained interactions. These concepts support reasoning
about dialog context; their inclusion does not, by itself, require
each concept or relationship to be standardized as a protocol object.

An Activity can extend beyond the correlations maintained by any one
protocol. Participants may choose whether to establish correlation with
additional Dialogs, but those that undertake to maintain a correlation
within a protocol-managed scope are responsible for preserving its
coherence as relevant information changes. A failure to maintain such
an established correlation differs from an undisclosed interaction
that was never included in that protocol's managed scope.

The remainder of this document is organized as follows. Section 2
states the core architectural principle, and Section 3 defines the
terminology used throughout the document. Section 4 introduces the
architectural model and illustrates it through motivating examples
that cross organizational boundaries. Section 5 examines how
multiple independently maintained Dialogs can coexist within a
single Activity, and Section 6 zooms in to the Tasks within a
Dialog. Section 7 introduces correlation context and the
implications of this architecture for its propagation. Section 8
describes the failure modes that arise from incorrect or
insufficient propagation. Sections 9 to 16 present a worked
propagation example, the relationship taxonomy, privacy
considerations across administrative boundaries, and the
relationship of this architecture to existing protocols and related
work. Sections 17 to 19 conclude with architectural requirements,
out-of-scope items, and open questions.

This document distinguishes settled architectural principles from
open design questions. The open questions are deliberate: they
identify the decision space reserved for the dialog-management
protocol and for coordination with related work. They are
candidates for resolution in subsequent protocol documents, not
gaps in the architecture.


# Core Architectural Principle

The architecture rests on a single central principle:

> Identify the objects and relationships needed to keep independently
> maintained Dialogs, Tasks, and Resources correlated across protocol
> and administrative boundaries. Define their correlation semantics
> without replacing protocol-local mechanisms, and distinguish
> relationships maintained locally from information that needs to be
> propagated for interoperability.

The model uses five primary objects:

~~~
Activity
Dialog
Task
Resource
Relationship
~~~

The model also defines four participant roles:

~~~
Activity Initiating Entity (AIE)
First Next Entity (FNE)
Assistant Agent (AA)
Assistant Tool (AT)
~~~

Roles are relative to the invocation chain: the AIE initiates an Activity, its directly invoked entity is the FNE, and entities invoked downstream are Assistant Agents or Assistant Tools. Terminology follows below.

This information model complements protocol-local context
management mechanisms. A dialog-management protocol may use the
model's semantics and convey relevant Correlation Metadata through
bindings to existing protocols; this architecture does not define
those bindings.

# Terminology

- Activity: is the root of an interaction wave, initiated by a single
  Activity Initiating Entity (AIE) to pursue an objective and reach an
  outcome. An Activity is initiated when the AIE invokes a single First
  Next Entity (FNE), establishing the Activity's initial Dialog. It
  groups the Dialogs and their Tasks, together with the Resources
  involved, for correlation purposes. This grouping does not imply a
  shared conversational context or a shared participant set. An AIE
  may initiate multiple Activities, and an Activity can involve
  multiple tools and protocols.

  - Examples:

    ~~~
    Analyze video V1
    Plan a trip
    Investigate a network fault
    Prepare a financial report
    ~~~

- Dialog: is a logical container established among a set of two or
  more participants that share a conversational context. It belongs to
  exactly one Activity, and is created to pursue an objective that
  contributes to that Activity. Within the Dialog, one or more Tasks
  may run, each pursuing a part of the Dialog's objective. A Dialog is
  related to its Activity and to other Dialogs, Tasks, and Resources of
  that Activity through correlation metadata. The metadata also records
  the Dialog's lifecycle state. A Dialog is frequently pairwise, but MAY
  involve more than two participants. The same set of participants may
  maintain several concurrent Dialogs, within the same Activity or
  across different Activities. Each Dialog is identified by its own
  identity, not by its participant set. Every Dialog has its own
  objective, conversational context, and lifecycle state.


- Task: is a bounded unit of work within a Dialog, created to pursue an
  objective that contributes to the Dialog's objective. A Task
  progresses through one or more Events, which advance that objective.
  A Task has its own lifecycle state.

  Example: as part of the Activity "plan a trip", Agent A and Agent
  B work together to generate a shortlist of suitable hotels in a
  city within the trip's destination. Generating that shortlist is
  the objective of the Dialog between Agent A and Agent B. Within
  that Dialog, retrieving and evaluating each hotel is a bounded unit
  of work, that is, a Task. Within each Task, Agent A and Agent B
  carry out Events - for example, request hotel X's details, clarify
  its breakfast arrangement, or check availability - and those Events
  share the Task's identifier as they progress.

- Event: is a message, a job invocation, a tool call, or a similar
  interaction carried out by the participants as they pursue a Task's
  objective. An Event has no objective of its own. It is part of
  carrying out the Task. An Event belongs to exactly one Task and
  carries that Task's identifier as it progresses. Events are local
  to their Task and are not correlation objects.

- Resource: is an identifiable object that participants consume,
  produce, refer to, or transform during an Activity, such as video,
  audio, text, documents, transcripts, tool outputs, and
  agent-produced artifacts. The same Resource can be used in more than
  one Activity. Each Activity may refer to it differently, and so may
  its Dialogs, Tasks, and Events. The architecture does not prescribe a
  particular method of referencing Resources. Inside an Activity, any
  Dialog, Task, or Event can refer to the Resource, and the same
  Resource can be referred to by several of them simultaneously.
  Resources can also be physically carried within Events.

- Relationship: is a typed, directional association among
  Activities, Dialogs, Tasks, or Resources. A relationship expresses
  an association between its endpoints. Relationship types express the
  meaning of those associations. Relationships types are Correlation
  Metadata. A relationship may exist and be retained without its type being
  disclosed (disclosure of relationship types is a decision for the endpoints to make). 
  The set of relationship types is not to be finalized by this document, the following presents some examples of relationship types. The dialog-management
  protocol (protocol design draft) should capture the definition, implementation, and the
  detailed enumerated list of the relationship types. Examples
  include:

  ~~~
  belongs-to
  delegated-from
  derived-from
  input-to
  output-of
  ~~~

- Upstream and Downstream: are directional terms relative to a given
  Dialog. Upstream refers to Dialogs from which work was delegated, in
  the direction of the Activity Initiating Entity (AIE). Downstream
  refers to Dialogs created by delegating work, in the direction away
  from the AIE. Delegation proceeds downstream. Results, status, and
  correlation updates proceed upstream.

- Context: in this document, refers to the set of relationships
  (correlations) among Activities, Dialogs, Tasks, and Resources, and
  to the Correlation Metadata that describes them. Context does not
  refer to the  internal state maintained by an AI agent (sometimes
  called a "context window"). Agent-internal memory and its content is out of scope.
  Context as defined here is also distinct from the conversational/event context that a Dialog's participants share. Whether event context should be regarded as part of the context is for the working group to ratify.

  The following should also be noted: Context, as defined in this document, represents an architectural correlation model and is broader than the Dialog context defined in the agentproto charter. The inclusion of an object or relationship in this model does not imply that it must be standardized or exchanged by the dialog-management protocol.

- Correlation Metadata: is the information that participants may
  exchange to keep independently maintained Activities, Dialogs,
  Tasks, and Resources correlated and to enable the establishment of a coherent context.
  Correlation metadata consists of different kinds of assertions, examples include: relationships,
  such as "D2 delegated-from D1" or "T1 derived-from V1", and
  lifecycle state, such as "D5 revoked". It does not include
  Resource content or an agent's internal state or memory.

- Protocol-Local Context: is the context establishment mechanism defined by the
  application protocol used within a Dialog. For example, an A2A
  `contextId` or `taskId`, etc are a Protocol-Local Context.

- Dialog Management: The correlation and lifecycle management of an
  Activity's Dialogs and Tasks, including the correlation of the
  Resources they use and the Activity that groups them. Although
  named after the Dialog as the fundamental unit of interaction,
  Dialog Management is not limited to Dialogs. A dialog-management
  protocol supports these functions and enables participants to
  exchange the information needed to maintain these correlations.

- Resource Binding: describes a Resource's use or interpretation within
an Activity, Dialog, or Task.

- Interaction Wave: is the dynamically expanding set of Dialogs,
  Tasks, and Resources within an Activity as work is delegated and
  executed across participants. It is rooted at the AIE and begins
  with the initial Dialog between the AIE and the FNE invoked for
  that Activity. Subsequent invocations within the Activity extend
  the wave through additional Dialogs.

- Activity Initiating Entity (AIE): is the entity that starts an
  Activity and forms the root of the corresponding interaction wave.
  The AIE is the originator of the Activity. An AIE can be a user, an
  agent, or an invoking system. A tool cannot be an AIE.

- First Next Entity (FNE): An entity directly invoked by an AIE to
  pursue the objective of a newly initiated Activity. Each AIE-to-FNE
  invocation establishes a distinct Activity and its initial Dialog.
  An AIE may initiate multiple Activities by invoking the same or
  different entities. An FNE may be an application, an agent, or a tool.
  The FNE designation reflects its role in the Activity rather than
  its entity type. An FNE may invoke Assistant Entities through
  additional Dialogs within that Activity. An FNE that is a tool
  cannot invoke other entities.


- Assistant Entity: An entity invoked by an FNE or another Assistant
  Entity as part of an Activity. The invoking entity and the Assistant
  Entity interact through a Dialog associated with that Activity. An
  Assistant Entity may be an Assistant Agent (AA) or an Assistant Tool
  (AT). Applications cannot serve as Assistant Entities because they
  can only be invoked by an AIE, and therefore can only serve as FNEs.

- Assistant Agent (AA): An Assistant Entity that is an AI agent. An
  AA interacts with its invoking entity through a Dialog and may
  invoke other Assistant Entities (AAs or ATs) to assist in completing
  Tasks assigned to it within the Activity.

- Assistant Tool (AT): An Assistant Entity that is a tool invoked to
  perform a structured Task. Interaction with an AT is one-shot: the
  invoking entity submits a Task and receives a response. A Task
  handled by an AT may be completed through a single Event. An AT
  cannot invoke other entities.

- Invocation: an entity's establishment of a new Dialog to pursue an
objective. AIE--FNE invocation starts an Activity; downstream
invocations create Dialogs within it. Repeated invocations create
distinct Dialogs, whereas additional Tasks or Events within a Dialog do
not constitute new invocations.

# Architectural Model

## Architecture Visualization

An Activity begins when an Activity Initiating Entity (AIE) invokes
one First Next Entity (FNE) to pursue its objective. The invocation
establishes the Activity and its initial Dialog between the AIE
and the FNE:

~~~
          Activity
    +-----------------------------+
    |                             |
    | +--+--+   Dialog   +--+--+  |
    | | AIE |<---------->| FNE |  |
    | +-----+            +-----+  |
    |                             |
    +-----------------------------+
~~~


If the goal of the Activity can be fulfilled by the FNE itself, no
additional entities are invoked for that Activity. Alternatively, if
assistance from additional entities is required, the FNE, or any
downstream assistant entity, may invoke assistant entities. Dialogs
are created between the invoking entity and each invoked assistant
entity, and the respective Tasks are handled within their associated dialogs, all under the
umbrella of the Activity, and the interaction wave propagates.
Dialogs created in this way belong to the Activity but might not
directly involve the AIE; each is related to the Dialog that
created it through lineage relationships (see "Dialog Lineage"). The
interaction wave may then look like the following:

~~~
          Activity
+------------------------------------------------------------------+
|                                                                  |
| +-----+   Dialog   +-----+   Dialog   +-----+   Dialog   +-----+ |
| | AIE |<---------->| FNE |<---------->| AA1 |<---------->| AA3 | |
| +-----+   task(s)  +-----+   task(s)  +-----+   task(s)  +-----+ |
|                       +                  +                       |
|                Dialog |           Dialog |                       |
|                       | task(s)          | task(s)               |
|                       v                  v                       |
|                    +-----+            +-----+                    | 
|                    | AT1 |            | AT2 |                    |
|                    +-----+            +-----+                    |
|                                                                  |
+------------------------------------------------------------------+
~~~

An Activity groups multiple independently maintained Dialogs, each of
which may contain Tasks:

Conceptually, the model looks like the following:

~~~
                   Activity
                      |
          +-----------+-----------+
          |                       |
        Dialog                  Dialog
          |                       |
        Tasks                   Tasks
       (EVENTS)                (EVENTS)
          |                       |                       
        +---------------------------+
        |     Resource Layer        |
        +---------------------------+                              
~~~

The figure above introduces the Resource Layer, the logical location
where Resources pertaining to an Activity reside. The Resource Layer
makes those Resources accessible to the Dialogs (and their tasks) of the Activity, and
the same Resource may be used by more than one Dialog. For example,
multimedia components may be consumed, produced, or transformed by
multiple Dialogs within the same Activity. The Resource Layer is a
logical construct: it does not prescribe where Resources are
physically stored and does not imply a globally visible or centrally
managed store.

Cross-Dialog lineage and shared Resource use are the architectural
focus. The interaction graph need not be globally stored or visible;
participants may retain only the correlations their operations require.

## Motivating Multimedia Example

Under the above model, various use cases can be envisioned. As a simple motivating example, consider a user who wants a video V1 analyzed. The user, acting as
the Activity Initiating Entity (AIE), starts an Activity with the
objective "analyze video V1" and invokes Agent A. Agent A is the
First Next Entity (FNE) of the Activity: the Dialog between the user
and Agent A (D1) is established which extends the activity; a task under this dialog is created and used to assign the objective of the activity to Agent A over D1, and V1 becomes available to the Activity and D1 as an initial Resource.


Agent A decomposes the analysis task: The visual content requires a
video-analysis specialist, and the audio content must be transcribed
into text before it can be analyzed. Agent A therefore invokes
Agent B, an Assistant Agent (AA) that can analyze video and text but
not audio. A Dialog (D2) is created between Agent A and Agent B;
Agent A transfers V1 to Agent B within D2 and assigns the
video-analysis Task. Agent A also invokes Tool X, an Assistant Tool
(AT), to transcribe the audio track of V1. A Dialog (D3) is created
between Agent A and Tool X. Interaction with Tool X is one-shot in
nature: Agent A submits the transcription Task and receives
transcript T1 in response. The Task in D3 references V1 by its Resource
identity; Tool X accesses V1 through the Resource Layer of the Activity. Agent A then provides
T1 to Agent B within D2, where it is analyzed as text alongside the
video.

Mapped to the participant roles:

~~~
User     AIE
Agent A  FNE
Agent B  AA  (invoked by Agent A)
Tool X   AT  (invoked by Agent A)
~~~

Note that roles follow the invocation chain rather than the nature
of the entity: had the user invoked Tool X directly, for example,
to pre-transcribe V1 before engaging Agent A, Tool X would have
been an FNE rather than an AT.

The scenario is depicted below:

~~~
                     User (AIE)
                          |
                          | Dialog D1:
                          | (Activity, V1)
                          v
                   Agent A (FNE)
              +-----------+-----------+
              |                       |
   Dialog D2: |                       | Dialog D3:
  (V1, Tasks) |                       | (Task, V1 ref.)
              v                       v
        Agent B (AA)            Tool X (AT)
       video analysis       audio transcription
              ^                       |
              |                       v
              |                      T1
              |                       | (back to agent A)
              |                       |
              |                       |
              +------ Dialog D2 ----Agent A (FNE)
              (T1 provided to Agent B)
~~~


V1 and T1 pertain to the same Activity and reside in its Resource
Layer. The example exhibits both cross-Dialog resource patterns of
the Architectural Model: the same Resource, V1, is used by two
Dialogs (D2 and D3), and T1 is produced in one Dialog (D3) and
consumed in another (D2). It also illustrates the two ways a
Resource can reach a participant. A Resource may be transferred to
the receiving participant within a Dialog, as V1 is transferred to
Agent B in D2, or it may be referenced by its stable Resource
identity and accessed through the Resource Layer, as V1 is accessed
by Tool X. The architecture does not mandate either mechanism. In
both cases, the Resource retains a single logical identity, and,
where needed, its provenance, so that every participant refers to
the same Resource regardless of how it physically obtained it.

For Agent B to interpret T1 correctly, it needs to know that T1 is
the transcript of the audio track of the same video V1 it is
analyzing, and that both Resources belong to the same Activity. Essentially, we need the following relationship to be established and understood by Agent B:

~~~
T1 derived-from V1
~~~

A conversational identifier alone is insufficient to express these
semantics. A protocol-local context identifier, such as an A2A
`contextId`, groups the messages and Tasks of a single Dialog, but
it cannot express that T1 was derived from V1, that T1 was produced
in D3 and is meaningful to the Tasks of D2, or that D1, D2, and D3
all belong to the same Activity. Tool X's local job identifier is of
no help either: it is meaningful only within D3. Moreover, Agent B
may be concurrently analyzing similar videos for other Activities;
without Activity correlation and Resource provenance, Agent B cannot
distinguish this T1 from a transcript belonging to an unrelated
Activity. What is required is a correlation model that carries
Resource provenance and Activity correlation across Dialog
boundaries. This architecture captures this correlation model as standardized typed Relationships, available across dialog boundaries.




## Cross-Organization Operation

The example of the previous section becomes architecturally more
interesting when its participants are operated by different
organizations:

~~~
User (AIE)      invokes Agent A (Organization 1)
Agent A (FNE)   Organization 1
Agent B (AA)    Organization 2
Tool X (AT)     Organization 3
~~~

Dialog D2, between Agent A and Agent B, and Dialog D3, between
Agent A and Tool X, cross organizational boundaries, while Dialog
D1 involves only the user and Organization 1.

Each organization maintains its own local identifiers for the
portions of the Activity it can see as shown below:

~~~
Organization 1:
  activity_id = O1-17

Organization 2:
  A2A context = AB-92
  A2A task = TB-44

Organization 3:
  tool job = JOB-991
~~~

In architectural terms, O1-17 is Organization 1's local handle for
the Activity, AB-92 is the Protocol-Local Context of Dialog D2,
TB-44 is a protocol-local Task identifier within D2, and JOB-991 is
Tool X's local identifier for the transcription Task in D3.

These local identifiers need not be globally shared, unique, or even
syntactically alike. Correlation must therefore work without replacing
or exposing protocol-local state. The architecture separates:

~~~
logical correlation
~~~

from:

~~~
local identifier representation
~~~

A domain may maintain private mappings between the two. 

The Resource Layer is logical, not administrative: V1 may be stored by
Organization 1 and accessed by Tool X subject to that organization's
access and privacy policies.

# Dialogs Within an Activity

As described in the Architectural Model, a single Activity can
involve multiple Dialogs.

An Activity does not imply a common conversational context, participant
set, protocol instance, or lifecycle for its Dialogs. In other words:

> An Activity groups multiple independently maintained Dialogs
> for correlation purposes.

Dialogs in different branches may have explicit relationships without
direct communication or shared conversation. Common Activity membership
alone does not imply a relationship.

An AIE may initiate multiple Activities to pursue a broader objective.
The AIE may preserve associations between those Activities locally and
coordinate their results or Resources across Activity boundaries.
Consequently, Dialogs belonging to different Activities may be
correlated indirectly through the AIE, without requiring a direct
Dialog-to-Dialog relationship or disclosure of the other Activity to
its participants. Having the same AIE does not, by itself, imply that
those Activities are related.

This distinction supports delegation, specialization, cross-domain work,
and recovery without collapsing independent interactions into one
Dialog.

The motivating example illustrates this property: Dialogs D1, D2,
and D3 are established by separate invocations, are maintained
independently, and may use different application protocols, yet all
belong to the same Activity.



## Reasons for Creating a New Dialog

New Dialogs routinely arise as an interaction wave expands through
delegation, specialization, tools, or recovery. For example:

- Delegation: An agent may delegate part of its work to another
  agent.

- Specialization: A participant may invoke a specialist because the
  current participants lack a required capability. For example, if
  Agent B cannot understand language L, it invokes an agent capable
  of language L and creates a new Dialog with it.

- Tool invocation: Invoking a tool creates a separate Dialog for the
  one-shot interaction.

- Administrative Separation: A new Dialog may be created when
  execution crosses into another organization, vendor, security
  domain, or policy boundary.

- Failure Recovery: A replacement Dialog may be created after the
  failure of a participant, route to a participant, the transport, or an intermediary entity. For
  example, if Agent B fails mid-Activity, a replacement Agent B' is
  started and opens a new Dialog with Agent A to continue from
  where they stopped.

- Privacy Separation: A participant may intentionally create a
  separate Dialog so that only a subset of Activity information is
  visible to the downstream participant. The same two participants
  may even run multiple Dialogs in parallel under the same Activity,
  each exposing a different subset of Activity information.

The motivating example already exhibits several of these reasons:
D2 arises from delegating video analysis to a specialist, D3 from
tool invocation, and both Dialogs cross administrative boundaries.

Applications decide when to establish Dialogs. The architectural concern
is correlating each Dialog with its Activity and relevant lineage, where
needed to capture causality.

## Dialog Independence

Each Dialog is independently maintained.

In the motivating example, D2 and D3 both belong to the same
Activity:

~~~
Activity: analyze video V1

D2:
  Agent A <-> Agent B

D3:
  Agent A <-> Tool X
~~~

D2 and D3 may differ in:

- Participants: Agent B versus Tool X;
- Protocol (e.g., HTTP vs MoQ) and transport (e.g., TCP  vs UDP);
- Protocol-local identifiers: A2A context AB-92 and Task TB-44 in
  D2, tool job JOB-991 in D3;
- Resources referenced: V1 and T1 in D2, but V1 by reference in D3;
- Message history;
- Lifecycle state: D2 is long-running, D3 is one-shot;
- Privacy policy and authorization level;
- Administrative domain: Organization 2 versus Organization 3.

Therefore:

~~~
D2 != D3
~~~

even though both belong to the same Activity.

Dialog independence extends to the same pair of participants. Two
entities may maintain multiple concurrent Dialogs with each other:
one per Activity on which they both work, or several within a
single Activity when privacy or task separation calls for it.
Suppose that, alongside D2, Agent A opens a clarification
dialog with Agent B:

~~~
Activity: analyze video V1

D2:
  Agent A <-> Agent B      (video-analysis Tasks)

D4:
  Agent A <-> Agent B      (clarification dialog)
~~~

Although D2 and D4 share participants, they have distinct local states
and potentially different disclosures:

~~~
D2 != D4
~~~

A Dialog is identified by its own identity and the Activity it
belongs to, never by its participant set.

A participant MUST NOT assume that another Dialog participating in
the same Activity shares the same local state, and MUST NOT assume
that its Dialog with a given counterpart is the only one.

## Multi-Party Dialogs

A Dialog may involve more than two participants sharing one
conversational context and lifecycle. For example, Agent A, Agent B, and
Agent C may participate together:

~~~
    Single three-party dialog 
  +-------------------------------+
  |            Agent A            |
  |               |               |
  |           +---+----+          |
  |          /          \         |
  |     Agent B        Agent C    | 
  +-------------------------------+
~~~

This differs from Agent B invoking Agent C to create another Dialog:

~~~
Agent A <------> Agent B <------> Agent C
           D2               D5
~~~

D2 and D5 remain distinct when Agent A and Agent C do not share
the same conversational context, even though D5 was created as a
result of D2 with a relationship of for example:

~~~
D5 delegated-from D2.
~~~

The distinction rests on shared Dialog context, not on the number
of interaction links between participants.

A multi-party Dialog's participant set may change over time:

~~~
Initially:       {Agent A, Agent B}
Agent C joins:   {Agent A, Agent B, Agent C}
Agent B leaves:  {Agent A, Agent C}
~~~

This architecture does not define participant-membership management
semantics. A dialog-management protocol, however, may need to
consider participant addition, removal, revocation, and membership
changes as part of Dialog lifecycle management.

Finally, any multi-party Dialog can be decomposed into multiple
pairwise Dialogs with appropriate relationship mappings. Whether to
use one multi-party Dialog or several pairwise Dialogs is a design
choice driven by privacy, cross-organization operation, and
efficiency. A protocol for dialog management and context
propagation SHOULD enable both modes of operation.

## Dialog Lineage

Dialogs can be related through explicit lineage. For example:

~~~
D5 delegated-from D2
~~~

indicates that D5 exists because work from D2 was delegated into a
new Dialog. A new Dialog may carry forward work or correlation context from a
previous Dialog:

~~~
D2' continuation-of D2
~~~

Here D2 and D2' have distinct logical identities. This relationship
does not represent resuming D2 with its original identity. A new
Dialog may instead supersede an earlier Dialog:

~~~
D2' replaces D2
~~~

The two relationships describe different associations and may coexist.
Their semantics are further discussed under "Recovery and Continuation".

This architecture distinguishes:

~~~
dialog identity
~~~

from:

~~~
dialog relationship
~~~

Two Dialogs remain independently identified even when one was
created from another.

## Hidden Delegation and Partial Visibility

A downstream Dialog need not be disclosed upstream. Suppose Agent B
invokes Agent C:

~~~
Agent A <------> Agent B <------> Agent C
           D2               D5
             <------------->
                Delegation
~~~

Focusing on this delegation, the resulting visibility may be:

~~~
Agent A knows:
  Agent B
  D2

Agent B knows:
  Agent A
  Agent C
  D2
  D5
  D5 delegated-from D2

Agent C knows:
  Agent B
  D5
~~~

In the above example, Agent A does not know that Agent C exists. Agent C does not know that Agent A exists. In other words, the visibility level is a decision that participants make.

Such hidden delegation is common across organizational, vendor, trust,
and privacy boundaries. The architecture separates:

~~~
relationship exists
~~~

from:

~~~
relationship is disclosed
~~~

A relationship can be retained locally by an intermediary without
being propagated in full. For example, agents inside an
organization need to know only that edge agent Y can help achieve
objective X. Agent Y connects to agent Z in another organization;
agent Z needs to know only that agent Y exists and is asking for V, 
not who asked Y. Yet agent Y must know that responses from agent Z
relate to requests from internal agents.

## Dialog Topologies

An Activity's Dialogs may branch, chain, and converge.

A participant may independently invoke another entity, so Activity
topology MAY emerge from decentralized decisions.

The following scenarios  are few possible examples of how topologies of an ACTIVITY MAY extend. 

### Branching

A participant in a Dialog can create several downstream Dialogs with other new participants as need arises. Agent B may invoke both Agent C and Agent E to pursue independent parts of its
work:

~~~
            
         Activity A1
              |
           D1 |
              v     D2, D4
           Agent A <------> Agent B
              |               /\
           D3 |           D5 /  \ D6
              v             /    \
            Tool Y         v       v
                        Agent C   Agent E
~~~

with the relationships:

~~~
D5 delegated-from D2
D6 delegated-from D2
~~~

D5 and D6 are independent: they may execute concurrently, use
different protocols, process different Resources, terminate
independently, and expose different subsets of upstream context.
Agent C and Agent E do not need to know of each other's existence.

### Chaining

A downstream participant can itself create further Dialogs. If
Agent C invokes Agent E, the Activity gains another level:

~~~
            
         Activity A1
              |
           D1 |
              v     D2, D4
           Agent A <------> Agent B
              |               /\
           D3 |           D5 /  \ D6
              v             /    \
            Tool Y         v      v
                        Agent C  Agent E
                           ^        ^
                           |   D7   |
                           +--------+
~~~


yielding:

~~~
D5 delegated-from D2
D7 delegated-from D5
~~~

The Activity therefore forms a delegation tree or, more generally,
a graph. No participant is required to maintain the complete
ancestry: Agent C may know that D5 has an opaque upstream origin
without knowing the identity of Agent A or of D2.

### Converging Dialogs

Separate branches can later contribute to another Dialog. If Agent
B invokes an aggregator, Agent F, to combine the results produced
independently in D5 and D6:

~~~
                D2
               /  \
              v    v
             D5    D6
              \    /
               \  /
                v
                D8
~~~

then D8 is correlated with more than one upstream Dialog:

~~~
D8 aggregated-from D5
D8 aggregated-from D6
~~~

`aggregated-from` indicates that D8 was created to aggregate or
consume results produced independently in multiple upstream
Dialogs. It is a multi-parent Dialog relationship, distinct from
the single-parent `delegated-from`. Where only correlation is
asserted and no aggregation causality is intended, the weaker
`related-to` relationship may be used instead.

### The Dialog Graph

The set of Dialogs participating in an Activity can therefore be
modeled as a directed graph:

~~~
G_D = (D, R_D)
~~~

where D is the set of Dialogs and R_D is the set of typed
relationships among them. Examples MAY include:

~~~
delegated-from
aggregated-from
continuation-of
related-to
replaces
~~~

The graph may branch, converge, and evolve over time, and no
participant is required to maintain it in full.

## Independent Dialog Lifecycle

Related Dialogs may hold different lifecycle states simultaneously (the
states below are illustrative; the dialog-management
protocol defines the exhaustive list of lifecycle states of a Dialog.)

~~~
D2 ACTIVE
D3 COMPLETED
D5 SUSPENDED
~~~


Termination of D5 does not imply termination of D2, and failure of
D5 does not necessarily invalidate D2 or D3.

This distinction matters for failure recovery, long-running
activities, asynchronous delegation, partial completion, and retry
behavior.

Related Dialogs can be correlated (they belong to the same Activity
and may be linked through lineage) yet maintain independent lifecycle
states. A transition in one Dialog does not inherently impose the same
transition on another Dialog or its Tasks. However, changes in
requirements, dependencies, or security and authorization conditions
may have consequences for related work. Where needed, sufficient
Correlation Metadata is preserved and selectively propagated to
identify affected Dialogs and Tasks and associate any resulting
changes with their causes.

Lifecycle coupling, for example:

~~~
if D2 terminates, D5 should terminate
~~~

is not implied by this architecture. The actual consequences for
related work depend on the applicable protocols, policies, and
application requirements. This architecture concerns their correlation,
not the operational transitions themselves. A suspended Dialog may be
resumed, or a sub-dialog may be created as a child of it to resume
partial operations, independently of other Dialogs.


## Recovery and Continuation

A failure of a participant, intermediary, or transport path may
interrupt communication without necessarily ending a logical Dialog.
Recovery requires distinguishing the continuity of the Dialog's
identity from continuity of its work and correlation context.

When the existing Dialog D2 is resumed, its logical identity remains
D2, even if its transport connection, protocol session, or
protocol-local identifiers change:

~~~
Before interruption:  D2
After resumption:     D2     (same logical Dialog)
~~~

No `continuation-of` relationship between distinct Dialogs is needed
in this case, because no new logical Dialog has been established.
The correlations involving D2 must remain available or be recoverable
where they are needed to maintain coherent context.

Alternatively, recovery may involve establishing a new Dialog D2'
with a distinct logical identity. Three situations are possible:

- Continuation without replacement: D2' carries forward work or
  correlation context associated with D2, but does not necessarily
  supersede D2.

  ~~~
  D2' continuation-of D2
  ~~~

- Replacement without continuation: D2' supersedes D2 as a Dialog,
  for example when the original objective is attempted again without
  carrying forward D2's previous execution context.

  ~~~
  D2' replaces D2
  ~~~

- Continuation and replacement: D2' both carries forward work or
  context associated with D2 and supersedes D2 as a Dialog.

  ~~~
  D2' continuation-of D2
  D2' replaces D2
  ~~~

`continuation-of` expresses continuity of work or correlation context
between distinct Dialogs. `replaces` expresses Dialog-level
supersession, rather than the transfer of an arbitrary subset of
Tasks. The two relationships are not mutually exclusive. Neither
imposes automatic Task lifecycle transitions, termination of the
original Dialog, or wholesale inheritance of its correlations. Where
only some work is carried forward, `continuation-of` together with
relevant Task or Resource relationships may express that narrower
continuity without asserting Dialog-level supersession.

Historical associations remain meaningful after recovery. For
example, if D3 was delegated from D2, that fact does not disappear
when D2' replaces D2. If D2' also needs to correlate subsequent work
with D3, the necessary associations must be preserved or established;
they cannot be inferred solely from `D2' replaces D2`.

Communication recovery and correlation recovery are distinct. A
transport may reconnect while a participant has lost the mappings
needed to correlate D2 with related Dialogs, Tasks, or Resources.
Conversely, those mappings may remain available while a participant
or its transport path is unreachable. Resuming the same Dialog identity
also does not, by itself, guarantee recovery of its application Task
state or of the outcome of an interrupted operation. An uncertain
operation outcome is not itself a correlation failure if the operation
remains correctly associated with its relevant Dialog and Task.

Within an established protocol-managed correlation scope, a
participant remains responsible for the correlations it undertook to
maintain through an interruption or recovery. Where necessary
correlations cannot be preserved or recovered, that limitation must
be distinguishable from successful correlation maintenance, subject
to the detection and reporting capabilities of the relevant protocol.
This does not imply visibility into unregistered downstream Dialogs or
perfect attribution of a recovery failure.

Recovery may involve several distinct layers:

- Correlation context: preservation or re-establishment of Dialog
  identities, relationships, and relevant Task and Resource
  associations, as represented by the context-management protocol.
- Application-protocol state: the sessions, Tasks, and operation
  outcomes maintained by protocols such as A2A and MCP, which may
  continue independently of a transport connection.
- Transport connectivity: re-establishment or migration of a
  connection or path through the underlying transport mechanisms.

The architecture identifies these distinct concerns and their
correlation implications, without specifying recovery procedures.
Open protocol-design questions include how lost mappings are recovered
or rebuilt, how peers distinguish resumption from new Dialog creation,
how correlation is handled when an intermediary disappears, and how
unreachable participants or uncertain operation outcomes are reported.
These questions are to be addressed in the dialog-management protocol
draft, in coordination with relevant IETF working groups (for example,
SIP Core for Dialog semantics, QUIC for transport migration, and TLS,
WIMSE, and OAuth for session, identity, and delegated authority) and
with the application protocols to which the protocol binds, such as
A2A and MCP.

## Interruptions and Barge-in

Work in progress can be interrupted. A barge-in is new input introduced while work is underway that may
modify, redirect, or interrupt it. It does not automatically determine
whether Tasks are modified, created, or replaced, or whether another
Dialog is established.

The running Activity of "analyze the video" shows the pattern. The user asked Agent A to
analyze video V1; Agent A delegated the video-analysis Tasks to
Agent B and the transcription Task to Tool X, and work is in
progress. Ten minutes in, the user barges in and tells Agent A to
analyze only the last 50% of the video. In this example, no new
entity is invoked: the same Activity and Dialogs remain, while the
requirement governing the work changes.

Two aspects of this barge-in are relevant to correlation:

- First, the new requirement must be associated with the work it
affects. In this example, Agent B's Task is modified to analyze only
the last 50% of the video. Other barge-ins could instead result in
new or replacement Tasks, suspension, cancellation, or no change to
particular Tasks. Whether Tool X's transcription Task is also affected
is an application decision: the existing transcript may still cover
the full video, and Agent B may simply use its relevant part, or
Tool X's Task may be narrowed to match.

- Second, the input that carries the barge-in may arrive in a
different modality from the original requirement. The original
requirement reached Agent A as text; the barge-in reached it as
voice. In this example, the spoken correction is a Resource used as
input to the affected Task:

~~~
AX1 input-to TB1
~~~

The correction is not a standalone requirement: "analyze only the
last 50%" is meaningful only relative to the original requirement it
narrows. Recording that the spoken correction modifies the original
requirement, rather than leaving the two statements unconnected, is
what lets any participant determine which requirement governs the
work after the interruption. How that association is encoded is left
to the dialog-management protocol.

A barge-in may have several correlation-relevant aspects, which
are not mutually exclusive:

- a lifecycle change in an affected Dialog, for example from active
  to interrupted;
- the establishment of a related Dialog, such as a clarification
  side-channel; and
- Resource provenance or binding information when the new input uses
  a different modality or changes the requirement associated with a Task.

Which aspects arise is an application decision. The architectural
concern is that the initiating input, affected Dialogs, Tasks, and
Resources, and any resulting changes can be correctly associated;
it does not prescribe the operational response.

A barge-in can also affect in-flight or subsequent Events within an
affected Task. Events remain local to their Tasks and do not acquire
independent lifecycle states. Where necessary, their association with
the affected Task and relevant requirements must remain interpretable.

A barge-in may affect several related Dialogs. Suppose Agent B has
created D5 with Agent C, Agent C has created D7, and D8 aggregates D5
and D6. In this example:

- D5 is interrupted: its in-flight Task is modified, because the
  requirement it was pursuing has changed.
- D7, which Agent C chained from D5, is suspended: the work it was
  doing served the requirement that has just changed.
- D8, which aggregates results produced independently in D5 and D6,
  is terminated: the Task that required D8 to aggregate from
  upstream Dialogs was replaced by the barge-in, so D8's reason for
  existence is gone and its awaiting Tasks are cancelled.
- D6, which runs independently of D5, is untouched.

One interruption readjusted work in four Dialogs: a lifecycle
transition and Task modification in D5, a suspension in D7, and a
termination in D8. Each is an explicit
fact of the model, tied back to the interruption through the lineage
relationships that link the four Dialogs.

These questions are candidates for coordination with transport-
oriented work (for example, WebTransport, WebRTC, and MoQ) and with
the dialog-management protocol. Open questions include:

* how lifecycle transitions behave when interruptions occur
  mid-course;
* how changes resulting from a barge-in are associated with affected
  Dialogs, Tasks, and Resources;
* how affected ongoing or subsequent Events remain interpretable
  within the relevant Task;
* how the association between the barge-in input and the requirement
  it affects is preserved across modality changes; and
* how far correlation information about the effects of a barge-in
  needs to travel through the Activity graph.

## Reuse Across Dialogs

The same Resource can appear in several Dialogs. In the running
example, V1 is used by both D2 and D3, and T1, produced in D3, is
consumed in D2 and later referenced in D5 as provenance:

~~~
Resource V1                 Resource T1
  |                           |
  +--> D2                     +--> D2
  |                           |
  +--> D3                     +--> D5
~~~

Sharing a Resource does not imply identical local interpretation:


~~~
D2:
  T1 analyzed as primary text input

D5:
  T1 retained only as provenance
~~~

Resource identity remains stable while interpretation is local.
These Dialog-local interpretations are Resource Bindings, and roles can change as the Activity
executes and delegations happen.

Sharing also does not imply the same Dialog. For example, that D2 and D5 both
reference V1 does not imply:

~~~
D2 == D5
~~~

or that Agent A, Agent B, and Agent C share one conversation.
Resource correlation and Dialog correlation are separate concerns:
a reference to a Resource carries a meaning understandable only
within its respective Dialog.

Likewise, a shared Activity does not imply a shared participant
set:

~~~
D2 belongs-to the Activity
D5 belongs-to the Activity
~~~

does not imply that all participants in D2 and D5 know or can
address one another. The Activity provides a logical grouping,
not a communication group.

## The Running Example, Assembled

The concepts introduced in this section can be assembled into a
single picture of the running example. The figure below unifies the
two views of the Architectural Model: the invocation view of the
interaction wave, instantiated with the example's participants and
roles, framed by the Activity root above and the Resource Layer
below, as in the containment view.

~~~
                   Activity: analyze video V1
                                |
                           User (AIE)
                                |
                                | Dialog D1 (Activity, V1)
                                v
                          Agent A (FNE)
                       + -----------------+
         Dialog D2     |                  |   Dialog D3
      delegated-from D1|                  | delegated-from D1
           task(s)     v                  v  task (one-shot)
                 Agent B (AA)        Tool X (AT)
                       +
      Dialog D5        |
      delegated-from D2|
      task(s)          |
                        v
                  Agent C (AA)
               (invisible to Agent A)

      +----------------------------------------------+
      |               Resource Layer                 |
      |  V1:  transferred to D2; referenced by D3    |
      |  T1:  produced in D3; consumed in D2;        |
      |       referenced as provenance in D5         |
      |  AX1: barge-in input (voice); input-to TB1   |
      +----------------------------------------------+
~~~

The figure exhibits, in one place, most of the properties
established in this section:

- Branching: D2 and D3 are both delegated from D1.
- Chaining: D5 is delegated from D2, extending the wave below
  Agent B.
- Hidden delegation: D5 and Agent C are invisible to Agent A.
- Independent lifecycle: the Dialogs need not share state; D3
  completes with its one-shot Task while D2 remains active and D5
  may be suspended.
- Resource reuse: V1 is shared by D2 and D3, and T1, produced in
  D3, is consumed in D2 and referenced as provenance in D5.
- Interruption: a barge-in (AX1, voice) narrows the objective of
  TB1 mid-course; its effects travel through lineage to the Dialogs
  that depend on it.

Every edge is an invocation, every Dialog is independently
maintained, and every correlation is an explicit relationship.
This is the structure that context propagation, the subject of the
following section, must preserve.


# Tasks Within a Dialog

Tasks are bounded units of work within Dialogs. An invocation
establishes a Dialog but does not necessarily create a Task. A Dialog
may contain zero or more Tasks, performed sequentially, in parallel, or
through one-shot Assistant Tool exchanges.

The previous section zoomed in from Activities to Dialogs. This
section zooms in one more level, to the Tasks inside each Dialog.
It examines Task identity, states, cross-Dialog relationships,
and the effects of failures and privacy boundaries.

The model defines the relevant semantics; protocol documents determine
their implementation.

## Task Identity: Local and Global

Tasks use identifiers defined by the local application protocol (such as
an A2A `taskId`). Such identifiers are not assumed to be meaningful or
resolvable outside their Dialog.

Unlike a Resource that needs stable identity across boundaries, a Task
need not have a universal identifier. Its cross-Dialog association may
be asserted by a participant that can resolve the relevant local Tasks. Whether to standardize a global Task identifier remains a
protocol-design decision.

## Task States

Tasks have lifecycle states independent of their Dialog's state. The
following are illustrative, not normative; the dialog-management
protocol defines the applicable states.

Progression states - a Task holds exactly one at a time:

~~~
defined       the Task exists; work has not begun
queued        the Task was accepted and is waiting behind
              other work
in-progress   work is in progress
awaiting      the Task cannot make progress on its own; it is
              blocked on something external, for example another
              Task's result, a decision, or an authorization
delegated     part of the work was passed to another Dialog;
              the Task awaits its result
awaiting-approval   the Task is blocked on a human decision
auth-required       the Task is blocked on additional
                    authorization
interrupted   work was paused or cut off; may resume
completed     the bounded work is done
failed        work ended without completing
timed-out     the Task exceeded its maximum duration
rejected      the work was declined before it began, for
              example out of scope or against policy
cancelled     the Task was ended administratively, for example
              by a revoked delegation
unknown       the state of the Task cannot be determined
~~~

Modifiers - facts about the Task that may persist alongside a
progression state:

~~~
modified      the objective or parameters of the Task changed
superseded    another Task replaces this one
~~~

Some states, such as completed, timed-out, rejected, or cancelled, may
be final. Retry behavior is an application decision.

A one-shot Task with an Assistant Tool can pass through the whole
set in one exchange: defined on assignment, in-progress on
submission, completed on response.

Structural relations such as delegation or replacement are generally
stable; lifecycle assertions such as completion and modification are
mutable and can become stale. Participant acting on an old copy can make
incorrect decisions, as discussed later under Stale Context.

A Task's actual state is therefore different from what a participant
knows of it:

- A participant knows a Task's state only as of the time the source
  of that state observed it (see "Context Freshness and Lifetime").
- When a participant cannot confirm a state that is not final, for
  example because the downstream participant that reported it has
  become unreachable, it treats the state as unknown instead of
  passing on the last state it received as current.
- The unknown state describes the participant's knowledge, not the
  Task's progress. Unlike completed, failed, or cancelled, it is not
  an outcome, and the Task itself may still be in progress.

A Task has one progression state at a time; modifiers may coexist with
it. Thus a Task may be in-progress and modified without changing its
progression state.

Lifecycle updates are also Correlation Metadata. "t_32 completed",
"t_21 modified", "t_15 superseded" are precisely the assertions
that may need to propagate between involved parties, in the
upstream direction alongside results and failure information, as
discussed later under Directional Propagation.

## Task Independence

Each Task has its own lifecycle. A change to one does not automatically
change another, even within the same Dialog.

Within a single Dialog, several Tasks can be in different states at
the same time:

~~~
t_12 in-progress
t_22 awaiting
t_32 completed
~~~

Completing, failing, or cancelling one Task does not automatically
change the other Tasks within its Dialog.

Task independence also applies across Dialogs. Relationships such as
`delegated-from` or `replaces` record association; it does not couple their lifecycles, and it does
not cause a change in one Task to propagate to another. Dependent work changes only when its participants
decide or applicable requirements demand it.

This permits partial completion, parallel work, and selective failure
within one Activity.

As with Dialog lifecycle, a Task's lifecycle state is mutable and may
become stale; a participant acting on an old copy can make incorrect
decisions, as discussed later under Stale Context.

## Task Relationships Across Dialogs

Examples of Task-level relationship types include:


~~~
delegated-from    t_12 delegated-from t_21
related-to        a weaker assertion of correlation, without
                  delegation causality
replaces          a Task supersedes another, typically a failed
                  or abandoned one
~~~

`delegated-from` is the primary cross-Dialog relationship: it
asserts that one Task exists because of a requirement found in
another. A Dialog may contain many Tasks while only one of them
causes a downstream delegation; `delegated-from` records which one
(see "Task Lineage").

At Task granularity, `replaces` expresses that one Task supersedes
another. Replacing a Dialog does not automatically replace all of
its Tasks. When a new Task actually supersedes an original Task, its
relationship can be expressed explicitly:

~~~
t_12' replaces t_12
~~~

Dependency on an existing result may be inferred from Resource
provenance; awaiting a future result is Task state rather than a new
relationship type.

This avoids introducing a separate verb for every application
dependency.

## Task Topologies

Dialog and Task lineage are related but distinct. Creating a Dialog does
not require creating a Task or Task-to-Task association. Where
delegation originates in a Task, its Task relationship can be
represented explicitly:

~~~
D5 delegated-from D2       (Dialog level)
t_15 delegated-from t_22   (Task level, when applicable)
~~~

The Dialog and Task graphs are related but need not mirror each other:

- A Dialog may contain zero or more Tasks, so Task relationships
  add finer-grained detail where applicable.
- Within one Dialog, Tasks relate to one another through their
  Resources: the output of one Task is the input of another,
  sequentially or in parallel.
- One-shot interactions with Assistant Tools form leaves of the
  task graph: submit, receive, done.

The set of Tasks participating in an Activity can therefore be
modeled as a directed graph:

~~~
G_T = (T, R_T)
~~~

where T contains the Tasks and R_T their typed relationships. This graph
may correspond to, but need not mirror, the Dialog graph G_D. No
participant MUST maintain it in full.

## Impact of Dialog Failures on Task Relationships

Dialog failures or lifecycle changes may affect Tasks without
determining their states. Where consequential changes occur, they need
to remain correlated with the affected Dialogs and upstream work.

- Dialog terminated: Tasks may complete, fail, be cancelled, or continue
under application-specific arrangements. Correlation captures relevant
consequences without imposing a uniform transition.

- Dialog replaced: When D2' replaces D2, that Dialog-level assertion
does not determine what happens to each Task in D2. A Task may be
continued through a new Task, superseded, retained in its existing
state, or handled otherwise according to the application. Where a
new Task t_12' actually supersedes t_12, that association can be
recorded explicitly:

~~~
t_12' replaces t_12
~~~

A Task that is not re-established in D2' must not automatically be
interpreted as lost, cancelled, or failed solely because its Dialog
was replaced. Its known outcome or unresolved status, where relevant,
remains part of the correlation context.

- Dialog suspended: Depending on the work and its dependencies, associated
Tasks may be interrupted, left awaiting, or continue through other
arrangements. The relevant Task states and any resulting changes may
need to be correlated with the Dialog's suspension.

- Broken lineage: Loss of an upstream Dialog relationship can make related
cross-Dialog Task assertions unverifiable and prevent result correlation
or upstream resolution.

## Privacy Considerations

Task objectives and states can reveal intent, workflow, timing, and
delegation. Sharing results need not require disclosing the Tasks that
produced them.

Administrative domains may therefore:

- disclose Dialog participation without Task-level detail;
- share a Task's output while withholding its objective;
- omit cross-Dialog task relationships entirely;
- generalize Task objectives before propagation.

Task associations may be kept privately by intermediaries. Even Task
states may be withheld or generalized according to the applicable disclosure policy.

# Binding to Protocol-Local Context

Application protocols define their own context, Task machinery, and
identifiers, and this architecture does not replace them.

The architecture associates Dialogs and Tasks with applicable
protocol-local constructs, without redefining their identifiers:

~~~
Activity
   |
   +-- Dialog         <-> protocol-local context identifier
   |      |
   |      +-- Task    <-> protocol-local task identifier
   |
   +-- Dialog
          |
          +-- Task
~~~

Local protocols retain their identifiers and Task state. The model adds
cross-Dialog relationships, such as `t_14 delegated-from t_22`, through
mappings when needed.

A protocol-local identifier is meaningful only within the Dialog in
which it was established, and identifiers from different Dialogs or
protocols are not comparable. They are therefore not sufficient on
their own to relate objects across Dialogs. An assertion such as
"t_14 delegated-from t_22" names a Task local to one Dialog and a
Task local to another; it can be made only by a participant that can
resolve both, and it can reach a further Dialog only when the
endpoints are made referenceable there, for example through an opaque
reference or a translated identifier. Where correlation must persist
across boundaries, the model carries stable logical identities rather
than local ones (see "Resource Identity" and "Activity Correlation").
This is the point at which protocol-local context ends and
correlation begins.

The boundary is between local identification and cross-Dialog
correlation, with shared semantics used where interoperability requires
them.


# Context Management and Correlation Metadata Propagation

The preceding sections defined the model's objects and relationships.
This section considers the Correlation Metadata needed to preserve and
propagate their associations across participants.

Here, "context" does not mean an AI agent's internal memory, context
window, or conversation history.

Agent-internal memory and its content remain outside this architecture's
scope.

Here, context means relationships. Context is the set of
correlations among Activities, Dialogs, Tasks, and Resources:
this video belongs to this Task, this transcript is derived-from
that video, this Dialog was delegated-from that one. What travels
between parties is Correlation Metadata: small, typed statements
about those relationships, such as a Task completed, a Dialog
replaced, or a delegation revoked. Context Management therefore means maintaining clear relationships among Activities, Dialogs, Tasks, and Resources. This is achieved by preserving the necessary Correlation Metadata locally and propagating it between participants when required. It does not mean copying
agent memory, shipping content, or transferring conversation
history.

In this Context Management and correlation metadata propagation model, the responsibility for
maintaining coherence is layered:

- The architecture captured in this document defines what the correlation metadata means and when it should travel;
- The dialog-management protocol encodes it and determines, per
  Dialog, what must be included to maintain correlation and ensure
  interoperability across organizations and local protocols;
- The local protocols it binds to, such as A2A and MCP, carry the
  application interactions that the Correlation Metadata
  accompanies; the metadata is associated with those interactions
  and does not replace them; and
- The transport protocols to which that protocol binds deliver the metadata.
  A transport is a carrier: it moves the metadata but does not
  define what is propagated.

This distinction frames the propagation problem.

Dialogs branch, converge, or replace one another; Tasks and Resources
introduce finer-grained associations. Because their contexts are
independently maintained, participants must preserve and selectively
propagate Correlation Metadata where necessary.

When extending a Dialog or delegating work, a participant SHOULD decide
what to preserve, disclose, translate, or bind locally:

- What correlations should be preserved?
- Which Resources should be propagated, and by value or by
  reference?
- Which identifiers should be translated?
- Which lineage should remain local?
- Which information should be withheld?
- Which new local bindings should be created?

Those choices depend on requirements, privacy and authorization
policies, and administrative boundaries. The architecture makes the
resulting correlations expressible; the protocol determines how they are
communicated.

What a participant discloses is its decision; whether the Activity can proceed on those terms is the consequence. Maintaining correlation among an Activity's Dialogs, Tasks, and Resources requires sufficient Correlation Metadata to establish the necessary associations. However, this does not require every participant to possess the complete correlation context of the Activity. Correlations may be preserved locally by participants that maintain the associations between upstream and downstream Dialogs, Tasks, and Resources.

A participant may withhold Correlation Metadata in accordance with its policies. However, when the information available to the participants responsible for maintaining a required correlation is insufficient, that correlation cannot be established. The affected Dialogs or Tasks may therefore be unable to proceed. The architecture requires that such failures be reported explicitly rather than result in indefinite stalls. Forcing disclosure is a matter of authorization and trust and is outside the scope of this document.

The following subsections identify the relevant scope, metadata, and
propagation limits.

## Correlation Scope, Extension, and Maintenance Responsibilities

The logical Activity and its protocol-managed correlation scope may
differ. Participants may create downstream Dialogs without enrolling
them in an existing correlation protocol. The architecture therefore
distinguishes:

~~~
Existence of a relationship
~~~

from

~~~
Relationship preservation by a participant
~~~

and from 

~~~
Relationship inclusion in a protocol-managed correlation scope
~~~

An interaction can exist without being included in the protocol's
managed scope.

### Logical and Managed Correlation Scope

The following are views of one Activity, not additional protocol
objects:

- Logical interaction scope: the Dialogs, Tasks, Resources, and
  relationships that actually contribute to the Activity, including
  those unknown to some or all of its participants.
- Maintained correlation scope: the associations that participants
  actually preserve and can relate, whether locally or through one or
  more context-management protocols.
- Protocol-managed correlation scope: the associations brought under
  a particular interoperable context-management protocol, with
  identified participants accepting the corresponding maintenance
  responsibilities. Such scope need not be globally enumerable or
  visible to any single participant.


For example, the AIE invokes the FNE through D1, the FNE invokes Agent A
through D2, and Agent A invokes Agent B through D3. An illustrative
context-management protocol (e.g., MACP) manages their correlations with
Act-1. Agent B then invokes Agent C through D5 using another
non-interoperable protocol (XYZ), or no context-management protocol:

~~~
               Logical interaction scope of Act-1
+-----------------------------------------------------------------+
|                                                                 |
| AIE ---D1--- FNE ---D2--- Agent A ---D3--- Agent B              |
|  |                                           |                  |
|  +--- ABC-managed correlation scope ---------+                  |
|                                              |                  |
|                                              +--- D5 (XYZ)--- C |
|                                                                 |
+-----------------------------------------------------------------+
~~~

D5 can be logically related to Act-1 although ABC has no knowledge of
D5 or its participants. D5's absence from ABC is not, by itself, a
correlation-maintenance failure. In particular, the logical interaction scope under Act-1 may extend beyond the correlation scope of an employed protocol (i.e., ABC or XYZ). In other words, the correlation scope of a protocol follows the relationships that have actually been established for management by that protocol. 

In summary, each management protocol has a correlation scope that may span the logical interaction scope of its associated activity. Leakage between correlation scopes is determined by the participants shared across those scopes.

### Selective Extension and Boundary Bridging (Scope Leakage)

Extending a protocol-managed correlation to another Dialog is a
participant decision. Agent B may still maintain a private bridge
between its Task in D3 and work delegated through D5:

~~~
           ABC-managed                        Outside ABC
       +------------------+              +-------------------+
       | D3: Task t_13    |              | D5: Task t_15     |
       | Agent A <-> B    |              | Agent B <-> C     |
       +---------+--------+              +---------+---------+
                 |                                 |
                 +-- B's local t_13 <-> t_15 map --+
                              (not disclosed)
~~~

B's private mapping can interpret D5 outcomes or relay relevant changes
without disclosing C or D5. This preserves local correlation but does
not place D5 within the first protocol's managed scope. The architecture
does not prescribe protocol adoption or bridging mechanisms.

### Maintenance Responsibilities Within the Established Scope

The principle is:

> Voluntary extension must be distinguished from responsible
> maintenance. 

Once participants establish protocol-managed correlations, they
undertake to maintain the associations they accepted. They must preserve
or communicate relevant changes to affected correlations, not broadcast
every update or disclose the complete upstream context. Intermediaries
may retain private mappings or translate identifiers.

Consider a barge-in in D1 changing requirements for Tasks in D2 and D3:

~~~
       D1                   D2                       D3
 AIE <----> FNE       FNE <----> agent A   Agent A <----> Agent B
     Task t_11            Task t_22                 Task t_13
        |                      |                       |
 barge-in changes              |                       |
 requirement for t_11          |                       |
        +-- relevant update -->+-- relevant update --->+
                               |                       |
                        correlate impact         correlate impact
                            on t_22                  on t_13

     An unrelated Dialog D5 does not require the same update.
~~~

Participants managing D1--D2--D3 have responsibilities for those
established associations. If Agent A fails to communicate a necessary
change between D2 and D3, that is a managed-correlation maintenance
failure, not merely a choice not to extend scope (i.e., a violation). The protocol should
make this distinction observable without forcing disclosure of private
metadata.

The protocol cannot impose maintenance duties on Agent C in an
unenrolled D5. Agent B remains responsible for its managed D3
associations, but failure within private D5 may be unobservable and its
root cause uncertain.

### Failure Visibility and Bounded Accountability

TBD

When a downstream participant becomes unreachable, the source time on
relayed lifecycle assertions (see "Context Freshness and Lifetime")
lets upstream participants see how long a reported state has gone
unconfirmed instead of treating it as current.

## The Propagation Model

Propagation carries Correlation Metadata among participants so
separately maintained Dialogs, Tasks, and Resources remain correctly
associated with their Activity.

Propagation adapts selected correlation information across Dialog
boundaries rather than copying a participant's entire state:

~~~
upstream correlation
      |
      | select      (choose the metadata the receiving Dialog needs)
      | translate   (map identifiers across the Dialog boundary)
      | correlate   (relate it to objects already present)
      | bind        (associate it with local objects and state)
      v
downstream correlation
~~~

The receiving Dialog's correlation context is related to, but need not
duplicate, upstream context. The bridging participant selects the
relevant associations.

Metadata may concern Activity association, Dialog or Task lineage, or
Resource identity and provenance, as described below.

Two properties follow, both developed later in this section:

- Propagation is selective: Only the metadata a receiving Dialog needs
  to maintain continuity, associate related interactions, or interpret
  shared Resources should be carried.
- Propagation is directional: What travels downstream differs from
  what travels upstream.

The architecture does not prescribe a fixed set of propagation events.
Propagation is need-driven. A participant selects the Correlation
Metadata needed when extending correlation to a new Dialog, subject
to applicable policies. Once a participant has accepted responsibility
for an established protocol-managed correlation, however, subsequent
changes relevant to that association are subject to its maintenance
responsibilities rather than a wholly new choice about whether to
maintain the correlation at all. Establishing a new Dialog, delegating
a Task, transferring a Resource, or resuming an interaction may each
create a need for propagation.

Participants may retain more correlations locally than they disclose;
receiving Dialogs need only the subset required for coherent
interaction.

## What Is Propagated

The main classes of Correlation Metadata are described below.


### Activity Correlation

A downstream interaction may need to know that it participates in the
same broader Activity as other interactions:

~~~
D2 belongs-to Activity A1
D5 belongs-to Activity A1
~~~
An Activity-ID is a stable, standardized semantic identity for
correlating an Activity's Dialogs, Tasks, and Resources.

Identity and disclosure are separate: a stable Activity-ID may be
shared, withheld, aliased, or mapped locally while preserving sufficient
correlation where needed.

How an Activity-ID is represented or exchanged is a policy and
protocol-design decision (see Identifier Translation).

### Dialog Lineage

When a new Dialog results from an existing one, the delegating
participant can retain the relationship:

~~~
D5 delegated-from D2
~~~

Dialog lineage preserves the causal or delegation relationship
between two Dialogs. Preserving lineage does not imply propagating or
disclosing it. A participant may retain lineage locally, and propagate
it only when the receiving Dialog needs it, for example for upstream
resolution or result correlation. When propagated, the relationship
may be carried directly or as an opaque reference, or it may be
withheld. Preservation, propagation, and disclosure are therefore
separate concerns.

Delegation is one lineage type among several; `continuation-of`,
`replaces`, `aggregated-from`, and `related-to` are the others, and
each propagates on the same terms described here.

### Task Lineage

A downstream Task may be correlated with the upstream Task from which
it originated:

~~~
t_15 delegated-from t_22
~~~

Task lineage is useful when a Dialog contains several Tasks
and only one of them caused the downstream delegation; delegated-from
records which one. Protocol-local Task identifiers are not intended
to travel; the relationship carries the correlation. Task lineage
is therefore optional and selective: it is established where the
receiving Dialog requires it for continuity, upstream resolution, or
correct interpretation, and withheld otherwise. Under hidden
delegation it may be retained only by the participant common to both
Dialogs.

As with Dialog lineage, `related-to` and `replaces` are examples of other Task
lineage types, and each propagates on the same terms.

### Resource Identity

A Resource that is referenced or transferred across Dialog, protocol,
or administrative boundaries may need to retain a stable logical
identity, so that independently operated participants can recognize
that they refer to the same Resource:

~~~
Dialog D2: Resource T1
Dialog D5: Resource T1
~~~

Where cross-Dialog correlation of a Resource is required, its logical
Resource identifier should be preserved across boundaries. Each domain
and application protocol remains free to hold its own local
representation; the Resource-ID provides the correlation:

~~~
Dialog D2: Resource-ID = T1, A2A artifactId = artifact-17
Dialog D5: Resource-ID = T1, A2A artifactId = artifact-91
~~~

A Resource's stable logical identity supports cross-Dialog correlation;
local identifiers, storage, and representation remain
implementation-specific. Content transfer is separate from the
correlation model.

Identity identifies the Resource; provenance describes its origins, and
Binding describes its local use.

#### Resource Provenance

Resource identity establishes how participants refer to the same
Resource. Resource Provenance is another resource-specific typed relationship. Examples include:

~~~
T1 derived-from V1
TXT-3 produced as a result of task 1 in dialog 1
~~~

TBD

#### Local Resource Binding

The same Resource can play a different role in each Dialog or Task in
which it appears. A Resource Binding associates a Resource with a
specific Dialog or Task and describes how it is used there:

~~~
T1 input-to TB1
T1 augments TC1
~~~

Unlike identity and provenance, a Binding is not an end-to-end
property of the Resource:

~~~
D2 / Task TB1:  T1 primary-input-to TB1
D5 / Task TC1:  T1 supporting-input-to TC1
~~~

The same logical Resource is referenced in both cases, but its local
role differs. When a Resource is propagated into a new Dialog, the
upstream Binding is not assumed to apply there. The delegating party
may carry it unchanged, translate it, supply it in part, leave it
unspecified, or let the receiving participant establish a new one.

Bindings are therefore local semantics. They may be carried across a
boundary when useful, but doing so does not make them universal:

~~~
T1 required-for TB1   does not imply   T1 required-for TC1
~~~

### Propagation Summary

The classes differ in what they are and how they typically travel:

| Information         | Role                                       | Typical propagation                              |
| ------------------- | ------------------------------------------ | ------------------------------------------------ |
| Activity-ID         | Stable identity of the Activity            | When cross-Dialog Activity correlation is needed  |
| Dialog lineage      | Causal or delegation relation between Dialogs | Retained where needed; selectively propagated |
| Task identifiers    | Protocol-local unit-of-work tracking       | Normally local to the application protocol       |
| Resource-ID         | Stable identity of a logical Resource      | When cross-Dialog Resource correlation is needed |
| Resource provenance | How a Resource was produced or derived     | When needed for correct interpretation           |
| Resource Binding    | Local role of a Resource                   | Primarily local; selectively carried             |

The rule is constant across all of them:

> Propagate only the correlation the receiving Dialog needs to maintain
> continuity, associate related interactions, or interpret shared
> Resources.


## Propagation Is Not Transitive by Default

Propagation in this architecture is selective rather than automatic.

A contextual element can be stable or important without being
transmitted to every participant. Each propagation decision depends on
whether the receiving Dialog requires that information for correlation,
continuity, or correct interpretation. Receiving context information does not imply that the receiver SHOULD
automatically propagate all of it further. For example:

~~~
A <----> B <----> C
~~~

B may receive from A:

~~~
Activity A1
Resource V1
Resource T1
A's identity
private policy metadata
~~~

and propagate to C only:

~~~
Activity correlation
V1
T1
~~~

Therefore:

> Context received from an upstream participant is not automatically
> transitive.

This principle governs the selection and disclosure of information
when extending correlation. It does not release a participant from
maintenance responsibilities already accepted for an established
protocol-managed association. A participant need not forward an
upstream update unchanged, but must preserve or communicate enough
information to maintain the correlations for which it is responsible,
or make the resulting inability to maintain them distinguishable.

Each propagation step is subject to a new projection decision. Such
decisions are left up to the participating parties to determine as
described in section 8.3 context projection. The defined protocol is a
tool that allows the selective propagation of context metadata.

This is important when agentic systems cross:

- organizational boundaries;
- trust boundaries;
- privacy boundaries;
- protocol boundaries; and
- changes in task extent.

Propagation is triggered by the needs of the receiving Dialog rather than
by a fixed set of protocol events. A new Dialog, delegated Task,
transferred Resource, resumed interaction, or modality transformation
may require propagation, but only the correlation information necessary
for that Dialog should be carried.

The architecture therefore defines propagation in terms of information
requirements and Dialog boundaries, rather than prescribing a universal
set of propagation events.

## Context Projection

Context Projection selects what is disclosed when B creates D2 from D1,
without copying D1's complete state. For example:

~~~
D1:
  Activity A1
  objective
  V1
  T1
  A's identity
  private policy
  conversation history
  internal reasoning
  credentials

        |
        | projection by B
        v

D2:
  Activity correlation
  delegated task objective
  V1
  T1
  relevant provenance
~~~

Projection therefore determines what is exposed to the downstream
Dialog. The projection is controlled by:

- task requirements;
- privacy policy;
- authorization policy;
- administrative-domain policy;
- protocol capabilities; and
- application semantics.

The projection algorithm is outside this architecture.


## Identifier Translation

Identifiers may change across domains, provided that the necessary
mappings preserve correlation:

~~~
Organization 1:
  Activity-ID = A1

Organization 2 external view:
  Activity alias = X7
~~~

Organization 2 can retain:

~~~
A1 <-> X7
~~~

privately.

Likewise, a Resource may be represented as:

~~~
Resource T1
~~~

in the architecture while having local identifiers:

~~~
A2A artifactId = artifact-17
MCP resource URI = ...
local database ID = 9f32
~~~

Identifier translation permits correlation without globally visible
implementation identifiers. Its mechanisms belong to the protocol
bindings or local-protocol specifications.

## Directional Propagation

Propagation can occur in both directions, but the information carried in
each direction can differ.

### Downstream propagation

Typically carries:

~~~
activity correlation
task objective
selected Resources
selected provenance
constraints
~~~

### Upstream propagation

Typically carries:

~~~
results
new Resources
status
questions
failure information
local observations
~~~

A downstream participant does not necessarily return the same context
that it received.

For example:

~~~
B -> C:
  V1
  T1
  analyze video

C -> B:
  result R1
  observation: T1 appears unnecessary
~~~

The second message is related to the same Activity but introduces new
local information.

## Propagation of Assertions

Correlation Metadata may be expressed as assertions, for example:

~~~
D2 delegated-from D1
T1 derived-from V1
T1 input-to TC1
~~~

An implementation should distinguish the existence of an assertion from
the authority or trustworthiness of the entity making that assertion. For example, receiving:

~~~
T1 derived-from V1
~~~

does not cryptographically prove the derivation. Likewise:

~~~
D2 delegated-from D1
~~~

does not by itself prove that the delegation was authorized.

Authentication, authorization, integrity protection, attestation, and
provenance mechanisms can be used independently where required. This
should be coordinated between the agentproto working group and other
respective IETF working groups such as WIMSE and OAuth.

Correlation assertions may also come from different sources. A
Dialog's or a Task's state may be reported by its participants,
observed by a peer, or asserted by a supervisory system. When two
sources disagree about the same object - for example, when one reports
a Task as in-progress while another reports it as failed - correlation
requires that the assertions remain associated with the correct object
and, where needed, distinguishable by their sources and relevant
context. This architecture does not determine which assertion is
authoritative or prescribe how disagreements are resolved.

### Lifecycle-Affecting Requests and Consequences

A lifecycle-affecting request, such as a request to cancel a Task, is
distinct from an assertion that the requested transition has occurred.
Application protocols may carry the request and report its outcome.
This architecture concerns the Correlation Metadata needed to associate
such requests and assertions with the affected Dialog or Task, and to
maintain correlations with dependent work in other Dialogs.

When an interruption, barge-in, cancellation, or security or
authorization change may affect related work, the necessary associations
may be preserved by a participant with visibility into the relevant
Dialogs and Tasks. Sufficient Correlation Metadata is propagated where
needed to allow affected participants to interpret the change and
associate resulting states or outcomes with it. The metadata need not
expose the full upstream context. Security- or authorization-related
information may be included where required for correlation; this
architecture does not define authorization decisions or enforcement.

## Context Freshness and Lifetime

Not all correlation assertions have the same lifetime. Examples include:

~~~
Dialog D2 terminated
delegation revoked
resource superseded
task completed
~~~

Correlation information may be:

~~~
stable
mutable
time-bounded
revocable
~~~

For example:

~~~
T1 derived-from V1
~~~

is typically stable provenance.

By contrast:

~~~
D2 active
~~~

is mutable lifecycle state.

Propagation mechanisms need appropriate freshness and update behavior
for each class.

A lifecycle assertion can reach a participant through intermediaries,
so the time its source observed the state can differ from the time the
assertion was forwarded or received. In the D1--D2--D3 chain (see
"Maintenance Responsibilities Within the Established Scope"), if
Agent B becomes unreachable, Agent A keeps relaying its last copy of
"t_13 in-progress" upstream, and the FNE cannot tell it from a
current one. Therefore:

- A lifecycle assertion carries the time its source observed the
  state (its source time).
- A participant that relays or translates the assertion, such as a
  bridging participant (see "The Propagation Model"), keeps the source
  time unchanged, including when it does not disclose the source (see
  "Hidden Delegation and Partial Visibility").
- A participant that receives the assertion measures its age from
  the source time rather than from when it received it.
- How freshness is evaluated, and how the source time is carried
  across intermediaries, are left to the dialog-management protocol.

## Propagation Revocation

Revocation invalidates a relationship as a basis for future propagation.
For example:

~~~
D2 delegated-from D1
~~~

is revoked, no new downstream propagation should be created based on
that relationship.

Revocation need not terminate the Dialog or erase the historical fact
that the relationship existed.

Protocol-design questions include whether revocation cascades, how
history is treated, and what participants do with earlier correlations
derived from a revoked relationship.

## Partial Visibility

No participant needs the complete Activity graph. For example:


For example:

~~~
            A
            |
           D1
            |
            B
           / \
         D2   Tool X
         |
         C
~~~

A may know only:

~~~
D1
B
~~~

C may know only:

~~~
D2
B
V1
T1
~~~

B may know the relationship between D1 and D2.

Tool X may know only:

~~~
V1
its local invocation
T1
~~~

The Activity can remain coherent with partial visibility. An
intermediary may conceal a downstream Dialog while preserving necessary
correlation, or establish it outside managed scope. Only accepted
managed correlations entail the corresponding maintenance obligations.

## Propagation Failure Modes

Correlation can fail even when application Tasks and transport operate
correctly. The following failures are distinct from application or
connectivity failures.

Propagation failures merit separate treatment.

Illustrative failure modes follow. These are just examples. The list may be extended as the work progresses

### Loss of Correlation

A participant may receive an interaction or Resource without being able
to associate it with the correct Activity, Dialog, Task, or Resource.

For example:

~~~
Agent D receives T1
~~~

but cannot determine which ongoing Activity or Dialog T1 belongs to.

The Resource may remain usable while cross-Dialog continuity is lost.

Loss of correlation can result from:

- Missing Activity correlation;
- Loss of Dialog lineage at an intermediary;
- Failure to preserve Resource identity; or
- Inability to map protocol-local identifiers to the broader correlation
  model.

### Incorrect Correlation

Incorrect correlation associates unrelated interactions or Resources.
For example:

For example:

~~~
T1 belongs-to Activity A1
~~~

may be incorrectly interpreted as:

~~~
T1 belongs-to Activity A2
~~~

Missed bindings may combine unrelated Activities or deliver results to the
wrong Dialog; correlation mappings are integrity-sensitive.

Implementations SHOULD therefore treat correlation identifiers and
relationship mappings as integrity-sensitive state.

### Over-Propagation

A participant may receive more metadata than needed. For example, a
summarizer requiring only T1 might receive:

For example, Agent D may require only T1 for summarization but receive:

~~~
Activity-ID A1
D3 delegated-from D2
D2 delegated-from D1
T1 derived-from V1
identities of upstream participants
~~~

even though none of that information is necessary for summarization.

Over-propagation increases:

- privacy exposure;
- topology disclosure;
- metadata size and overhead transaction;
- implementation coupling; and
- cross-domain tracking risk.

Selective propagation protects privacy and reduces coupling.

### Stale Context

Some context changes over time:

For example:

~~~
D2 ACTIVE
~~~

may later become:

~~~
D2 TERMINATED
~~~

or a propagation relationship may be revoked.

Stale context can produce incorrect decisions. Stable relationships and
mutable lifecycle assertions require different freshness handling.

For example, stable provenance differs from mutable Dialog state:

~~~
T1 derived-from V1
~~~

from mutable state, such as:

~~~
D2 ACTIVE
~~~

and allow propagation mechanisms to define appropriate freshness and
update behavior.

### Semantic Drift

A downstream Dialog may incorrectly reuse an upstream local
interpretation. For example:

For example:

~~~
D1 / TB1:
  T1 required-for TB1
~~~

must not automatically be interpreted as:

~~~
D2 / TC1:
  T1 required-for TC1
~~~

The Resource identity is preserved, but its local Binding may differ.

Semantic drift arises when a local Resource Binding is treated as
universally applicable.

### Broken Lineage

An intermediary may lose the relation connecting a delegated Dialog to
its origin. For example:

For example:

~~~
D1 -> D2
~~~

is created, but B later loses the relationship:

~~~
D2 delegated-from D1
~~~

Without that link, subsequent questions, results, and failures may not
be routed or correlated upstream.

Broken lineage obstructs:

- upstream resolution;
- result correlation;
- failure recovery; and
- audit or troubleshooting of delegated execution.

### Excessive Lineage Propagation

Excessive lineage disclosure is the opposite risk:

A system may preserve and propagate an unnecessarily complete delegation
chain:

~~~
D5 <- D4 <- D3 <- D2 <- D1
~~~

to every downstream participant.

An accurate but over-shared chain may reveal unnecessary participants,
domains, or workflow topology.

Preservation and disclosure are distinct.

### Inconsistent Local Mapping

Local mappings may include:

~~~
Activity A1 <-> local activity-773

Resource T1 <-> local artifact-19
~~~

Inconsistent mappings can bind a valid logical identifier to the wrong
local object, appearing externally as a cross-domain correlation
failure.

Such failures are local implementation failures, but they can appear
externally as incorrect cross-domain correlation.

### Partial Propagation Failure

In a multi-element propagation operation, some context may arrive while
other required context is lost.

For example, a participant may receive:

~~~
Resource T1
Activity-ID A1
~~~

but not the Resource identifier or relationship necessary to associate
T1 with the Task awaiting it.

Partial propagation can be more difficult to detect than complete
propagation failure because the receiving participant may have enough
information to continue execution while silently losing intended
correlation.

### Architectural Implication

These failure modes reinforce a central principle of this architecture:

> Correct correlation requires preserving sufficient information to maintain 
> the relationships among an Activity's Dialogs, Tasks, and Resources. 
> Such information may be maintained locally or selectively propagated
> between participants, depending on which associations need to be 
> established and by whom.

A propagation mechanism SHOULD therefore support detection or mitigation
of missing, stale, inconsistent, or excessive correlation information
without requiring every participant to maintain the complete Activity
graph. In a protocol-managed scope, the architecture further
recognizes the need to distinguish failures to maintain accepted
correlations from interactions that were never included in that scope.
Where supported by evidence, the affected managed step and its
responsible participant should be identifiable. This does not imply
complete visibility into undisclosed downstream interactions or
perfect attribution of the underlying cause of failure.

## Propagation Example

The video-analysis example can be expressed as follows.

Initial Dialog:

~~~
Activity A1

D1:
  A <-> B

Task TB1:
  V1 input-to TB1
~~~

Tool transformation:

~~~
A -> Tool X

T1 derived-from V1
T1 output-of IX1
~~~

T1 enters D1:

~~~
T1 input-to TB1
~~~

B creates a downstream Dialog:

~~~
D2 delegated-from D1
TC1 delegated-from TB1
~~~

B projects:

~~~
Activity correlation
V1
T1
T1 derived-from V1
task objective
~~~

into D2.

C develops local context:

~~~
T1 input-to TC1
T1 appears unnecessary
~~~

C asks B for clarification.

B uses locally retained lineage:

~~~
D2 delegated-from D1
TC1 delegated-from TB1
~~~

to resolve the question upstream.

The complete upstream Dialog does not need to be copied into D2, and the
downstream Dialog does not need to be exposed to A.

This is the intended meaning of context propagation in this
architecture.

## Relationship Taxonomy

The following are examples of relationship types that are useful
at the architecture level. They are illustrative, not an exhaustive
or normative set; the dialog-management protocol defines the
relationship types that are interoperable.


TBD


## Privacy and Administrative Boundaries

TBD

## Relationship to Dialog-Management Protocols

A dialog-management protocol may standardize only the subset of the
architectural model needed for interoperability. For example:

For example, a protocol may define:

~~~
Dialog-ID
Dialog-Relationship
Lifecycle State
Propagation State
~~~

without defining:

~~~
required-for
optional-for
application-specific planning semantics
~~~

This leaves application-specific semantics outside the interoperable
baseline.

### Architectural vs. Protocol Surface

The architecture defines Activities, Dialogs, Tasks, Resources, and
typed Relationships; it does not require every concept to become a
standardized protocol field.

The agentproto charter identifies Dialog identifiers, lifecycle
semantics, and Dialog-context propagation as subjects for its
standards-track protocol work. This information model examines related
correlation concepts that may inform that work, including:

- Dialog identity and identifier aliases;
- Dialog lineage and other typed relationships (e.g., delegated-from,
  aggregated-from, continuation-of, related-to, replaces);
- Dialog lifecycle and propagation assertions;
- Activity and Task associations where cross-Dialog correlation needs
  them; and
- Resource identity or provenance associations relevant to correlating
  interactions across modalities or tool invocations.

Protocol design determines which information is exchanged, locally
retained, or supported by application protocols. Resource Binding
remains primarily an application interpretation.

## Relationship to A2A

A2A provides useful local constructs such as:

~~~
contextId
taskId
artifactId
messageId
~~~

These map naturally into the architecture:

~~~
A2A contextId -> protocol-local Dialog context

A2A taskId -> Task

A2A artifactId -> local representation of a Resource
~~~

This architecture does not replace those constructs.

It provides a model for correlating them with objects outside one A2A
context.

## Relationship to MCP and Tool Protocols

TBD

## Relationship to Distributed Tracing

TBD

## Relationship to vCon

TBD

## Architectural Requirements

The following capabilities inform protocol design; they do not each
require distinct wire fields:

1. multiple Dialogs within one Activity;
2. independent protocol-local context identifiers;
3. stable Resource correlation where useful;
4. delegation lineage;
5. selective projection;
6. hidden downstream delegation;
7. cross-domain operation;
8. identifier translation;
9. provenance correlation;
10. privacy-preserving correlation;
11. correlation of lifecycle-affecting requests, resulting assertions,
    and dependency-driven consequences across Dialogs and Tasks;
12. distinction between the logical Activity scope and correlations
    established or maintained under a particular protocol;
13. selective extension of managed correlation, with responsibilities
    for maintaining the associations participants have accepted; and
14. detection or reporting of failures within managed correlation
    scope, with bounded identification of affected steps and responsible
    participants where supported by available evidence.

## Out-of-Scope

This document does not define:

- An agent's internal memory;
- Prompts or retrieved documents;
- Chain of thought;
- Agent planning semantics;
- Authorization decisions or enforcement mechanisms;
- Delegated-authority mechanisms;
- Trust establishment mechanisms;
- Agent discovery mechanisms;
- A wire format.

## Open Questions

1. Activity and Task are architectural concepts, not baseline
   protocol objects. The exact protocol surface for optional Activity
   and task-lineage extensions is to be determined in the protocol
   draft.
2. Should Dialog lineage be visible downstream or remain local by
   default?
3. What is the minimum Resource correlation required for multimodal
   dialog continuity?
4. Should Resource Binding semantics remain entirely
   application-defined?
5. How should logical Activity identity survive identifier translation?
6. How should partial resources such as video segments be represented?
7. Which relationships require integrity protection?
8. How should context-resolution questions be correlated across Dialogs?
9. How much lineage should cross administrative boundaries?
10. Which subset of this model belongs in a dialog-management protocol?

## IANA Considerations

This document requests no IANA actions.

## Security Considerations

Correlation metadata can reveal workflows, delegation, and Resource
relationships.

Implementations should minimize propagated information, avoid semantic
identifiers, and treat correlation identifiers as non-authoritative.

Possession of an identifier does not imply authorization.

Relationship assertions may require separate integrity or provenance
mechanisms.

## Privacy Considerations

A distributed Activity MUST NOT require a globally visible participant
list or a globally stable identifier.

Implementations should:

- minimize propagated metadata;
- preserve participant opacity where possible;
- translate identifiers at administrative boundaries;
- avoid propagating private dialog history;
- limit retention of correlation mappings.

## Appendix A. Canonical Architecture

~~~
Activity
 |
 +-- Dialog
 |    |
 |    +-- Task
 |         |
 |         +-- Resource
 |
 +-- Dialog
      |
      +-- Task
           |
           +-- Resource
~~~

Illustrative relationships:

~~~
Dialog belongs-to Activity
Task belongs-to Dialog
Dialog delegated-from Dialog
Task delegated-from Task
Resource derived-from Resource
Resource input-to Task
Resource output-of Task
~~~

